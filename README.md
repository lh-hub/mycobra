## Cobra 学习文档

https://github.com/spf13/cobra

Cobra 是一个用来创建命令行程序的库，应用在许多 Go 项目上。

### 概念

Cobra 命令由 commands, arguments 和 flags 三部分构成，不太清楚有什么用，继续往下看。

### 安装和导入

```bash
go get -u github.com/spf13/cobra@latest
```

```go
import "github.com/spf13/cobra"
```

### 练习

创建 go 项目

```bash
cd  $HOME/GoProjects
mkdir mycobra
cd mycobra
go mod init mycobra
```

安装 Cobra

```bash
go get -u github.com/spf13/cobra@latest
```

初始化 Cobra 命令行应用

```bash
cd mycobra
cobra-cli init
go run main.go
```

项目结构如下

```
  ▾ mycobra/
    ▾ cmd/
        root.go
      main.go
```

添加 commands

```bash
cobra-cli add serve
cobra-cli add config
cobra-cli add create -p 'configCmd'
```

然后城市带参数运行，会看到相应的输出

```bash
go run main.go serve
go run main.go config
go run main.go config create
```

### 源代码分析

main.go 很简单，只是导入了一个包，执行了 Execute 方法

```go
package main

import "mycobra/cmd"

func main() {
	cmd.Execute()
}
```

cmd/root.go 中首先定义了 main 中引入的 Execute 方法，执行了 rootCmd.Execute()；然后创建了 cobra.Command 对象，最后有个 init() 在引入文件时做初始化用。

总结一下，一共三板斧：

1. 引入文件时初始化内容，调用 init()
2. 创建 cobra.Command 对象，在这个对象中有真正的 rootCmd.Execute
3. 定义方法包 Execute()，用于给外部调用

```go
package cmd

import (
	"os"

	"github.com/spf13/cobra"
)

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
	Use:   "mycobra",
	Short: "A brief description of your application",
	Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
}

func Execute() {
	err := rootCmd.Execute()
	if err != nil {
		os.Exit(1)
	}
}

func init() {
	rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

cmd/serve.go 中定义了命令 serve 用法。首先实现了 cobra.Command 对象，并实现了 Run 方法，然后通过 init() 方法将命令添加到 rootCmd 中，此时就可以调用 serve 命令了。

子命令因为不同被外部调用，三板斧就只剩两板了：

1. 实现 cobra.Command 对象
2. 在 init() 中将实现的对象添加到 rootCmd 中

```go
package cmd

import (
   "fmt"

   "github.com/spf13/cobra"
)

// serveCmd represents the serve command
var serveCmd = &cobra.Command{
   Use:   "serve",
   Short: "A brief description of your command",
   Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
   Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("serve called")
   },
}

func init() {
   rootCmd.AddCommand(serveCmd)
}
```
