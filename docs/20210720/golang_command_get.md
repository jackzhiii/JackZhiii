### 命令介绍

* **bug:** 是用来给官方发布bug报告的一个快速命令

* **build:** 编译包和依赖

  * usage: go build [-o output] [build flags] [package]

  * flags:

    * -a: 强制构建已经是最新的包

    * -n:把需要执行的编译命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的

    * -p n: 指定可以并行可运行的编译数目，默认是CPU数目

    * -race: 允许检测程序数据竞争

    * -msan: 启用与内存消毒剂的相互操作

    * -asmflags '[pattern=]arg list': 传递每个go工具asm调用的参数

    * -buildmode mode: 编译模式

    * -compiler name：指定所使用的编译器

    * -gccgoflags '[pattern=]arg list'：gccgo 编译/链接器参数

    * -gcflags '[pattern=]arg list'：垃圾回收参数

    * -ldflags '[pattern=]arg list'：

      * ```go
        '-s -w': 压缩编译后的体积
        ```

      * ```go
        -s: 去掉符号表
        ```

      * ```go
        -w: 去掉调试信息，不能gdb调试了
        ```



##### 跨平台编译

​	编译跨平台的只需要修改`GOOS`、`GOARCH`、`CGO_ENABLED`三个环境变量即可

- GOOS: 目标平台的操作系统(darwin, freebsd, linux, windows)

- GOARCH: 目标平台的体系架构32位还是64位(386、amd64、arm)

- 交叉编译不支持 CGO 所以要禁用它

- **Window环境举例**

  - Window环境下编译 Mac 和 Linux 64位可执行程序

    ```go
    SET CGO_ENABLED=0
    SET GOOS=darwin
    SET GOARCH=amd64
    go build main.go
    
    SET CGO_ENABLED=0
    SET GOOS=linux
    SET GOARCH=amd64
    go build main.go
    ```

* ### Mac环境举例

  * Mac 下编译 Linux 和 Windows 64位可执行程序

    ```go
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
    CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
    ```

* ### Linux环境举例

  * Linux 下编译 Mac 和 Windows 64位可执行程序

    ```go
    CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
    CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
    ```

    