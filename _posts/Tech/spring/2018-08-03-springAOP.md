---
layout: post
category: spring
title: springAOP
---

[link](https://www.cnblogs.com/liuruowang/p/5711563.html)

AOP称为面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。

## 一 AOP的基本概念

(1)Aspect(切面):通常是一个类，里面可以定义切入点和通知

(2)JointPoint(连接点):程序执行过程中明确的点，一般是方法的调用

(3)Advice(通知):AOP在特定的切入点上执行的增强处理，有before,after,afterReturning,afterThrowing,around 	这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。

(4)Pointcut(切入点):就是带有通知的连接点，在程序中主要体现为书写切入点表达式

(5)AOP代理：AOP框架创建的对象，代理就是目标对象的加强。Spring中的AOP代理可以使JDK动态代理，也可以是CGLIB代理，前者基于接口，后者基于子类


## 二 Spring AOP

Spring中的AOP代理还是离不开Spring的IOC容器，代理的生成，管理及其依赖关系都是由IOC容器负责，Spring默认使用JDK动态代理，在需要代理类而不是代理接口的时候，Spring会自动切换为使用CGLIB代理，不过现在的项目都是面向接口编程，所以JDK动态代理相对来说用的还是多一些。

Spring AOP 模块提供拦截器来拦截一个应用程序，例如，当执行一个方法时，你可以在方法执行之前或之后添加额外的功能。


## 三 基于注解的AOP配置方式

1.启用@AsjectJ支持

在applicationContext.xml中配置下面一句:
```xml
<aop:aspectj-autoproxy />
```

2.通知类型介绍

(1)Before:在一个方法执行之前，执行通知。在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可

(2)AfterReturning:在目标方法正常完成后做增强,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值

(3)AfterThrowing:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名

来访问目标方法中所抛出的异常对象

(4)After:在目标方法完成之后做增强，无论目标方法时候成功完成。@After可以指定一个切入点表达式

(5)Around:在建议方法调用之前和之后，执行通知。环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个ProceedingJoinPoint

3.例子：

![](https://images2015.cnblogs.com/blog/760279/201607/760279-20160727152818888-1665005413.png)

(1)Operator.java --> 切面类

```java
@Component
@Aspect
public class Operator {
    
    @Pointcut("execution(* com.aijava.springcode.service..*.*(..))")
    public void pointCut(){}
    
    @Before("pointCut()")
    public void doBefore(JoinPoint joinPoint){
        System.out.println("AOP Before Advice...");
    }
    
    @After("pointCut()")
    public void doAfter(JoinPoint joinPoint){
        System.out.println("AOP After Advice...");
    }
    
    @AfterReturning(pointcut="pointCut()",returning="returnVal")
    public void afterReturn(JoinPoint joinPoint,Object returnVal){
        System.out.println("AOP AfterReturning Advice:" + returnVal);
    }
    
    @AfterThrowing(pointcut="pointCut()",throwing="error")
    public void afterThrowing(JoinPoint joinPoint,Throwable error){
        System.out.println("AOP AfterThrowing Advice..." + error);
        System.out.println("AfterThrowing...");
    }
    
    @Around("pointCut()")
    public void around(ProceedingJoinPoint pjp){
        System.out.println("AOP Aronud before...");
        try {
            pjp.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("AOP Aronud after...");
    }
    
}
```

(2)UserService.java --> 定义一些目标方法

```java
@Service
public class UserService {
    
    public void add(){
        System.out.println("UserService add()");
    }
    
    public boolean delete(){
        System.out.println("UserService delete()");
        return true;
    }
    
    public void edit(){
        System.out.println("UserService edit()");
        int i = 5/0;
    }
    
    
}
```

(3).applicationContext.xml

```xml
<context:component-scan base-package="com.aijava.springcode"/>
    
<aop:aspectj-autoproxy />
```
(4).Test.java
```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        UserService userService = (UserService) ctx.getBean("userService");
        userService.add();
    }
}
```

上面是一个比较简单的测试，基本涵盖了各种增强定义。注意:做环绕通知的时候，调用ProceedingJoinPoint的proceed()方法才会执行目标方法。

4.通知执行的优先级

进入目标方法时,先织入Around,再织入Before，退出目标方法时，先织入Around,再织入AfterReturning,最后才织入After。

注意:Spring AOP的环绕通知会影响到AfterThrowing通知的运行,不要同时使用!同时使用也没啥意义。

5.切入点的定义和表达式

切入点表达式的定义算是整个AOP中的核心，有一套自己的规范

Spring AOP支持的切入点指示符：

(1)execution:用来匹配执行方法的连接点

    A:@Pointcut("execution(* com.aijava.springcode.service..*.*(..))")

第一个*表示匹配任意的方法返回值，..(两个点)表示零个或多个，上面的第一个..表示service包及其子包,第二个*表示所有类,第三个*表示所有方法，第二个..表示

方法的任意参数个数

    B:@Pointcut("within(com.aijava.springcode.service.*)")

within限定匹配方法的连接点,上面的就是表示匹配service包下的任意连接点

    C:@Pointcut("this(com.aijava.springcode.service.UserService)")

this用来限定AOP代理必须是指定类型的实例，如上，指定了一个特定的实例，就是UserService

    D:@Pointcut("bean(userService)")

bean也是非常常用的,bean可以指定IOC容器中的bean的名称

6.基于XML形式的配置方式

开发中如果选用XML配置方式，通常就是POJO+XML来开发AOP,大同小异，无非就是在XML文件中写切入点表达式和通知类型

## 四 Spring 中基于 AOP 的 XML架构
为了在本节的描述中使用 aop 命名空间标签，你需要导入 spring-aop j架构，如下所述：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

   <!-- bean definition & AOP specific configuration -->

</beans>
```

你还需要在你的应用程序的 CLASSPATH 中使用以下 AspectJ 库文件
```
aspectjrt.jar
aspectjweaver.jar
aspectj.jar
aopalliance.jar
```

### 声明一个 aspect

一个 aspect 是使用 元素声明的，支持的 bean 是使用 ref 属性引用的，如下所示：
```xml
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
   ...
   </aop:aspect>
</aop:config>
<bean id="aBean" class="...">
...
</bean>
```

这里，“aBean” 将被配置和依赖注入，就像前面的章节中你看到的其他的 Spring bean 一样。

### 声明一个切入点

一个切入点有助于确定使用不同建议执行的感兴趣的连接点（即方法）。在处理基于配置的 XML 架构时，切入点将会按照如下所示定义：
```xml
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
   <aop:pointcut id="businessService"
      expression="execution(* com.xyz.myapp.service.*.*(..))"/>
   ...
   </aop:aspect>
</aop:config>
<bean id="aBean" class="...">
...
</bean>

```

下面的示例定义了一个名为 “businessService” 的切入点，该切入点将与 com.tutorialspoint 包下的 Student 类中的 getName() 方法相匹配：
```xml
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
   <aop:pointcut id="businessService"
      expression="execution(* com.tutorialspoint.Student.getName(..))"/>
   ...
   </aop:aspect>
</aop:config>
<bean id="aBean" class="...">
...
</bean>
```

注意，aop的ref指向的bean是aspect啊！！！就是有切入点和advice的那个类

```xml
你可以使用 <aop:{ADVICE NAME}> 元素在一个 中声明五个建议中的任何一个，如下所示：
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
      <aop:pointcut id="businessService"
         expression="execution(* com.xyz.myapp.service.*.*(..))"/>
      <!-- a before advice definition -->
      <aop:before pointcut-ref="businessService" 
         method="doRequiredTask"/>
      <!-- an after advice definition -->
      <aop:after pointcut-ref="businessService" 
         method="doRequiredTask"/>
      <!-- an after-returning advice definition -->
      <!--The doRequiredTask method must have parameter named retVal -->
      <aop:after-returning pointcut-ref="businessService"
         returning="retVal"
         method="doRequiredTask"/>
      <!-- an after-throwing advice definition -->
      <!--The doRequiredTask method must have parameter named ex -->
      <aop:after-throwing pointcut-ref="businessService"
         throwing="ex"
         method="doRequiredTask"/>
      <!-- an around advice definition -->
      <aop:around pointcut-ref="businessService" 
         method="doRequiredTask"/>
   ...
   </aop:aspect>
</aop:config>
<bean id="aBean" class="...">
...
</bean>
```