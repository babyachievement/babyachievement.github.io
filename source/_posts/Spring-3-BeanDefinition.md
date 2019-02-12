---
title: Spring-3-BeanDefinition
date: 2018-12-27 10:07:52
tags: ["Spring"]
---

## BeanDefinition是什么？

　　作为一个IOC容器框架，Spring管理Bean的生命周期，自然需要知道这些Bean用哪些类创建？在什么时候需要被创建？这些Bean在创建的时候需要哪些参数？创建后需要为其设置哪些属性？需要调用什么初始化方法？Bean的是什么类型的？Bean销毁时调用什么方法？

<!-- more -->

　　这些数据都是通过BeanDefinition描述的，在BeanDefinition中，使用beanClassName描述创建Bean的类，或者创建FactoryMethod，lazyInit判断Bean是在什么时候去创建，使用constructorArgumentValues描述创建时需要的参数，使用propertyValues描述创建Bean后要设置的属性，使用initMethodName描述初始化bean需要调用的方法，使用scope描述Bean的类型（singleton|prototype），使用destroyMethodName描述销毁Bean时要调用的方法。


　　除此之外，BeanDefinition还提供了primary属性，用于判断是不是首选；role用于区分Bean是ROLE_APPLICATION，ROLE_SUPPORT还是ROLE_INFRASTRUCTURE。

## BeanDefinition分类

{% asset_img BeanDefinition.png BeanDefinition类图 %}

　　BeanDefinition接口实现了AttributeAccessor和BeanMetadataElement接口，BeanMetadataElement接口具有访问source（配置源）的能力。AttributeAccessor接口提供了操作附加属性的能力。BeanDefinition有两个子类：AbstractBeanDefinition和AnnotatedBeanDefinition。

　　RootBeanDefinition,ChildBeanDefinition和GenericBeanDefinition都继承了AbstractBeanDefinition。

1. GenericBeanDefinition是一站式的标准bean definition，除了具有指定类、可选的构造参数值和属性参数这些其它bean definition一样的特性外，它还具有通过parenetName属性来灵活设置parent bean definition。

2. RootBeanDefinition表示可合并的bean definition：即在spring beanFactory运行期间，支持一个特定的bean。RootBeanDefinition可以由多个彼此继承的BeanDefinition创建。RootBeanDefinition本质上是运行时统一的bean definition 视图。RootBeanDefinition也可以用来在配置阶段进行注册单个bean definition。但spring 2.5后，注册bean definition的首先方法是使用GenericBeanDefinition。GenericBeanDefinition支持动态定义父类依 赖，而非硬编码将其作为root bean definition。

　　通常， GenericBeanDefinition用来注册用户可见的bean definition(可见的bean definition意味着可以在该类bean definition上定义post-processor来对bean进行操作)，此外可以使用parentName灵活地配置parent bean。RootBeanDefinition / ChildBeanDefinition用来定义已经预先确定了parent/child关系的bean definition。


　　AnnotatedBeanDefinition接口添加了获取注解元数据的方法getMetadata和getFactoryMethodMetadata。有三个子类ScannedGenericBeanDefinition，ConfigurationClassBeanDefinition和AnnotatedGenericBeanDefinition。

```Java
public interface AnnotatedBeanDefinition extends BeanDefinition {

	/**
	 * Obtain the annotation metadata (as well as basic class metadata)
	 * for this bean definition's bean class.
	 * @return the annotation metadata object (never {@code null})
	 */
	AnnotationMetadata getMetadata();

	/**
	 * Obtain metadata for this bean definition's factory method, if any.
	 * @return the factory method metadata, or {@code null} if none
	 * @since 4.1.1
	 */
	@Nullable
	MethodMetadata getFactoryMethodMetadata();

}
```

1. ScannedGenericBeanDefinition类继承了GenericBeanDefinition类，并实现了AnnotatedBeanDefinition接口，这个类是基于ASM ClassReader获取注解信息的，ComponentScanAnnotationParser扫描类时将@Component或@Component为元注解的注解（如@Service、@Controller）的类解析成这个ScannedGenericBeanDefinition。 扫描过程具体参考：

```Java
org.springframework.context.annotation.ComponentScanAnnotationParser#parse
org.springframework.context.annotation.ClassPathBeanDefinitionScanner#scan
org.springframework.context.annotation.ComponentScanBeanDefinitionParser#parse

 -org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan
  -org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#findCandidateComponents
   -org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#scanCandidateComponents
```

2. ConfigurationClassBeanDefinition，ConfigurationClassBeanDefinitionReader内部静态类，继承了RootBeanDefinition，并实现了AnnotatedBeanDefinition接口，Configuration Class内定义的Bean会被解析成ConfigurationClassBeanDefinition。

3. AnnotatedGenericBeanDefinition 主要用于测试希望在AnnotatedBeanDefinition上做操作的代码，比如Spring组件扫描支持中的策略实现。另外AnnotatedGenericBeanDefinition也用来描述@Configuration类。

```Java
	private void registerBeanDefinitionForImportedConfigurationClass(ConfigurationClass configClass) {
		AnnotationMetadata metadata = configClass.getMetadata();
		AnnotatedGenericBeanDefinition configBeanDef = new AnnotatedGenericBeanDefinition(metadata);
		...
	}
```


[1]. [Spring-IOC RootBeanDefinition源码分析](https://www.cnblogs.com/leihuazhe/p/9481124.html)





