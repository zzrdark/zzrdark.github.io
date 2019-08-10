---
layout: post
author: zzrdark
title: 设计模式-工厂模式
tags: 设计模式
categories: 设计模式 
data: 2019-08-10 12:10:00
---



* content
{:toc}


# 个人感悟：
-   设计模式都是处理复杂问题的，如果问题本身很简单，使用设计模式反而累赘，增加了开发的复杂性。
-   遇到最简单的情况，直接new
-   如果创建对象过程简单，但是需要匹配不同的情况，返回不同对象时，考虑使用简单工厂。
-   如果遇到对象创建过程复杂（比如数据库连接这样的复杂对象，不仅仅牵涉到new 一个对象这么简单），这样需要把创建过程抽象出来，统一编写，对外提供干净的接口调取即可：考虑使用工厂模式；
-   如果遇到对象创建过程复杂、而且多对象匹配问题（比如默认去缓存库redis取，如果缓存库娶不到，去oracle数据库取），在上一条我们已经把redis、oracle的创建做成了工厂，此时需要对二者统一再抽象一层：考虑使用抽象工厂


## 简单工厂模式：
-   做静态工厂方法（StaticFactory Method）模式，但不属于23 种设计模式之一
-   简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类
-   Spring 中的BeanFactory 就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean 对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定




```` java 
public class SimpleFactory {

    public Milk getMilk(String name){
        if("特仑苏".equals(name)){
            return new Telunsu();
        }else if("伊利".equals(name)){
            return new Yili();
        }else if("蒙牛".equals(name)){
            return new Mengniu();
        }else {
            System.out.println("不能生产您所需的产品");
            return null;
        }
    }

}
````


## 抽象工厂模式：
-   何时使用：系统的产品有多于一个的产品族，而系统只消费其中某一族的产品
-   如何解决：在一个产品族里面，定义多个产品。每个具体的工厂负责一个产品族。抽象工厂的返回值为最高级抽象产品



```` java 
/**
 *
 * 抽象工厂是用户的主入口
 * 在Spring中应用得最为广泛的一种设计模式
 * 易于扩展
 * Created by Tom on 2018/3/4.
 */
public abstract class AbstractFactory {

    //公共的逻辑
    //方便于统一管理

    /**
     * 获得一个蒙牛品牌的牛奶
     * @return
     */
    public  abstract Milk getMengniu();

    /**
     * 获得一个伊利品牌的牛奶
     * @return
     */
    public abstract  Milk getYili();

    /**
     * 获得一个特仑苏品牌的牛奶
     * @return
     */
    public  abstract  Milk getTelunsu();

    public abstract Milk getSanlu();

}

/**
 * Created by Tom on 2018/3/4.
 */
public class MilkFactory extends  AbstractFactory {


    @Override
    public Milk getMengniu() {
        return new Mengniu();
    }

    @Override
    public Milk getYili() {
        return new Yili();
    }

    @Override
    public Milk getTelunsu() {
        return new Telunsu();
    }

    @Override
    public Milk getSanlu() {
        return new Sanlu();
    }
}
````