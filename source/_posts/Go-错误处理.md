---
title: 'Go: 错误处理'
date: 2018-11-21 14:55:21
tags: ["Go"]
---

# 什么是错误？

错误表示程序中的异常情况。比如在打开文件的时候，如果文件不存在，这就是个异常情况，使用错误来表示。

Go中的错误是普通的旧值。使用内置的error类型表示。就像其他内置的类型一样，错误值可以存储在变量中，作为函数的返回值等。

```Go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}
```
在上面的例子中，尝试打开文件/test.txt，os包中的Open函数的签名为：
```
func Open(name string) (file *File, err error)
```

如果文件成功打开，那么Open函数将返回文件句柄，error将为nil，如果打开文件是出错了，一个非nil的错误将被返回。如果函数或方法返回错误，那么按照惯例，错误将作为最后一个返回值返回。

Go中处理错误的管用方式是将返回的错误与nil比较。nil表示没有错误出现，非nil表示有错误出现。


# 错误类型的表示

error类型是一个接口类型，定义如下：
```Go
type error interface {  
    Error() string
}
```

error接口只包含一个Erorr()string方法，任何实现了该接口的类型都可以作为错误使用。调用fmt.Println函数打印错误的时候，内部将调用Error()string方法获取错误的描述。

# 从error中提取更多信息的不同方式

我们既然知道error是个接口类型，那么如何从错误中提取更多信息。

## 1. 断言底层struct类型从struct字段中获取更多信息

Open函数返回的错误类型为*PathError，PathError是一个结构类型，定义如下：

```
type PathError struct {  
    Op   string
    Path string
    Err  error
}

func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }
```

我们可以在代码中断言error类型是不是* os.PathError,以获取Path和Op字段：

```Go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    if err, ok := err.(*os.PathError); ok {
        fmt.Println("File at path", err.Path, "failed to open")
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}
```

## 2. 断言底层struct类型通过方法中获取更多信息

第二个获取更多错误信息的方式是通过调用底层错误类型的方法。比如DNSError结构类型，它定义了TimeOut,Temporary等方法，可以调用这些方法获取更多的错误信息。

```Go
package main

import (  
    "fmt"
    "net"
)

func main() {  
    addr, err := net.LookupHost("golangbot123.com")
    if err, ok := err.(*net.DNSError); ok {
        if err.Timeout() {
            fmt.Println("operation timed out")
        } else if err.Temporary() {
            fmt.Println("temporary error")
        } else {
            fmt.Println("generic error: ", err)
        }
        return
    }
    fmt.Println(addr)
}
```

## 3. 直接比较

第三个获取更多错误信息的方式是通过与一个error类型的变量直接比较。

filepath包中的Glob函数用于返回匹配模式的所有文件的名称。这个函数在模式异常时会返回一个ErrBadPattern错误。


ErrBadPattern定义在filepath中：
```Go
var ErrBadPattern = errors.New("syntax error in pattern")  
```

ErrBadPattern在模式异常时被Glob函数返回。可以通过比较检查这个错误信息：
```Go
package main

import (  
    "fmt"
    "path/filepath"
)

func main() {  
    files, error := filepath.Glob("[")
    if error != nil && error == filepath.ErrBadPattern {
        fmt.Println(error)
        return
    }
    fmt.Println("matched files", files)
}
```


# 不要忽略错误

绝对不要忽略错误。忽略错误将引来麻烦。