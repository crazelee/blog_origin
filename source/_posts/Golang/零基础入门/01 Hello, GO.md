---
title: 【Golang零基础入门】01 Hello，Go
date: 2019-03-06 10:59:08
categories: [Golang, 零基础入门]
tags:
- Golang
- Golang零基础入门
copyright: ture
---

## 一、第一个Go程序：hello.go

```go
package main

import "fmt"

func main(){
	fmt.Println("Hello, Go")
}
```

<!--more-->

## 二、go run 一次性调试

```go
$ go run hello.go
Hello, Go
```

**说明：**
1. Go是一门编译型语言，Go语言的工具链将源代码及其依赖转换成计算机的机器指令。
2. go run 编译一个或多个以.go结尾的源文件，链接库文件，并运行最终生成的可执行文件。

## 三、go build 直接生成可执行程序
如果不只是一次的调试，你可以编译这个程序，保存编译结果以备将来之用。可以用build子命令，将直接生成可执行文件 `hello`，之后直接运行hello可执行文件，结果与go run时是一致的：

```go
$ go build hello.go
$ ./hello
Hello, GO
```

## 四、一些知识点

1. Go语言通过包（package）组织，类似其他语言的库、模块、命名空间。每个源文件都以一条 package 声明语句开始，这个例子里就是 package main 

2. main 包比较特殊。它定义了一个独立可执行的程序，而不是一个库。在 main 里的 main 函数也很特殊，它是整个程序执行时的入口

3. 使用（import）来导入别的包，` import "fmt" `

4. gofmt 工具把代码格式化为标准格式，最好在每次提交Git前执行一次，保证代码风格一致

5. go clean 移除当前源码包里编译生成的文件，如test.out

6. 按照惯例，最好每个包的包声明前添加注释

 ```go
// package for app's config
package config
```

7. 空标识符（_下划线）类似垃圾箱，将不需要的变量丢弃，如下，只需要range的value，不需要索引

 ```go
for _, arg := range args {
    // todo
}
```

