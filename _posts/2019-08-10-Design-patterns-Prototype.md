---
layout: post
author: zzrdark
title:设计模式-原型模式
tags: 设计模式
categories: 设计模式 
data: 2019-08-10 12:30:00

---



[TOC]



# 原型模式（Prototype）

-   应用场景：原型模式就是从一个对象再创建另一个可定制的对象，而且不需要知道任何创建的细节。
-   所谓的原型模式：就是java中的克隆技术，以某个对象为原型，复制出的新对象。
-   显然新的对象具备原型对象的特点，效率高（避免了重新执行的构造过程步骤



## 浅拷贝：
-   继承Cloneable接口，实现clone方法，仅仅是浅拷贝
    -   会把栈上的东西拷贝进去（对象的话，拷贝过去就是对象引用）
    -   能够直接拷贝其实实际内容的数据类型，只支持9种，拷贝过去的就是对象引用



## 深拷贝：
-   采用序列化与反序列化的方法。
    -   自定义一个deepClone

```` java 
@Override
    protected Object clone() throws CloneNotSupportedException {
        return this.deepClone();
    }


    public Object deepClone(){
        try{

            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);

            QiTianDaSheng copy = (QiTianDaSheng)ois.readObject();
            copy.birthday = new Date();
            return copy;

        }catch (Exception e){
            e.printStackTrace();
            return null;
        }

    }
````