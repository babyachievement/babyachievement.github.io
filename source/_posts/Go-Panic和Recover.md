---
title: Go-Panic和Recover
date: 2018-11-22 11:35:44
tags: ["Go"]
---

Go中处理异常的通用方式是使用error。Error对于程序中的大部分异常是足够用的，但是对于处理程序中出现导致其无法再继续执行的异常则是不够的。这种情景下我们使用panic终止程序。当函数碰到panic时，它将终止执行，任何延迟的函数将会执行，然后将控制权返回给它的调用者。程序会在当前goroutine的所有函数返回后打出panic信息和stack trace，然后终止程序。

可以使用recover重新获取到panic程序的控制权。panic和recover可以看做Java中的try-catch-finally。

## 什么时候使用panic?

尽可能的使用errors，而不是panic和recover。只有在程序无法继续运行的时候再使用panic和recover机制。

panic有两种使用场景：

1. 当一个不可恢复的异常发生的时候，程序不能在继续执行。一个例子就是web服务器无法绑定端口。
2. 程序员错误。比如一个方法接受指针参数，别人却使用nil作为参数，这种情况下，我们可以使用panic作为调用方法的错误。


## Panic例子

```Go
package main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

panic会先打印出错误信息，然后将调用栈打出来。

```
panic: runtime error: last name cannot be nil

goroutine 1 [running]:
main.fullName(0xc000071f68, 0x0)
	E:/git_projects/virtualworks/src/programming/panict/panict.go:12 +0x14c
main.main()
	E:/git_projects/virtualworks/src/programming/panict/panict.go:20 +0x54
```

### Defer while panicking

当发生panic的时候，defer调用会怎样？正如之前所说，function遇到panic的时候，会将先将函数内的defer调用执行结束，然后将控制权返回给调用者，直到当前goroutine最顶层调用执返回之后，再打印出panic信息和调用堆栈。


```Go
package main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    defer fmt.Println("deferred call in fullName")
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

输出
```
deferred call in fullName  
deferred call in main  
panic: runtime error: last name cannot be nil

goroutine 1 [running]:  
main.fullName(0x1042bf90, 0x0)  
    /tmp/sandbox060731990/main.go:13 +0x280
main.main()  
    /tmp/sandbox060731990/main.go:22 +0xc0
```


## Recover

recover是内置的函数用于重新获取出现panic的gorountine的控制权。

recover只有在defer函数的内部调用才有效。在一个defer函数内部调用recover将恢复执行并获取错误信息来会停止panic序列，如果在defer函数外调用recover，并不能停止panic序列。

```Go
package main

import (  
    "fmt"
)

func recoverName() {  
    if r := recover(); r!= nil {
        fmt.Println("recovered from ", r)
    }
}

func fullName(firstName *string, lastName *string) {  
    defer recoverName()
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

recoverName函数调用了recover，返回了传给panic的值。当fullName panic的时候，defer函数recoverName将被调用，其调用recover函数停止了panic队列。

程序输出为：
```
recovered from  runtime error: last name cannot be nil  
returned normally from main  
deferred call in main  
```


## Panic, Recover 和 Goroutines

recover只有在相同的goroutine内部调用时才会起作用。无法从其他goroutine内发生的panic恢复。

## 运行时panic

panic也能会因为运行时错误导致，比如数组越界。运行时panic等同于使用runtime.Error类型的参数调用panic。runtime.Error是内置的类型，其实现了error接口。

```Go
type Error interface {  
    error
    // RuntimeError is a no-op function but
    // serves to distinguish types that are run time
    // errors from ordinary errors: a type is a
    // run time error if it has a RuntimeError method.
    RuntimeError()
}
```

```Go
package main

import (  
    "fmt"
)

func a() {  
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}
func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

上面例子执行结果：
```
panic: runtime error: index out of range

goroutine 1 [running]:  
main.a()  
    /tmp/sandbox780439659/main.go:9 +0x40
main.main()  
    /tmp/sandbox780439659/main.go:13 +0x20
```

和其他panic一样，运行时panic也能恢复。


```Go
package main

import (  
    "fmt"
)

func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
    }
}

func a() {  
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```


## 恢复后如何获取调用栈

调用recover后，会失去panic调用栈，但可以使用Debug.PringStack函数获取调用栈。


```Go
package main

import (  
    "fmt"
    "runtime/debug"
)

func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
        debug.PrintStack()
    }
}

func a() {  
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```