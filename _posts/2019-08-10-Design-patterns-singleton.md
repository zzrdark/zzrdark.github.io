---
layout: post
author: zzrdark
title:设计模式-单例模式
tags: 设计模式
categories: 设计模式 
data: 2019-08-10 12:20:00

---



[TOC]





# 单例模式
-   应用场景：保证一个类仅有一个实例，并提供一个访问它的全局访问点、还需要考虑现场安全问题
    -   饿汉式、懒汉式、注册登记式、枚举式、序列化与反序列化会出现多例
-   Spring 中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory
-   但没有从构造器级别去控制单例，这是因为Spring 管理的是是任意的Java 对象
-   Spring 下默认的Bean 均为单例



## 饿汉式：
-   在实例使用之前，不管你用不用，我都先new 出来再说，避免了线程安全问题
-   优点：没有加任何的锁、执行效率比较高。
-   缺点：类加载的时候就是初始化，不管你用还是不用，我都占着空间，比较浪费内存
-   绝对的线程安全，在线程还没出现的以前就是实例化了，


```` java 
public class Hungry {

    private Hungry(){}
    //先静态、后动态
    //先属性、后方法
    //先上后下
    private static final Hungry hungry = new Hungry();

    public static Hungry getInstance(){
//        Hungry hungry;

        return  hungry;
    }

}

//test
````


## 懒汉式：
-    在第一次使用的时候开始初始化

```` java 
//在外部需要使用的时候才进行实例化
public class LazyOne {
    private LazyOne(){}


    //静态块，公共内存区域
    private static LazyOne lazy = null;

    public static LazyOne getInstance(){

        //调用方法之前，先判断
        //如果没有初始化，将其进行初始化,并且赋值
        //将该实例缓存好
        if(lazy == null){
            //两个线程都会进入这个if里面
            //不是线程安全
            lazy = new LazyOne();
        }
        //如果已经初始化，直接返回之前已经保存好的结果

        return lazy;

    }

}

//保证了线程安全，但是性能损耗大
public class LazyTwo {

    private LazyTwo(){}

    private static LazyTwo lazy = null;

    public static synchronized LazyTwo getInstance(){

        if(lazy == null){
            lazy = new LazyTwo();
        }
        return lazy;

    }
}


//懒汉式单例
//特点：在外部类被调用的时候内部类才会被加载
//内部类一定是要在方法调用之前初始化
//巧妙地避免了线程安全问题

//这种形式兼顾饿汉式的内存浪费，也兼顾synchronized性能问题
//完美地屏蔽了这两个缺点
//史上最牛B的单例模式的实现方式
public class LazyThree {

    private boolean initialized = false;

    //默认使用LazyThree的时候，会先初始化内部类
    //如果没使用的话，内部类是不加载的
    private LazyThree(){

        synchronized (LazyThree.class){
            if(initialized == false){
                initialized = !initialized;
            }else{
                throw new RuntimeException("单例已被侵犯");
            }
        }

    }


    //每一个关键字都不是多余的
    
    //static 是为了使单例的空间共享
    //保证这个方法不会被重写，重载
    public static final LazyThree getInstance(){

        System.out.println("调用");
        //在返回结果以前，一定会先加载内部类
        return LazyHolder.LAZY;
    }


    //默认不加载
    private static class LazyHolder{
        private static final LazyThree LAZY = new LazyThree();
        static {
            System.out.println("内部类被实例化");
        }

    }
//反射
public class LazyThreeTest {

    public static void main(String[] args) {

        try{

            //很无聊的情况下，进行破坏
            Class<?> clazz = LazyThree.class;


            //通过反射拿到私有的构造方法
            Constructor c = clazz.getDeclaredConstructor(null);
            //强制访问，强吻，不愿意也要吻
            c.setAccessible(true);

            //暴力初始化
            Object o1 = c.newInstance();

            //调用了两次构造方法，相当于new了两次
            //犯了原则性问题，
            Object o2 = c.newInstance();
            System.out.println(o1 == o2);
//            Object o2 = c.newInstance();
			
            Method method = clazz.getDeclaredMethod("getInstance");
            Object returnOb = method.invoke(o1);
            if (returnOb instanceof LazyThree ){
                LazyThree lazyThree = (LazyThree) returnOb;
                System.out.println("lazyTree:"+lazyThree);
            }


        }catch (Exception e){
            e.printStackTrace();
        }

    }
}
````


## 注册模式：
-   Spring中的做法。


```` java 
//容器
//Spring中的做法，就是用这种注册式单例
public class BeanFactory {

    private BeanFactory(){}

    //线程安全
    private static Map<String,Object> ioc = new ConcurrentHashMap<String,Object>();

    //获取实例，如果不存在则生成一个并放到容器中
    public static Object getBean(String className) {

        if (!ioc.containsKey(className)) {
            Object obj = null;
            try {
                obj = Class.forName(className).newInstance();
                ioc.put(className, obj);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return obj;
        } else {
            return ioc.get(className);
        }
    }
}
````

## 序列化：
-   反序列化导致单例被破坏序列化
-   序列化
    -   序列化就是说把内存中的状态通过转换成字节码的形式
    -   从而转换一个IO流，写入到其他地方(可以是磁盘、网络IO)
    -   内存中状态给永久保存下来了


```` java 
//反序列化时导致单例破坏
public class Seriable implements Serializable {


    public  final static Seriable INSTANCE = new Seriable();
    private Seriable(){}

    public static  Seriable getInstance(){
        return INSTANCE;
    }
    
    
	//实现 readResolve 方法能保证单例
    private  Object readResolve(){
        return  INSTANCE;
    }
}
````