---
title: Spring-3.1-Loading BeanDefinitions
date: 2018-12-28 17:56:11
tags: ["Spring"]
---
## BeanDefinition的自动加载
　　　　一种是从XML中解析，二是注解自动扫描，三是显式创建并注册到BeanDefinitionRegistry中。

### XML 中BeanDefinition加载
　　　　BeanDefinition的加载，由《Spring-2-ApplicationContext》中知道obtainFreshBeanFactory会调用refreshBeanFactory和getBeanFactory方法，refreshBeanFactory方法由子类实现，实现了该方法的子类有两个：AbstractRefreshableApplicationContext和GenericApplicationContext，GenericApplicationContext的实现只是设置了序列化ID，并不提供自动扫描，这里只介绍AbstractRefreshableApplicationContext的实现。AbstractRefreshableApplicationContext的实现中调用了loadBeanDefinitions方法，该loadBeanDefinitions方法为抽象方法，将执行真正的BeanDefinition加载，具体又由它的子类去实现。AbstractRefreshableApplicationContext的子类AbstractXmlApplicationContext，AnnotationConfigWebApplicationContext，GroovyWebApplicationContext和XmlWebApplicationContext实现了该方法，而加载XML中BeanDefinition的是AbstractXmlApplicationContext和XmlWebApplicationContext子类。


　　　　AbstractXmlApplicationContext和XmlWebApplicationContext都是通过beanDefinitionReader去加载BeanDefinition，不同的是XmlWebApplicationContext只从configLocations中读取XML，而AbstractXmlApplicationContext还会尝试从configResources读取，其实最终都会通过Resource加载XML文件。


```Java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
	// Create a new XmlBeanDefinitionReader for the given BeanFactory.
	XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

	// Configure the bean definition reader with this context's
	// resource loading environment.
	beanDefinitionReader.setEnvironment(getEnvironment());
	beanDefinitionReader.setResourceLoader(this);
	// EntityResolver用于获取XSD和DTD文件，来验证Element
	beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

	// Allow a subclass to provide custom initialization of the reader,
	// then proceed with actually loading the bean definitions.
	initBeanDefinitionReader(beanDefinitionReader);
	loadBeanDefinitions(beanDefinitionReader);
}
```

```Java
AbstractXmlApplicationContext#loadBeanDefinitions(org.springframework.beans.factory.xml.XmlBeanDefinitionReader)
 AbstractBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.Resource...)
  XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.Resource)
   XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.support.EncodedResource)
    XmlBeanDefinitionReader#doLoadBeanDefinitions(InputSource inputSource, Resource resource)
     XmlBeanDefinitionReader#registerBeanDefinitions(Document doc, Resource resource)
      DefaultBeanDefinitionDocumentReader#registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
       DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions(Element root) 
        DefaultBeanDefinitionDocumentReader#parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate)
```

　　　　在parseBeanDefinitions方法中就涉及到XML namespace处理器了,如果Element的namespaceUri为空或者为[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)则按照默认的NameSpace去处理Element，默认会处理的标签有beans、alias、bean和import；否则则会调用BeanDefinitionParserDelegate的parseCustomElement方法，尝试获取NamespaceHandler去解析XML Element。

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
	if (delegate.isDefaultNamespace(root)) {
		NodeList nl = root.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (node instanceof Element) {
				Element ele = (Element) node;
				
				if (delegate.isDefaultNamespace(ele)) {
					parseDefaultElement(ele, delegate);
				}
				else {
					delegate.parseCustomElement(ele);
				}
			}
		}
	}
	else {
		delegate.parseCustomElement(root);
	}
}
```


BeanDefinitionParserDelegate的parseCustomElement方法:
```Java
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
	String namespaceUri = getNamespaceURI(ele);
	if (namespaceUri == null) {
		return null;
	}
	NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
	if (handler == null) {
		error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
		return null;
	}
	return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```
　　　　自定义的NamespaceHandler是通过"META-INF/spring.handlers"文件注册的,如下图中的SofaBootNamespaceHandler。，用户通过继承NamespaceHandlerSupport类就能自定义NamespaceHandler。

{% asset_img NameSpaceHandler.png SofaBootNamespaceHandler %}

{% asset_img SofaBootNamespaceHandler.png SofaBootNamespaceHandler %}


　　　　在解析XML中的元素时涉及到XML元素验证，DTD验证文件由BeansDtdResolver加载，XSD文件由PluggableSchemaResolver加载，XSD验证文件是在"META-INF/spring.shemas"属性文件中声明的，key为systemId，value为文件路径。


 
### 注解自动扫描

　　　　AnnotationConfigWebApplicationContext实现了AbstractRefreshableApplicationContext.loadBeanDefinitions的方法，在该方法中调用ClassPathBeanDefinitionScanne.scan扫描包。

　　　　ConfigurationClassPostProcessor实现了BeanDefinitionRegistryPostProcessor和BeanFactoryPostProcessor接口，在postProcessBeanDefinitionRegistry和postProcessBeanFactory中会调用ConfigurationClassParser解析@Configuration注解标注的类，最终会通过ComponentScanAnnotationParser调用到ClassPathBeanDefinitionScanner的doScan。此外会调用 ConfigurationClassBeanDefinitionReader的loadBeanDefinitionsForConfigurationClass处理Configuration中BeanMethod等生成ConfigurationClassBeanDefinition。


```Java
ClassPathBeanDefinitionScanner#doScan(String... basePackages)
 ClassPathScanningCandidateComponentProvider#findCandidateComponents(String basePackage)
  // 扫描@Component，并生成ScannedGenericBeanDefinition
  ClassPathScanningCandidateComponentProvider#scanCandidateComponents

```

### 显式注册

org.springframework.beans.factory.support.BeanDefinitionRegistry#registerBeanDefinition