---
title: Spring-7-AOP基础
date: 2019-01-10 10:20:23
tags: ["Spring", "AOP"]
---
　　　　AOP联盟将AOP体系从高到低分成三个层次，最高层是语言和开发环境，在这个层次中有几个重要的概念：“基础”（base）为增强对象或者说目标对象；“切面”（aspect）通常包含低于基础的增强应用；“配置”（configuration）可以视为编织，将基础和切面编织组合起来以达到对目标对象的编织。

　　　　编织逻辑的具体实现方法有反射、程序预处理、拦截器框架、类装载器框架、元数据处理等。Spring AOP中使用的是Java语言特性如Java Proxy、拦截器等技术。

<!-- more -->

## Advice（通知）

Advice定义了在连接点做什么，为切面增强提供织入入口。在Spring AOP中，主要描述Spring AOP围绕方法调用而注入的切面行为。Advice由AOP联盟定义，在org.aopalliance包中。Spring AOP使用了这个接口，并做了扩展，提供了更细化的通知类型，如BeforeAdvice、AfterAdvice、ThrowAdvice等。

1. BeforeAdvice: 目标被被使用前调用，目前只有目标方法被调用前的通知MethodBeforeAdvice，未来可能会有Field Advice；
2. AfterAdvice：后置通知表记接口，子接口有AfterReturningAdvice和ThrowsAdvice；
3. ThrowAdvice：异常处理表记接口，AfterAdvice的子接口，此接口并没有提供方法，是通过反射调用，所以此接口的实现必须有以下四个方法中的一个，具体看一参考ThrowsAdviceInterceptor类：
 * public void afterThrowing(Exception ex)
 * public void afterThrowing(RemoteException)
 * public void afterThrowing(Method method, Object[] args, Object target, Exception ex)
 * public void afterThrowing(Method method, Object[] args, Object target, ServletException ex)

## PointCut（切点）

PointCut用来表示Advice将要作用的连接点，通过PointCut来定义需要增强的方法集合。Spring提供了一些开箱即用的PointCut类，类图如下：

{% asset_img Pointcut.png Pointcut类图 %}

PointCut接口提供了两个方法分别返回ClassFilter和MethodMatcher，用于过滤类和匹配方法。

## Advisor(通知器)

Advisor将Advice和PointCut组合起来，定义了在哪个PointCut应用哪个Advice。