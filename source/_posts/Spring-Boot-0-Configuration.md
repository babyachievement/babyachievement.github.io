---
title: Spring-Boot-0-@Configuration
date: 2018-12-21 15:33:34
tags: ["Spring Boot"]
---

# SpringBoot 启动注册主程序类

org.springframework.context.support.GenericApplicationContext#registerBeanDefinition

org.springframework.boot.SpringApplication#run(java.lang.String...)
org.springframework.boot.SpringApplication#prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner)
org.springframework.boot.SpringApplication#load(ApplicationContext context, Object[] sources) 
org.springframework.boot.BeanDefinitionLoader#load()
org.springframework.boot.BeanDefinitionLoader#load(java.lang.Object)
org.springframework.boot.BeanDefinitionLoader#load(java.lang.Class<?>)
org.springframework.context.annotation.AnnotatedBeanDefinitionReader#register(Class<?>... annotatedClasses)
org.springframework.context.annotation.AnnotatedBeanDefinitionReader#registerBean(java.lang.Class<?>)
org.springframework.context.annotation.AnnotatedBeanDefinitionReader#register

将@SpringBootApplication注解类注册到BeanDefinition中，其他的类将由ConfigurationClassPostProcessor自动扫描。ConfigurationClassPostProcessor在processConfigBeanDefinitions中创建了ConfigurationClassParser，并调用ConfigurationClassParser.parse(candidates)方法解析@Configuration注解的类。整个调用流程如下：

```Java
// BeanDefinitionRegistryPostProcessor扩展点
org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry
// 从BeanDefinitionRegistry找到所有的Full和Lite Configuration BeanDefinition
org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions(BeanDefinitionRegistry registry)
org.springframework.context.annotation.ConfigurationClassParser#parse(java.util.Set<org.springframework.beans.factory.config.BeanDefinitionHolder>)
org.springframework.context.annotation.ConfigurationClassParser#parse(org.springframework.core.type.AnnotationMetadata, java.lang.String)
org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass(ConfigurationClass configClass)
org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)throws IOException
// 调用ComponentScanAnnotationParser解析@ComponentScan
org.springframework.context.annotation.ComponentScanAnnotationParser#parseparse(AnnotationAttributes componentScan, final String declaringClass)
// 扫描包
org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan
```




## full VS lite Configuration

1. 如果类上使用了@Configuration，则为full Configuration
2.如果不满足1，检查类上是否使用了@Component、@ComponentScan、@Import或@ImportResource,如果都没有使用，则检查方法上是否使用了@Bean

## @Configuration解析过程
ConfigurationClassPostProcessor同时实现了BeanDefinitionRegistryPostProcessor和

从已有的 BeanDefinition 中，找到配置类，然后解析出更多 BeanDefinition


postProcessBeanFactory
这里就是增强配置类，添加一个 ImportAwareBeanPostProcessor，这个类用来处理 ImportAware 接口实现

## Full Configuration

Full Configuration 和

ConfigurationClassEnhancer

BeanMethodInterceptor

scope proxy