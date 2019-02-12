---
title: Thymeleaf
date: 2018-11-27 14:58:54
tags: ["Java"]
--- 

## Thymeleaf标准表达式语法

Thymeleaf有五种语法：

* ${} :变量表达式
* *{} :选择表达式
* #{} :消息（i8n）表达式
* @{} :链接表达式
* ~{} :片段表达式

### 变量表达式

变量表达式是OGNL表达式，或者Spring EL，在上下文变量中执行，也称作模型属性。写法如下：
```Java
${session.user.name}
```

变量表达式可以作为标签属性：
```HTML
<span th:text="${book.author.name}">
```
上面的写法等同于：
```
((Book)context.getVariable("book")).getAuthor().getName()
```