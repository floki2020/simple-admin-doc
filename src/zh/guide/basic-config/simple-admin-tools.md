---
order: 1
title: 'Simple Admin Tools 开发工具'
---

# Simple admin tools

Simple admin tools 是一个基于go-zero的fork项目。
它提供了许多额外的功能，例如:

- go-swagger : 基于go-swagger而不是官方的@doc注解
- 多国语言支持
- 优化错误信息处理,支持多语言错误
- 简单易用的校验器
- 支持代码生成，生成API,RPC 和 web 端的CRUD代码
- 支持多种额外插件如GORM, RocketMQ
- 对Simple Admin 的针对性优化

由于本工具是fork项目，因此使用goctls会有些麻烦. 使用fork主要是为了同步官方最新代码。

我们不能直接使用 go get and go install 命令安装 goctl 因为他会安装官方的文件，我们需要自行编译。

下面是构建goctls的过程.

> 构建 goctls

```shell
git clone https://github.com/suyuan32/simple-admin-tools.git

cd tools/goctl

go mod tidy

# 输出goctls文件避免和官方的goctl冲突
go build -o goctls goctl.go

cp ./goctls $GOPATH/bin/goctls
```

::: info 简单方式
直接运行在 goctl 文件夹运行 `sh build.sh`
:::

## 自动下载依赖

```shell
goctls env check -i -f --verbose
```

这个命令会自动安装 protoc, swagger 等依赖.

### API 命令

> 命令和goctl一样，但是需要改成 goctls.

```shell
$ goctls api -h
NAME:
   goctl api - generate api related files

USAGE:
   goctl api command [command options] [arguments...]

COMMANDS:
   new       fast create api service
   format    format api files
   validate  validate api file
   doc       generate doc files
   go        generate go files for provided api in yaml file
   java      generate java files for provided api in api file
   ts        generate ts files for provided api in api file
   dart      generate dart files for provided api in api file
   kt        generate kotlin code for provided api file
   plugin    custom file generator

OPTIONS:
   -o value        output a sample api file
   --home value    the goctl home path of the template, --home and --remote cannot be set at the same time, if they are, --remote has higher priority
   --remote value  the remote git repo of the template, --home and --remote cannot be set at the same time, if they are, --remote has higher priority
                   The git repo directory must be consistent with the https://github.com/zeromicro/go-zero-template directory structure
   --branch value  the branch of the remote repo, it does work with --remote
   --help, -h      show help
```

> 例子:

```shell
goctls api go -api core.api -dir .
```

根据 core.api 里的定义生成 go 文件， -dir 设置输出位置.

### Rpc 命令

```shell
$ goctl rpc protoc -h
NAME:
   goctl rpc protoc - generate grpc code

USAGE:
   example: goctl rpc protoc xx.proto --go_out=./pb --go-grpc_out=./pb --zrpc_out=.

DESCRIPTION:
   for details, see https://go-zero.dev/cn/goctl-rpc.html

OPTIONS:
   --zrpc_out value  the zrpc output directory
   --style value     the file naming format, see [https://github.com/zeromicro/go-zero/tree/master/tools/goctl/config/readme.md]
   --home value      the goctl home path of the template
   --remote value    the remote git repo of the template, --home and --remote cannot be set at the same time, if they are, --remote has higher priority
                     The git repo directory must be consistent with the https://github.com/zeromicro/go-zero-template directory structure
   --branch value    the branch of the remote repo, it does work with --remote
   --verbose, -v     enable log output
```

> 例子: 生成proto的模板

```shell
goctls rpc template -o=user.proto
```

> 生成的文件

```shell
syntax = "proto3";

package user;
option go_package=". /user";

message Request {
  string ping = 1;
}

message Response {
  string pong = 1;
}

service User {
  rpc Ping(Request) returns(Response);
} 

```

> 生成go文件

```shell
goctl rpc protoc user.proto --go_out=. --go-grpc_out=. --zrpc_out=.
```

::: info
可以访问官网查看更多 [RPC command](https://go-zero.dev/docs/goctl/zrpc)
:::

> go.mod 配置

导入fork项目需要使用 replace 命令。

```mod
module github.com/suyuan32/simple-admin-core

go 1.19

require (
 github.com/casbin/casbin/v2 v2.52.1
 github.com/casbin/gorm-adapter/v3 v3.7.4
 github.com/go-playground/locales v0.14.0
 github.com/go-playground/validator/v10 v10.11.1
 github.com/golang-jwt/jwt/v4 v4.4.2
 github.com/google/uuid v1.3.0
 github.com/mojocn/base64Captcha v1.3.5
 github.com/pkg/errors v0.9.1
 github.com/stretchr/testify v1.8.0
 github.com/suyuan32/simple-admin-tools/plugins/registry/consul v0.0.0-20220923060146-bde681863b8d
 github.com/zeromicro/go-zero v1.4.1
 golang.org/x/crypto v0.0.0-20220722155217-630584e8d5aa
 google.golang.org/grpc v1.49.0
 google.golang.org/protobuf v1.28.1
 gorm.io/gorm v1.23.8
)

replace github.com/zeromicro/go-zero v1.4.3 => github.com/suyuan32/simple-admin-tools v0.1.5
```

> 简单的方法是使用 goctls migrate命令.

```shell
goctls migrate --zero-version v1.4.3 --tool-version v0.1.5
```

> 这个命令可以快速添加 replace 语句， 但是不要运行多次， 会导致添加重复和 replace, 后续需要升级依赖的话直接修改 simple-admin-tools 的版本即可，然后
 **go mod tidy**.

> 新版本API和RPC均支持新建时直接设置tool版本进行replace
