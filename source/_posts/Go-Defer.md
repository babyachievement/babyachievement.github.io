---
title: 'Go: Defer'
date: 2018-11-21 14:21:29
tags: ["Go"]
---

## Defer是什么？

Defer语句用于在所在的函数执行return之前调用函数。类似于Java中finally的作用。

```Go
package main

import (
	"fmt"
)

func finished() {
	fmt.Println("Finished finding largest")
}

func largest(nums []int) {
	defer finished()
	fmt.Println("Started finding largest")
	max := nums[0]
	for _, v := range nums {
		if v > max {
			max = v
		}
	}
	fmt.Println("Largest number in", nums, "is", max)
}

func main() {
	nums := []int{78, 109, 2, 563, 300}
	largest(nums)
}
```

上面的例子中largest函数的第一行包含了defer finished()，这条语句将在largest函数执行结束前调用。该例子的执行结果为：

```
Started finding largest
Largest number in [78 109 2 563 300] is 563
Finished finding largest
```

## Defer 方法

Defer的作用不限于函数，对于方法调用同样适用。

```Go
package main

import (
	"fmt"
)

type person struct {
	firstName string
	lastName string
}

func (p person) fullName() {
	fmt.Printf("%s %s",p.firstName,p.lastName)
}

func main() {
	p := person {
		firstName: "John",
		lastName: "Smith",
	}
	defer p.fullName()
	fmt.Printf("Welcome ")
}
```


## 参数计算

defer函数的参数是在defer语句执行的时候计算，而不是在实际函数调用结束时计算：

```Go
package main

import (  
    "fmt"
)

func printA(a int) {  
    fmt.Println("value of a in deferred function", a)
}
func main() {  
    a := 5
    defer printA(a)
    a = 10
    fmt.Println("value of a before deferred function call", a)

}
```

上面的例子的输出结果为：
```
value of a before deferred function call 10
value of a in deferred function 5
```

可以看到printA调用输出并不是10。尽管a的值在执行完defer语句后变成了10，延迟调用printA函数的输出仍然为5。

## defer栈

当一个函数有多个defer调用时，它们会被添加到一个栈中，按照后进先出（LIFO）的顺序执行。

```Go
package main

import (  
    "fmt"
)

func main() {  
    name := "Naveen"
    fmt.Printf("Orignal String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}
```

执行结果：
```
Orignal String: Naveen  
Reversed String: neevaN  
```

## defer的实际运用

defer用于那些无论代码流如何执行都需要执行的函数。


```Go
package main

import (  
    "fmt"
    "sync"
)

type rect struct {  
    length int
    width  int
}

func (r rect) area(wg *sync.WaitGroup) {  
    defer wg.Done()
    if r.length < 0 {
        fmt.Printf("rect %v's length should be greater than zero\n", r)
        return
    }
    if r.width < 0 {
        fmt.Printf("rect %v's width should be greater than zero\n", r)
        return
    }
    area := r.length * r.width
    fmt.Printf("rect %v's area %d\n", r, area)
}

func main() {  
    var wg sync.WaitGroup
    r1 := rect{-67, 89}
    r2 := rect{5, -67}
    r3 := rect{8, 9}
    rects := []rect{r1, r2, r3}
    for _, v := range rects {
        wg.Add(1)
        go v.area(&wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}

```

上面的例子中area函数无论走那个代码分支都需要执行wg.Done()函数，将其通过defer延迟调用，即使以后添加新的判断分支，也不会影响到wg.Done()函数的执行。