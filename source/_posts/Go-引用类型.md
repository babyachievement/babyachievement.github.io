---
title: Go-引用类型
date: 2019-01-21 17:54:51
tags: ["Go"]
---

Go语言里的引用类型有如下几个：切片、映射、通道、接口和函数类型。当声明上述类型的变量时，创建的变量被称为标头（header）值。从技术细节上说，字符串也是一种引用类型。每个应用类型创建的标头值是包含一个执行底层数据结构的指针。每个引用类型还包含一组独特的字段，用于管理底层数据结构。因为标头值是为复制而设计的，所以用与按不需要共享一个引用类型的值。标头值里包含一个指针，因此通过复制来传递一个应用类型的值的副本，本质上就是在共享底层的数据结构。

```Go
type IP []type
```

上述代码展示了一个名为IP的类型，这个类型被声明为字节切片。当腰围绕相关的内置类型或者应用类型来声明用户自定义的行为时，直接基于已有类型来声明用户定义的类型会很好用。编译器只允许为命名的用户定义的类型声明方法，如下代码所示：

```Go
func (ip IP) MarshalText() ([]byte, error) {
	if len(ip) == 0 {
		return []byre(""), nil
	}
	
	if len(ip) != IPv4len && len(ip) != IPv6len {
		return nil, errors.New("invalid IP address")
	}
	
	return []byte(ip.String()), nil
}
```

上述代码里定义的MarshalText方法是用IP类型的值接收者声明的。一个值接受者，正像预期的那样通过复制来传递引用，从而不需要通过指针来共享应用类型的值。这种传递方法也可以应用到函数或者方法的参数传递中，以及返回值中。

```Go
func ipEmtpyString(ip IP) string {
	if len(ip) == 0 {
		return ""
	}
	
	return ip.String()
}
```