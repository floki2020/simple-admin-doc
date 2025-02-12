---
order: 1
title: 'API 微服务'
---


# 3 分钟开发 API 服务

::: warning
首先确认你安装了以下软件:

- simple-admin-tool (goctls) v0.2.2 +

必须了解 go zero 的 API 命令  [API命令](https://go-zero.dev/cn/docs/goctl/api) [api文件编写](https://go-zero.dev/cn/docs/advance/api-coding) \
\
参考 [Example](https://github.com/suyuan32/simple-admin-example-api) 项目生成一遍，确认生成文件与Example项目一致，Example项目有完整的命令
:::


## API服务的职责
在 simple admin 中， API 服务充当网关的角色，主要提供以下功能：

- 用户鉴权， 如 JWT
- 数据处理， 如数据过滤筛选，国际化翻译
- 限流和熔断

一个API可以接入多个 RPC， 提供统一的请求入口

## 创建 API 项目

创建 example

```shell
goctls api new example --i18n=true --casbin=true --go_zero_version=v1.4.4 --tool_version=v0.2.2 --trans_err=true --module_name=github.com/suyuan32/simple-admin-example-api --port=8081 --gitlab=true
```

### `api new` 参数介绍

| 参数              | 必须  | 默认值   | 介绍                     | 使用方法                                                                                               |
|-----------------|-----|-------|------------------------|----------------------------------------------------------------------------------------------------|
| i18n            | 否   | false | 是否启用 i18n              | true 为启用                                                                                           |
| casbin          | 否   | false | 是否启用 casbin            | true 为启用                                                                                           |
| module_name     | 是   |       | go.mod 中的module名称      | 如果项目需要被在外部import，需要像上面例子设置为github或者其他地方的仓库网址， 为空则只在本地使用                                            |
| go_zero_version | 是   |       | go zero版本              | 需要到[go-zero](https://github.com/zeromicro/go-zero/releases)查看最新release                             |
| tool_version    | 是   |       | simple admin tools 版本号 | 需要到[tool](https://github.com/suyuan32/simple-admin-tools/releases)查看simple admin  tools 最新 release |
| trans_err       | 否   | false | 国际化翻译错误信息              | true 为启用                                                                                           |
| gitlab          | 否   | false | 是否生成 gitlab-ci.yml     | true 为生成                                                                                           |
| port            | 否   | 9100  | 端口号                    | 服务暴露的端口号                                                                                           |

**详细参数请在命令行查看 `goctls api new --help`**

> 你可以看到以下结构

![Example](/assets/example-struct.png)


### 文件结构

```text
├── desc                              api声明文件存放目录
├── etc                               配置文件目录
└── internal
    ├── config
    ├── handler                       handler目录
    │   ├── base
    │   ├── student
    │   └── teacher
    ├── i18n                          国际化i18n文件目录
    │   └── locale
    ├── logic                         业务代码目录
    │   ├── base
    │   ├── student
    │   └── teacher
    ├── middleware                    中间件目录
    ├── svc                           全局参数目录
    └── types                         类型声明目录


```

> 然后编辑 etc/example.yaml

```yaml
Name: example.api
Host: 0.0.0.0
Port: 8081
Timeout: 30000

Auth:
  AccessSecret: # the same as core
  AccessExpire: 259200

Log:
  ServiceName: exampleApiLogger
  Mode: file
  Path: /home/ryan/data/logs/example/api
  Level: info
  Compress: false
  KeepDays: 7
  StackCoolDownMillis: 100

Prometheus:
  Host: 0.0.0.0
  Port: 4000
  Path: /metrics


RedisConf:
  Host: 127.0.0.1:6379
  Type: node

DatabaseConf:
  Type: mysql
  Host: 127.0.0.1
  Port: 3306
  DBName: simple_admin
  Username: root # set your username
  Password: "123456" # set your password
  MaxOpenConn: 100
  SSLMode: disable
  CacheTime: 5

CasbinConf:
  ModelText: |
    [request_definition]
    r = sub, obj, act
    [policy_definition]
    p = sub, obj, act
    [role_definition]
    g = _, _
    [policy_effect]
    e = some(where (p.eft == allow))
    [matchers]
    m = r.sub == p.sub && keyMatch2(r.obj,p.obj) && r.act == p.act

ExampleRpc:
  Endpoints:
    - 127.0.0.1:8080
```

> 运行代码

```shell
go run example.go -f etc/example.yaml
```

> 如果看到

```shell
Starting server at 127.0.0.1:8081...
```

说明运行成功.

## 代码生成（基于Proto）

::: warning
proto 必须为 ```goctls rpc ent``` 生成的 proto
:::

```shell
goctls api proto --proto=/home/ryan/GolandProjects/simple-admin-example-rpc/example.proto --style=go_zero --api_service_name=example --rpc_service_name=Example --o=./ --model=Student --rpc_name=Example --grpc_package=github.com/suyuan32/simple-admin-example-rpc/example
```

### `api proto` 参数介绍

| 参数               | 必须  | 默认值     | 介绍                | 使用方法                                                            |
|------------------|-----|---------|-------------------|-----------------------------------------------------------------|
| proto            | 是   |         | proto文件地址         | 输入proto文件的绝对路径, 注意要为合并后的proto即根目录下的proto ，不是desc 文件夹中的          |
| style            | 否   | go_zero | 文件名格式             | go_zero为蛇形格式                                                    |
| api_service_name | 是   |         | 服务名称              | api 服务的 service 名称, 在api声明文件中                                   |
| rpc_service_name | 是   |         | 服务名称              | rpc 服务的名称, 与proto文件中的service名称一致                                |
| o                | 是   |         | 输出位置              | 文件输出位置，可以为相对路径，指向main文件目录                                       |
| model            | 是   |         | 模型名称              | schema中内部struct名称，如example中的Student                             |
| rpc_name         | 是   |         | RPC名称             | 输入Example则生成文件会生成l.svcCtx.ExampleRpc                            |
| grpc_package     | 是   |         | RPC *_grpc.go 包路径 | 在example中是 github.com/suyuan32/simple-admin-example-rpc/example |
| multiple         | 否   | false   | 多服务               | 若 proto 文件中有多个service, 需要设置为 true                               |

**详细参数请在命令行查看 `goctls api proto --help`**

::: info
multiple 例子, multiple 用于根据不同服务生成多个 rpcclient

```shell
goctls api proto --proto=/home/ryan/GolandProjects/simple-admin-example-rpc/example.proto --style=go_zero --api_service_name=example --rpc_service_name=school --o=./ --model=Teacher --rpc_name=School --grpc_package=github.com/suyuan32/simple-admin-example-rpc/example --multiple=true
```

[代码](https://github.com/suyuan32/simple-admin-example-api/tree/multiple)
:::

> 生成效果

![pic](/assets/api_gen_struct.png)

> 详情查看 simple admin example api 地址 <https://github.com/suyuan32/simple-admin-example-api>

::: warning
还需要手动添加下 service_context, config, etc, ExampleRpc
:::