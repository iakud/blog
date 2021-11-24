---
categories:
- program
date: "2019-12-04T10:59:04Z"
title: Go Modules
---

#### 一、Go Modules介绍

go modules是golang1.11新加的特性。

*官方定义：modules是相关Go包的集合。modules是源代码交换和版本控制的单元。go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于GOPATH的方法来指定在给定构建中使用哪些源文件。*
<!--more-->
#### 二、go mod命令

go mod有以下命令：

| 命令     | 说明                                       | 备注                           |
| -------- | ------------------------------------------ | ------------------------------ |
| download | download modules to local cache            | 下载依赖包                     |
| edit     | edit go.mod from tools or scripts          | 编辑go.mod                     |
| graph    | print module requirement graph             | 打印模块依赖图                 |
| init     | initialize new module in current directory | 在当前目录初始化mod            |
| tidy     | add missing and remove unused modules      | 拉取缺少的模块，移除不用的模块 |
| vendor   | make vendored copy of dependencies         | 将依赖复制到vendor下           |
| verify   | verify dependencies have expected content  | 验证依赖是否正确               |
| why      | explain why packages or modules are needed | 解释为什么需要依赖             |

#### 三、go.mod如何在项目中使用

go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如`go get`、`go build`、`go mod`等修改和维护go.mod文件。

go.mod提供了四个命令

- `module`语句指定包的名字（路径）
- `require`语句指定的依赖项模块
- `replace`语句可以替换依赖项模块
- `exclude`语句可以忽略依赖项模块

*官方说明：除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希。*

go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。go.mod和go.sum都应检入版本控制。

go.sum不需要手工维护，所以可以不用太关注。

##### 1、go mod init modname

运行完之后，会在当前目录下生成一个go.mod文件，这是一个关键文件，之后的包管理都是通过这个文件管理。

##### 2、go build main.go

main.go文件中增加了`import “go.etcd.io/etcd/clientv3”`的语句，执行build后依赖包记录在go.mod中

```
module nn4

go 1.13

require (
	github.com/coreos/etcd v3.3.18+incompatible // indirect
	github.com/coreos/go-systemd v0.0.0-00010101000000-000000000000 // indirect
	github.com/coreos/pkg v0.0.0-20180928190104-399ea9e2e55f // indirect
	github.com/gogo/protobuf v1.3.1 // indirect
	github.com/google/uuid v1.1.1 // indirect
	go.etcd.io/etcd v3.3.18+incompatible
	go.uber.org/zap v1.13.0 // indirect
	google.golang.org/grpc v1.25.1 // indirect
)
```

##### 3、go mod download

依赖包会自动下载到$GOPATH/pkg/mod，多个项目可以共享缓存的mod。

##### 4、go mod vendor

将依赖包放到vendor目录下，只把项目中import的部分复制到了vendor目录。

##### 5、go build main.go

go命令寻找依赖包会自动通过vendor目录来查找