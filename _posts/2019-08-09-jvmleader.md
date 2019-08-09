---
layout: post
title: jvm总篇
categories: jvm
tags: java jvm 
author: zzr
date:   2019-08-09 00:00:00
---

* content
{:toc}


# 深入理解java虚拟机概述

总共分5章

- 第一章 走进java
  - 了解java技术来龙去脉，并介绍如何编译一个open jdk 7 
- [第二章 自动内存管理机制](#自动内存管理机制) 
  - 自动内存管理的诸多优势，但是出了问题如何排查问题
    - [讲解java 虚拟机 中内存如何划分](#运行时数据区域)。讲解各个区域内存溢出异常的常见原因
    - 分析垃圾收集的算法和jdk1.7中提供的几款垃圾收集齐的特点以及运作原理，用过代码验证java 虚拟机中自动内存以及回收的主要规则
    - 介绍了随jdk发布6个命令行工具与两个可视化的故障处理工具的使用方法
    - 与读者分享了几个比较有代表性的实际案例，还准备了一个所有开发人员都能亲身实战的练习，读者可通过实践来获得故障处理和调优的经验
- 第三章 虚拟机执行子系统
  - 了解虚拟机必不可少的组成部分
    - 讲解class文件各个组成部分
    - 介绍了加载过程的“加载”、“验证”、“准备”、“解析”、“初始化”  5个阶段
    - 分析虚拟机如何执行代码
    - 通过4个类加载以及执行子系统的案例
- 第四章 程序编译与代码优化
  - 编译字节码、从字节码编译成本地机器码的这两个过程
    - 分析泛型、主动装箱、拆箱、条件编译等多个语法糖的前因后果，并且使用插入式注解处理器实现一个检查程序命名规范的编译器插件
    - 讲解了虚拟机的热点探测方法
- 第五章 高效并发
  - 讲解多线程、高并发
    - 讲解java 内存模型的结构以及操作以及原子性、可见性、有序性
    - 讲解线程安全设计的概念和分类，同步实现的方式以及虚拟机底层的运作原理







## 自动内存管理机制

![运行时数据区](https://raw.githubusercontent.com/zzrdark/noteimg/master/img/20190716144129.jpg)

- 运行时数据区域
  - 程序计数器
    - 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令
    - 分支、循环、跳转、异常处理、线程恢复等基础功能度需要依赖这个计数器完成。
    - 当多个线程在一起运行的时候，各条线程度不能互相影响，需要独立存储。我们称为这类内存区域为线程私有的内存
    - 计数器记录
      - 如果正在执行java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址
      - 如果正在执行native方法，这个计数器则为空（undefined）
  - java虚拟机栈(java virtual Machine Stacks )
    - 和程序计数器同为线程私有的，生命周期和线程相同。
    - 执行方法时，会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息
    - 所谓的堆栈的栈是指 虚拟机栈 ，或者说虚拟机栈中的局部变量表
      - 局部变量表存放哪些？
        - 基本数据类型（boolean\byte\char\short\int\float\long\double）
          - long \double 会占用两个局部变量空间（slot）
          - 其余的只占用一个
        - 对象引用（reference类型和returnAddress类型）   
      - 当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的
      - 在这个区域内会有两个异常情况
        - StackOverflowError ： 如果线程请求的栈深度大于虚拟机所允许的深度，则会抛出
        - OutOfMemoryError:   虚拟机栈可以动态扩展 ，如果扩展时无法申请到足够的内存，则会抛出
  - 本地方法栈（Native Method Stack）
    - 本地方法栈和虚拟机栈锁发挥的作用很相似
      - 其实通过英文名字可以看出区别
      - 虚拟机栈 是为 java 方法（字节码）服务
      - 本地方法栈是为native方法服务
      - 同时也会抛出StackOverflowError 和 OutOfMemoryError 异常
  - java堆
    - 是java虚拟机中管理最大的一块内存
    - 是所有线程共享的一块内存区域
    - 存放对象实例
    - 垃圾收集器
      - java堆是垃圾收集器管理的主要区域
    - java堆 可以分成 新生代和老年代
      - 细致点分 Eden空间、From Survivor 空间、To Survivor 
      - 还可以划分出多个线程私有的分配缓冲区（Thread local AllocationBuffer ）
    - 数据结构 
      - 物理上不连续的内存空间，逻辑上连续。
    - 配置： -Xmx  和  -Xms   当堆上没有内存可以完成实例分配，则抛出OutOfMememoryError 
  - 方法区
    - 线程共享内存区域
    - 用于加载类信息、常量、静态变量、即时编译器后的代码等数据、
    - 又称：Non-Heap(非堆)
    - 方法区称为：“永久代”
    - 配置：-XX:MaxPermSize
    - jdk 1.7 之前 实现方法区的方式是永久代(PermGen)。永久代和堆相互隔离。
    - 存储在永久代的部分数据就已经转移到 Java Heap(堆) 或者 Native memory(本地内存)。但是 1.7的版本并没有完全移除永久代
    - jdk1.7以后已经把原本放在永久代的 字符串常量池 移出
    - 无法满足内存分配时，将抛出OutOfMemoryError
    - jdk1.8后 取消了实现方式不同了，使用的元空间 (MetaSpace，使用的是本地内存 ，并不在虚拟机上)永久的代替代了永久代。和堆也是不相连。
  - 运行时常量池
    - 其实他是属于方法区的一部分
    - 用于存储编译期生成的个钟字面量和符号引用
    - 类加载之后进入方法区后，才把这部分内容存放常量池中
    - 常量池具备动态性，两个产生的可能
      - 编译时产生
      - 运行期间也可能产生
        - 比如String.intern()方法
    - 常量池无法再申请内存时会抛出OutOfMemoryError异常
    
  - 直接内存
    - 内存空间
      - 其实这个并不属于java虚拟机运行时数据区的一部分
      - 也不是java虚拟机规范中定义的内存区域
      - 但是如果内存申请不够，也会导致OutOfMemoryError 
      - 这个直接内存受到本机总内存限制
    - jdk1.5中加入了NIO(new Input/Output)
      - 它可以使用Native函数库直接分配堆外内存
      - 操作：通过java堆的DirectByteBuffer对象作为这块内存的引用进行操作
      - 避免了在java堆和native堆来回的复制数据
