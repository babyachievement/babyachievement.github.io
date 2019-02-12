---
title: Spring-Security-Authentication
date: 2018-11-28 10:36:50
tags:
---


org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter#configure(org.springframework.security.config.annotation.web.builders.HttpSecurity)

通过FormLoginConfigurer将UsernamePasswordAuthenticationFilter添加到HttpSecurity的filter中
通过ExpressionUrlAuthorizationConfigurer将FilterSecurityInterceptor添加到HttpSecurity的filter中

org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter#getHttp

将WebAsyncManagerIntegrationFilter，SecurityContextPersistenceFilter,HeaderWriterFilter,LogoutFilter,RequestCacheAwareFilter,SecurityContextHolderAwareRequestFilter,AnonymousAuthenticationFilter,SessionManagementFilter,ExceptionTranslationFilter天骄到HttpSecurity的filter列表中。


WebSecurityConfigurerAdapter的以下两个方法：

```Java
protected void configure(HttpSecurity http) throws Exception {
		logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");

		http
			.authorizeRequests() // 应用ExpressionUrlAuthorizationConfigurer
				.anyRequest().authenticated()
				.and()
			.formLogin().and() // 应用FormLoginConfigurer
			.httpBasic();
	}
```

```Java
protected final HttpSecurity getHttp() throws Exception {
		if (http != null) {
			return http;
		}

		DefaultAuthenticationEventPublisher eventPublisher = objectPostProcessor
				.postProcess(new DefaultAuthenticationEventPublisher());
		localConfigureAuthenticationBldr.authenticationEventPublisher(eventPublisher);

		AuthenticationManager authenticationManager = authenticationManager();
		authenticationBuilder.parentAuthenticationManager(authenticationManager);
		authenticationBuilder.authenticationEventPublisher(eventPublisher);
		Map<Class<? extends Object>, Object> sharedObjects = createSharedObjects();

		http = new HttpSecurity(objectPostProcessor, authenticationBuilder,
				sharedObjects);
		if (!disableDefaults) {
			// @formatter:off
			http
				.csrf().and()
				.addFilter(new WebAsyncManagerIntegrationFilter()) //WebAsyncManagerIntegrationFilter
				.exceptionHandling().and()	// ExceptionTranslationFilter
				.headers().and()	// HeaderWriterFilter
				.sessionManagement().and()	// SessionManagementFilter
				.securityContext().and()	// SecurityContextPersistenceFilter
				.requestCache().and()	//RequestCacheAwareFilter
				.anonymous().and()	//AnonymousAuthenticationFilter
				.servletApi().and()	//SecurityContextHolderAwareRequestFilter
				.apply(new DefaultLoginPageConfigurer<>()).and()//DefaultLoginPageGeneratingFilter和DefaultLogoutPageGeneratingFilter
				.logout();// LogoutFilter
			// @formatter:on
			ClassLoader classLoader = this.context.getClassLoader();
			List<AbstractHttpConfigurer> defaultHttpConfigurers =
					SpringFactoriesLoader.loadFactories(AbstractHttpConfigurer.class, classLoader);

			for (AbstractHttpConfigurer configurer : defaultHttpConfigurers) {
				http.apply(configurer);
			}
		}
		configure(http);
		return http;
	}
```
HttpSecurity最终会用上面的那些filter构建一个DefaultSecurityFilterChain，在构建DefaultSecurityFilterChain前，会使用FilterComparator对filter进行排序。
HttpSecurity会添加到WebSecurity的securityFilterChainBuilders中，被用来构建成一个FilterChainProxy，这个FilterChainiProxy在WebSecurityConfiguration中被注册到ApplicationContext中，名称为springSecurityFilterChain。

AbstractSecurityWebApplicationInitializer在onStartup时，调用insertSpringSecurityFilterChain，根据bean名称springSecurityFilterChain获取到，然后用其创建DelegatingFilterProxy，最终添加到ServletContext的

