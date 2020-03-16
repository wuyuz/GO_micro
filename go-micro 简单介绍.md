## go-micro 简单介绍



#### go-mod 介绍

GO1.11MODULE可以设置为三个字符串值之一：off，on或auto（默认值）。

- off模式

  ```
  off 则go命令从不使用新模块支持。它查找vendor 目录和GOPATH以查找依赖关系;也就是继续使用“GOPATH模式”。
  ```

- on模式：

  ```
  on 则go命令需要使用模块，go 会忽略 $GOPATH 和 vendor 文件夹，只根据go.mod下载依赖。
  ```

- auto模式：

  ```
  auto 或未设置，则go命令根据当前目录启用或禁用模块支持。仅当当前目录位于$GOPATH/src之外并且其本身包含go.mod文件或位于包含go.mod文件的目录下时，才启用模块支持。
  ```

  

如果需要查看go module的详细文档(非常长)，可以在控制台输入

```go
go help modules

go mod命令

download    download modules to local cache (下载依赖的module到本地cache))
edit        edit go.mod from tools or scripts (编辑go.mod文件)
graph       print module requirement graph (打印模块依赖图))
init        initialize new module in current directory (再当前文件夹下初始化一个新的module, 创建go.mod文件))

tidy        add missing and remove unused modules (增加丢失的module，去掉未用的module)
vendor      make vendored copy of dependencies (将依赖复制到vendor下)
verify      verify dependencies have expected content (校验依赖)
why         explain why packages or modules are needed (解释为什么需要依赖)

初始化mod

go mod init [module]可以创建一个go.mod，只有一行信息module。
```

go命令通过查找当前目录中的go.mod或者当前目录的父目录，或者祖父目录，依次递归查找。
 go.mod文件可以通过require，replace和exclude语句使用的精确软件包集。

```
require语句指定的依赖项模块
replace语句可以替换依赖项模块
exclude语句可以忽略依赖项模块
```



- go mod download

  ```
  可以下载所需要的依赖，但是依赖并不是下载到GOPATH中，而是GOPATH/pkg/mod中，多个项目可以共享缓存的module。
  ```

  在国内访问 golang.org/x 的各个包都需要翻墙，你可以在go.mod中使用replace替换成github上对应的库。

  ```
  require (
  	github.com/hashicorp/go-uuid v1.0.1 // indirect
  	github.com/micro/protoc-gen-micro v0.8.0
  	github.com/stretchr/testify v1.3.0
  	google.golang.org/appengine v1.4.0
  )
  ```

- go clean -modcache 清除缓存

  ```
  go mod 新东西偶尔还会出现问题 ,这个命令可以尝试修复,不过执行之前可以考虑备份一下pkg/mod中的包.以防不测.
  ```

  

