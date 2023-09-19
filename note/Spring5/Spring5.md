## 1. Spring框架概述

![img](https://cdn.nlark.com/yuque/0/2023/png/38605385/1695086040848-f3414c6f-e63b-40ec-bd48-7a4e0064e470.png)

1.  Spring是轻量级的开源的JavaEE框架；   轻量级-jar包少可独立使用
2. Spring可以解决企业应用开发的复杂性
3. Spring有两个核心部分：IOC和AOP

  （1）IOC：控制反转，把创建对象的过程交给Spring进行管理

  （2）AOP：面向切面，不修改源代码进行功能增强

1. Spring的特点

​    （1）方便解耦，简化开发

​    （2）AOP编程的支持

​    （3）方便程序测试Junit

​    （4）方便与其他优秀框架进行整合

​    （5）降低Java EEApi的使用难度，比如对JDBC的封装等

​    （6）方便进行事务的操作

​    （7）方便解耦，简化开发

## 2.  IOC容器

1. 什么是IOC 控制

​    (1) 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理

​    (2) 使用IOC的目的，为了耦合度降低

​    (3)入门案例中的bean1.xml形式就是IOC的实现

### 2.1. IOC底层原理

1. xml解析、工厂模式、反射

#### 2.1.1. 1.0 传统：

耦合度低

![img](https://cdn.nlark.com/yuque/0/2023/png/38605385/1695092173714-4606b007-185b-4d94-a8a1-5e273c816770.png)

#### 2.1.2. 2.0 工厂模式：

![img](https://cdn.nlark.com/yuque/0/2023/png/38605385/1695092378573-32ed78a5-540b-4e90-a3ee-24182156ed5b.png)

#### 2.1.3. 3.0  IOC

耦合度高

![img](https://cdn.nlark.com/yuque/0/2023/png/38605385/1695093363070-4d9eab5f-f760-447b-92bf-c66554763be3.png)





### 2.2. IOC接口(BeanFactory接口)

### 2.3. IOC操作Bean管理(基于xml)

### 2.4. IOC操作Bean管理(基于注解)

## 3. AOP

## 4. JdbcTempLate

## 5. 事务管理

## 6. Spring5新特性