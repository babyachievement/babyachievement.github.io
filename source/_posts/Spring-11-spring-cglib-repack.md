---
title: Spring-11-spring-cglib-repack
date: 2019-02-12 15:56:36
tags: ["Spring", "AOP", "CGLIB"]
---

在读Spring源码时看到了Spring代码中有一个Enhancer类，Spring使用CGLIB库了，为什么还要有Enhancer类？难道不会与CGLIB类冲突吗？带着以上两个疑问通读了一下S平日你改的Enhancer类，发现在setSuperclass中调用了setContextClass方法，在其周围发现了"SPRING PATCH"注释，该方法在Spring自定义的另一个类AbstractClassGenerator中添加，并且在其周围同样发现了"SPRING PATCH"注释。全局搜索了一下"SPRING PATCH"，发现在多个地方都有该注释：


{% asset_img Spring_Patch.png SPRING PATCH %}

从字面上理解是Spring添加的patch，用来修复什么呢？先看一下AbstractClassGenerator中contextClass字段，该属性在generate方法中用到，用作ReflectUtils.defineClass的参数，看了一下defineClass方法的注释：“on JDK 9”，恍然大悟，原来是为了兼容JDK9，并且在其git commit中也有注释：“MethodHandles.Lookup.defineClass for CGLIB class definition purposes”。

ReflectUtils的defineClass方法定义如下；

```
public static Class defineClass(String className, byte[] b, ClassLoader loader,
			ProtectionDomain protectionDomain, Class<?> contextClass) throws Exception {

	Class c = null;
	if (contextClass != null && privateLookupInMethod != null && lookupDefineClassMethod != null) {
		try {
			MethodHandles.Lookup lookup = (MethodHandles.Lookup)
					privateLookupInMethod.invoke(null, contextClass, MethodHandles.lookup());
			c = (Class) lookupDefineClassMethod.invoke(lookup, b);
		}
		catch (InvocationTargetException ex) {
			Throwable target = ex.getTargetException();
			if (target.getClass() != LinkageError.class && target.getClass() != IllegalArgumentException.class) {
				throw new CodeGenerationException(target);
			}
			// in case of plain LinkageError (class already defined)
			// or IllegalArgumentException (class in different package):
			// fall through to traditional ClassLoader.defineClass below
		}
		catch (Throwable ex) {
			throw new CodeGenerationException(ex);
		}
	}
	if (protectionDomain == null) {
		protectionDomain = PROTECTION_DOMAIN;
	}
	if (c == null) {
		if (classLoaderDefineClassMethod != null) {
			Object[] args = new Object[]{className, b, 0, b.length, protectionDomain};
			try {
				if (!classLoaderDefineClassMethod.isAccessible()) {
					classLoaderDefineClassMethod.setAccessible(true);
				}
				c = (Class) classLoaderDefineClassMethod.invoke(loader, args);
			}
			catch (InvocationTargetException ex) {
				throw new CodeGenerationException(ex.getTargetException());
			}
			catch (Throwable ex) {
				throw new CodeGenerationException(ex);
			}
		}
		else {
			throw new CodeGenerationException(THROWABLE);
		}
	}
	// Force static initializers to run.
	Class.forName(className, true, loader);
	return c;
}
```

MethodHandles.privateLookupIn和MethodHandles.Lookup.defineClass方法都是Java 9以后才有的，所以为了调用Lookup中的defineClass才在一些类中添加了patch代码。自然，自定义了一些类为了防止冲突，Spring在spring-core.gradle中定义了cglibRepackJar任务，改任务会将net.sf.cglib打包成spring-cglib-repack，打包spring-core jar时会根据添加了patch类，将spring-cglib-repack中相应的类排除掉。


```
jar {
	// Inline repackaged cglib classes directly into spring-core jar
	dependsOn cglibRepackJar
	from(zipTree(cglibRepackJar.archivePath)) {
		include "org/springframework/cglib/**"
		exclude "org/springframework/cglib/core/AbstractClassGenerator*.class"
		exclude "org/springframework/cglib/core/AsmApi*.class"
		exclude "org/springframework/cglib/core/KeyFactory.class"
		exclude "org/springframework/cglib/core/KeyFactory\$*.class"
		exclude "org/springframework/cglib/core/ReflectUtils*.class"
		exclude "org/springframework/cglib/proxy/Enhancer*.class"
		exclude "org/springframework/cglib/proxy/MethodProxy*.class"
	}

	dependsOn objenesisRepackJar
	from(zipTree(objenesisRepackJar.archivePath)) {
		include "org/springframework/objenesis/**"
	}
}
```