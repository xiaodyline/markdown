# Go 模块化教程

## 1. 什么是 Go 模块化

Go 模块化主要指使用 `go mod` 管理项目依赖、组织项目结构、拆分代码职责，使代码具备更好的复用性、可维护性和可扩展性。

在较早的 Go 开发方式中，项目通常依赖 `GOPATH` 进行管理。现在主流方式已经变为基于模块的开发方式，即每个项目以一个 `go.mod` 文件作为模块定义入口。

一个 Go 模块通常可以理解为一个完整项目，模块内部再按功能拆分为多个包。

------

## 2. 模块、包、文件三者关系

### 2.1 文件

Go 源码文件就是 `.go` 文件，例如：

```go
main.go
user.go
config.go
```

### 2.2 包

多个同目录、同 `package` 声明的 `.go` 文件共同组成一个包。

例如：

```go
package service
```

表示这些文件属于 `service` 包。

### 2.3 模块

一个模块由 `go.mod` 定义，模块下可以包含多个包。

例如：

```go
module github.com/example/demo
```

表示当前项目模块路径是 `github.com/example/demo`。

------

## 3. 为什么要模块化

模块化带来的核心价值主要有以下几方面：

1. 便于代码拆分
   不同功能写在不同包中，职责更清晰。
2. 便于复用
   公共逻辑可以抽成独立包，多处调用。
3. 便于维护
   项目规模变大后，目录结构更稳定，不容易混乱。
4. 便于依赖管理
   使用 `go.mod` 和 `go.sum` 可以明确依赖版本。
5. 便于团队协作
   不同成员可在不同模块或不同包中协同开发。

------

## 4. 初始化一个 Go 模块

假设要创建一个项目 `myapp`。

### 4.1 创建项目目录

```bash
mkdir myapp
cd myapp
```

### 4.2 初始化模块

```bash
go mod init myapp
```

如果将来代码要托管到 GitHub，通常写成：

```bash
go mod init github.com/xiaodyline/monitor
```

执行后会生成：

```go
module myapp

go 1.22
```

这就是最基础的 `go.mod` 文件。

------

## 5. 最简单的模块示例

项目结构如下：

```text
myapp/
├─ go.mod
└─ main.go
```

`main.go` 内容如下：

```go
package main

import "fmt"

func main() {
	fmt.Println("hello go module")
}
```

运行：

```bash
go run .
```

这里的 `go run .` 表示运行当前目录这个包。

------

## 6. 包的基本拆分

当项目代码变多时，不应全部写在 `main.go` 中，而应该按功能拆分包。

### 6.1 示例目录结构

```text
myapp/
├─ go.mod
├─ main.go
└─ utils/
   └─ math.go
```

### 6.2 编写工具包

```
utils/math.go
package utils

func Add(a, b int) int {
	return a + b
}
```

### 6.3 在主程序中导入

```
main.go
package main

import (
	"fmt"
	"myapp/utils"
)

func main() {
	result := utils.Add(3, 5)
	fmt.Println(result)
}
```

运行后输出：

```text
8
```

这里需要注意两点：

1. 导入路径是 `模块名/目录名`
2. 函数名首字母大写表示可导出，外部包才能访问

------

## 7. Go 中的导出规则

Go 没有 `public`、`private` 关键字，而是通过首字母大小写控制可见性。

### 7.1 可导出

```go
func Add() {}
type User struct {}
var Name string
```

首字母大写，其他包可访问。

### 7.2 不可导出

```go
func add() {}
type user struct {}
var name string
```

首字母小写，仅当前包内部可访问。

------

## 8. 一个稍规范的项目结构

对于普通业务项目，可以采用如下结构：

```text
myapp/
├─ go.mod
├─ go.sum
├─ main.go
├─ cmd/
│  └─ server/
│     └─ main.go
├─ internal/
│  ├─ service/
│  │  └─ user_service.go
│  └─ repository/
│     └─ user_repo.go
├─ pkg/
│  └─ utils/
│     └─ stringx.go
└─ config/
   └─ config.go
```

### 8.1 各目录含义

#### `cmd/`

放程序入口。
如果一个项目有多个可执行程序，可以在这里放多个入口。

例如：

```text
cmd/server/main.go
cmd/worker/main.go
```

#### `internal/`

放项目内部使用的代码。
这个目录下的包只能被当前模块内部导入，模块外不能导入。

适合放：

- 业务逻辑
- 数据访问层
- 项目私有实现

#### `pkg/`

放可复用的公共包。
通常用于未来可能被其他项目复用的代码。

#### `config/`

放配置读取逻辑、配置结构体等。

------

## 9. `main` 包和普通包的区别

### 9.1 `package main`

表示这是一个可执行程序入口包，必须包含 `func main()`。

例如：

```go
package main

func main() {

}
```

### 9.2 普通包

普通包只是提供功能，不直接运行。

例如：

```go
package service

func CreateUser() {

}
```

------

## 10. 多文件协作

同一个目录下可以有多个 `.go` 文件，它们只要属于同一个包，就会一起参与编译。

例如：

```text
service/
├─ user.go
└─ order.go
user.go
package service

func CreateUser() string {
	return "create user"
}
order.go
package service

func CreateOrder() string {
	return "create order"
}
```

它们都属于 `service` 包，可以互相调用同包内未导出内容。

------

## 11. 模块内的分层示例

下面给出一个简单的业务分层示例。

### 11.1 项目结构

```text
myapp/
├─ go.mod
├─ main.go
├─ internal/
│  ├─ repository/
│  │  └─ user_repository.go
│  └─ service/
│     └─ user_service.go
```

### 11.2 repository 层

```
internal/repository/user_repository.go
package repository

func FindUserNameByID(id int) string {
	if id == 1 {
		return "Alice"
	}
	return "Unknown"
}
```

### 11.3 service 层

```
internal/service/user_service.go
package service

import "myapp/internal/repository"

func GetUserName(id int) string {
	return repository.FindUserNameByID(id)
}
```

### 11.4 main 入口

```
main.go
package main

import (
	"fmt"
	"myapp/internal/service"
)

func main() {
	name := service.GetUserName(1)
	fmt.Println(name)
}
```

------

## 12. 使用第三方依赖

Go 模块化的一个核心能力就是管理第三方包。

### 12.1 导入依赖

例如引入 Gin：

```go
import "github.com/gin-gonic/gin"
```

当执行：

```bash
go mod tidy
```

或者：

```bash
go run .
```

Go 会自动下载缺失依赖，并更新 `go.mod` 和 `go.sum`。

### 12.2 `go.mod` 示例

```go
module myapp

go 1.22

require github.com/gin-gonic/gin v1.10.0
```

### 12.3 `go.sum` 作用

`go.sum` 用于记录依赖包校验信息，保证下载内容一致可靠。
一般应提交到版本控制，不建议随意删除。

------

## 13. 常用 `go mod` 命令

### 13.1 初始化模块

```bash
go mod init 模块名
```

### 13.2 整理依赖

```bash
go mod tidy
```

作用是：

- 自动补充缺少的依赖
- 删除未使用的依赖
- 更新 `go.sum`

### 13.3 下载依赖

```bash
go mod download
```

### 13.4 查看依赖

```bash
go list -m all
```

### 13.5 查看为什么依赖某模块

```bash
go mod why 模块路径
```

例如：

```bash
go mod why github.com/gin-gonic/gin
```

------

## 14. 本地包导入规则

假设模块名是：

```go
module github.com/yourname/myapp
```

目录结构如下：

```text
myapp/
├─ go.mod
├─ main.go
└─ utils/
   └─ math.go
```

则导入时应写为：

```go
import "github.com/yourname/myapp/utils"
```

不是写相对路径，不是写 `./utils`，也不是只写 `utils`。

Go 的包导入遵循“模块路径 + 子目录”的规则。

------

## 15. `internal` 的意义

`internal` 是 Go 官方支持的一种目录约束机制。

例如：

```text
myapp/
├─ internal/
│  └─ service/
│     └─ user_service.go
```

外部其他模块不能导入：

```go
import "myapp/internal/service"
```

只有 `myapp` 模块内部代码可以导入。

这适合限制项目内部实现细节，防止外部误用。

------

## 16. `pkg` 是否必须使用

不是必须。

很多初学者容易把 `pkg` 机械化使用，但实际应根据项目规模决定。

通常可以这样理解：

- 明确只给当前项目内部用的，放 `internal`
- 明确希望对外复用的，放 `pkg`
- 小项目不复杂时，直接按业务拆目录也可以

不要为了“看起来规范”而过度设计。

------

## 17. 一个完整的小型模块化项目示例

### 17.1 目录结构

```text
myapp/
├─ go.mod
├─ cmd/
│  └─ app/
│     └─ main.go
├─ internal/
│  ├─ service/
│  │  └─ calc_service.go
│  └─ handler/
│     └─ calc_handler.go
└─ pkg/
   └─ mathx/
      └─ add.go
```

### 17.2 `pkg/mathx/add.go`

```go
package mathx

func Add(a, b int) int {
	return a + b
}
```

### 17.3 `internal/service/calc_service.go`

```go
package service

import "myapp/pkg/mathx"

func Sum(a, b int) int {
	return mathx.Add(a, b)
}
```

### 17.4 `internal/handler/calc_handler.go`

```go
package handler

import (
	"fmt"
	"myapp/internal/service"
)

func Run() {
	result := service.Sum(10, 20)
	fmt.Println("sum =", result)
}
```

### 17.5 `cmd/app/main.go`

```go
package main

import "myapp/internal/handler"

func main() {
	handler.Run()
}
```

### 17.6 运行方式

进入项目根目录后执行：

```bash
go run ./cmd/app
```

这样会运行 `cmd/app` 这个入口包。

------

## 18. 多入口程序

一个项目有时不止一个可执行程序，例如：

- Web 服务
- 后台任务
- 命令行工具

这时可以使用多个入口：

```text
myapp/
├─ cmd/
│  ├─ server/
│  │  └─ main.go
│  └─ worker/
│     └─ main.go
```

运行：

```bash
go run ./cmd/server
go run ./cmd/worker
```

这种结构在中大型项目里非常常见。

------

## 19. 模块版本管理基础

当你的项目依赖第三方模块时，`go.mod` 会记录版本：

```go
require github.com/gin-gonic/gin v1.10.0
```

Go 会根据版本拉取依赖。
在多人协作时，统一版本可以减少环境不一致问题。

如果升级依赖，可以执行：

```bash
go get github.com/gin-gonic/gin@latest
```

或者指定版本：

```bash
go get github.com/gin-gonic/gin@v1.10.0
```

之后再执行：

```bash
go mod tidy
```

------

## 20. 本地模块替换 `replace`

开发时，有时需要让一个模块依赖本地另一个模块。

例如当前项目依赖：

```go
github.com/yourname/commonlib
```

但本地还在开发这个模块，此时可以在 `go.mod` 中写：

```go
replace github.com/yourname/commonlib => ../commonlib
```

这样 Go 会优先使用本地目录。

这在本地联调、拆分多个仓库时非常有用。

------

## 21. `go.work` 的作用

当你同时开发多个模块时，可以使用 `go.work` 统一管理。

例如：

```text
workspace/
├─ app/
│  └─ go.mod
└─ commonlib/
   └─ go.mod
```

在 `workspace` 下执行：

```bash
go work init ./app ./commonlib
```

会生成 `go.work`：

```go
go 1.22

use (
	./app
	./commonlib
)
```

这样多个模块可以在一个工作区内协同开发，不必频繁写 `replace`。

------

## 22. 模块化开发中的常见问题

### 22.1 导入路径写错

错误示例：

```go
import "./utils"
```

正确方式应使用模块路径：

```go
import "myapp/utils"
```

### 22.2 函数无法访问

如果函数名首字母小写，其他包不能调用。

错误示例：

```go
func add(a, b int) int
```

正确示例：

```go
func Add(a, b int) int
```

### 22.3 一个目录里混用多个包名

同目录下的 `.go` 文件应保持同一个包名，否则会报错。

### 22.4 把所有代码都写在 `main.go`

小项目可以这样做，但项目稍大后会迅速失控，应及时拆包。

### 22.5 过度分层

模块化不是目录越多越好。
应以职责清晰、便于维护为原则，而不是为了形式拆分。

------

## 23. 推荐的学习顺序

对于初学者，建议按下面顺序理解 Go 模块化：

第一步，理解模块、包、文件之间的关系。
第二步，学会使用 `go mod init`、`go mod tidy`。
第三步，学会拆分本地包并正确导入。
第四步，理解导出规则和 `main` 包。
第五步，尝试做一个带 `internal`、`cmd` 的小项目。
第六步，再学习 `replace`、`go.work`、多模块协作。

------

## 24. 一个适合练手的目录模板

```text
project-demo/
├─ go.mod
├─ cmd/
│  └─ app/
│     └─ main.go
├─ internal/
│  ├─ service/
│  │  └─ user_service.go
│  └─ repository/
│     └─ user_repository.go
├─ pkg/
│  └─ utils/
│     └─ stringx.go
└─ configs/
   └─ config.yaml
```

这个模板适合做以下类型项目：

- 命令行工具
- 小型后端服务
- 课程作业
- 练手项目

------

## 25. 总结

Go 的模块化可以概括为三层理解：

文件层面，多个 `.go` 文件组成一个包。
包层面，不同目录按功能拆分不同包。
模块层面，整个项目由 `go.mod` 管理依赖和路径。

在实际开发中，建议遵循以下原则：

1. 小项目先保证能跑通，再逐步模块化
2. 包的拆分以职责清晰为核心
3. 内部逻辑优先放 `internal`
4. 可复用能力再考虑放 `pkg`
5. 使用 `go mod tidy` 维护依赖整洁
6. 多入口程序放在 `cmd`
7. 多模块协作时使用 `replace` 或 `go.work`

掌握这些内容后，已经可以完成大多数 Go 项目的基础模块化设计。