---
title: Go-第一等公民function
date: 2018-11-22 14:18:06
tags: ["Go"]
---

## 什么是一等公民函数

一个语言如果支持将函数赋值给变量，作为参数传递，且能作为其他函数的返回值，我们就说函数在这个语言中是一等公民。Go语言支持一等公民函数。

## 匿名函数

在定义函数的时候不为其提供函数名，只是将其作为值赋值给变量或者作为参数或返回值直接返回，这类函数叫做匿名函数。匿名函数的使用方法和普通函数一样，在其后面加上括号和参数。

```Go
package main

import "fmt"

func main() {
	a := func() {
		fmt.Println("Hello Anonymous FunctioN!")
	}

	a()
}
```

匿名函数也可以不通过赋值进行调用:

```Go
package main

import "fmt"

func main() {
	func() {
		fmt.Println("Hello Anonymous FunctioN!")
	}()
}
```


## 用户定义函数类型

就像我们可以自定义struct类型一样，我们也可以定义函数类型。

```Go
type add func(a int, b int) int  
```

上面的代码定义了一个函数类型add，其接受两个int类型参数并返回int值。这样我们就可以定义add类型的变量了。


```Go
package main

import (  
    "fmt"
)

type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}
```


## 高阶函数

高阶函数需要满足一下两个条件之一：
* 使用一个活或多个函数作为参数
* 返回函数


### 将函数作为参数传递给其他函数

```Go
package main

import (  
    "fmt"
)

func simple(a func(a, b int) int) {  
    fmt.Println(a(60, 7))
}

func main() {  
    f := func(a, b int) int {
        return a + b
    }
    simple(f)
}
```

### 从其他函数返回函数

```Go
package main

import (  
    "fmt"
)

func simple() func(a, b int) int {  
    f := func(a, b int) int {
        return a + b
    }
    return f
}

func main() {  
    s := simple()
    fmt.Println(s(60, 7))
}
```


### 闭包

闭包是一种特殊的匿名函数。闭包是能够访问定义在函数体外的变量的匿名函数。

```Go
package main

import (  
    "fmt"
)

func main() {  
    a := 5
    func() {
        fmt.Println("a =", a)
    }()
}
```

如上面的例子，我们定义了一个匿名函数，它能够访问到其函数体外的变量a，因此这个匿名函数式闭包。每个闭包被绑定到它自己周围的变量。

```Go
package main

import (  
    "fmt"
)

func appendStr() func(string) string {  
    t := "Hello"
    c := func(b string) string {
        t = t + " " + b
        return t
    }
    return c
}

func main() {  
    a := appendStr()
    b := appendStr()
    fmt.Println(a("World"))
    fmt.Println(b("Everyone"))

    fmt.Println(a("Gopher"))
    fmt.Println(b("!"))
}
```

在上面的例子中appendStr函数返回一个闭包。这个闭包绑定到变量t上。闭包a和b都绑定到t上，但他们有自己的t值。我们首先调用了a并给其传递参数World，那么a版本的t的值为Hello World。然后我们使用参数Everyone调用了闭包b，b有自己的变量t，b的t值的初始值是Hello，在调用结束后，b版本的t的职位Hello Everyone。函数最后执行结果为：

```
Hello World  
Hello Everyone  
Hello World Gopher  
Hello Everyone !
```

## 一等函数的实战

我们写一个程序基于一些条件过滤student slice。

```Go
package main

import "fmt"

type student struct {
	firstName string
	lastName  string
	grade     string
	country   string
}

func filter(students []student, f func(s student) bool) []student {
	var result []student
	for _, s := range students {
		if f(s) {
			result = append(result, s)
		}
	}
	return result
}

func main() {
	s1 := student{
		firstName: "Naveen",
		lastName:  "Ramanathan",
		grade:     "A",
		country:   "India",
	}
	s2 := student{
		firstName: "Samuel",
		lastName:  "Johnson",
		grade:     "B",
		country:   "USA",
	}
	s := []student{s1, s2}
	f := filter(s, func(s student) bool {
		if s.grade == "B" {
			return true
		}
		return false
	})
	fmt.Println(f)
}
```