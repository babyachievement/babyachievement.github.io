---
title: Spring Boot-1. 架构
date: 2018-12-20 10:30:01
tags: ["Spring Boot"]
---


## ImportSelector

Import

## autoconfigure模块

@ConfigurationProperties和spring-configuration-metadata.json

spring-boot-autoconfigure-processor


ConditionalOnMissingBean 
Conditional 

## starter模块

##Spring boot启动过程

org/springframework/boot/BeanDefinitionLoader.java:148

org.springframework.context.annotation.AnnotatedBeanDefinitionReader#register

org.springframework.context.annotation.AnnotatedBeanDefinitionReader#registerBean(java.lang.Class<?>)


org/springframework/context/annotation/ComponentScanAnnotationParser.java:123