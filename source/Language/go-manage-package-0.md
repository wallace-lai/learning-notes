# 【Go】如何组织自己的包

作者：wallace-lai</br>
时间：2023-07-19</br>
更新：2023-07-19</br>

## 创建工程目录并使用go mod初始化
假设工程目录叫demo，创建demo目录并使用go mod进行初始化。初始化成功后目录下会多一个go.mod文件。

```shell
~/workspace$ mkdir demo
~/workspace$ cd demo
~/workspace/demo$ go mod init example.com/demov1
go: creating new go.mod: module example.com/demov1
~/workspace/demo$ 
~/workspace/demo$ ls
go.mod
~/workspace/demo$ 
```

## 组织自己的package
接下来我们编写属于自己的名为calc的package，如下所示：创建calc子目录；在子目录下分别创建calc.go和tools.go源文件。在源文件中编写我们的代码，注意包名为calc。

```go
// calc.go
package calc

func Add(x, y int) int {
        return x + y
}

func Sub(x, y int) int {
        return x - y
}

func add(x, y int) int {
        return Add(x, y)
}
```

```go
// tools.go
package calc

func Mul(x, y int) int {
        return x * y
}
```

至此，calc包就编写完成了。我们回到工程目录demo下创建main.go以调用calc子包中的方法，如下所示。注意在导calc包时需要加上工程目录名作为前缀，然后在引用calc包中方法时需要用`calc.method()`的形式。
```go
// main.go
package main

import (
        "fmt"
        "demo/calc"     // note
)

func main() {
        sum := calc.Add(1, 2)   // note
        mul := calc.Mul(3, 4)

        fmt.Println(sum)
        fmt.Println(mul)
}
```

最终，整个工程的文件组织形式如下所示。

```shell
~/workspace/demo$ tree .
.
├── calc
│   ├── calc.go
│   └── tools.go
├── go.mod
└── main.go

1 directory, 4 files
~/workspace/demo$ 
```

以上便是在Go语言中组织子包的一个小案例。