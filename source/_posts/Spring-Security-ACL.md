---
title: 'Spring Security: ACL'
date: 2018-11-26 20:04:00
tags: ["Spring-Security"]
---

ACL(Access Control List)，中文翻译为访问控制列表，用以控制对象的访问权限。为什么要有ACL?在复杂的应用中，我们不仅要知道对谁进行授权（Authentication），还要知道在什么地方（MethodInvocation），以及对什么对象（SomeDomainObject）进行权限认证。

## lookupStrategy
lookupStrategy的作用就是从数据库中读取信息，把这些信息提供给aclService使用，所以我们要为它配置一个dataSource，配置中还可以看到一个aclCache，这就是上面我们配置的缓存，它会把资源最大限度的利用起来。

中间一部分可能会让人感到困惑，为何一次定义了三个adminRole呢？这是因为一旦acl信息被保存到数据库中，无论是修改它的从属者，还是变更授权，抑或是修改其他的ace信息，都需要控制操作者的权限，这里配置的三个权限将对应于上述的三种修改操作，我们把它配置成，只有ROLE_ADMIN才能执行这三种修改操作。

## PermissionEvaluator 
