---
title: Spring-6-BeanFactoryPostProcessor
date: 2018-12-27 15:45:51
tags: ["Spring"]
---

BeanFactoryPostProcessor是一个扩展点接口，用来在BeanFactory创建完成，BeanDefinition加载完后修改Bean Definition中的属性。Application Context会自动探测到Bean Definitions中的BeanFactoryPostProcessor，并将它们在其他Bean创建前应用。这个接口对于使用自定义配置文件中的值覆盖应用上下文中配置的属性值，比如PropertyResourceConfigurer和它的子类就是用来实现这个目的的。BeanFactoryPostProcessor只可能会修改BeanDefinition，不会修Bean。如果修改Bean，会导致Bean过早实例化，侵犯容器，产生副作用。


BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor

<!-- more -->

一些常见的BeanFactoryPostProcessor：
* PropertyResourceConfigurer 加载配置中属性值
* ConfigurationClassPostProcessor，使用CGLIB增强Configuration类，具体参考enhanceConfigurationClasses
* CustomEditorConfigurer：注册PropertyEditor
* Spring Boot中org.springframework.boot.context.config.ConfigFileApplicationListener.PropertySourceOrderingPostProcessor，用于对属性源进行排序，将默认的属性放到最后