---
title: "[Golang Learning]  Viper - A Complete Configuration Solution"
short: "Viper is a complete configuration solution for Go applications including 12-Factor apps. It is designed to work within an application, and can handle all types of configuration needs and formats. Let's know aboout it. "
date: 13.Dec.2017
tags:
    - c
    - golang
    - server
    - notes
---

# What is Viper
   
 [Viper](https://github.com/sp13/viper) 是sp13开发的一组用于配置Go程序配置包，它拥有以下特性：
- 设置默认值
- 从JSON，TOML,  YAML，HCl,和java属性配置文件等读取配置文件
- 现场读取（实时？）
- 从环境变量ENV读取值
- 远程读取配置文件
- 读缓冲区
- key不区分大小写

# Do like this

```go
//main.go
package main

import (
    "fmt"
    "github.com/spf13/viper"
    "os"
    "strings"
)

const cmdRoot = "core"

func main() {
    viper.SetEnvPrefix(cmdRoot)
    /*当AutomaticEnv被调用时，Viper将在任何时候检查viper.Get请求时检查环境变量。它将应用以下规则：
    它将检查一个环境变量，其名称与大写字母匹配， 并且如果设置了EnvPrefix前缀。 */
    viper.AutomaticEnv()
    replacer := strings.NewReplacer(".", "_")
    viper.SetEnvKeyReplacer(replacer)
    viper.SetConfigName(cmdRoot)
    viper.AddConfigPath("./")

    err := viper.ReadInConfig()
    if err != nil {
        fmt.Println(fmt.Errorf("Fatal error when reading %s config file:%s", cmdRoot, err))
        os.Exit(1)
    }

    environment := viper.GetBool("security.enabled")
    fmt.Println("security.enabled:", environment)

    fullstate := viper.GetString("statetransfer.timeout.fullstate")
    fmt.Println("statetransfer.timeout.fullstate:", fullstate)

    abcdValue := viper.GetString("peer.abcd")
    fmt.Println("peer.abcd:", abcdValue)
}
```
core.toml 配置文件
```toml
title = "config test"
[statetransfer]
    recoverdamage=true 
	blocksperrequest=20
	maxdeltas=200
	[statetransfer.timeout]
		singleblock="2s" 
		singlestatedelta = "2s"
		fullstate="60s"

[peer]
	abcd="3322d"
```
运行结果：
```bash
$ ./vip
security.enabled: false
statetransfer.timeout.fullstate: 60s
peer.abcd: 3322d
```
# The Best Friends Cobra

Cobra提供了subcommand(子命令)功能，同时提供了与posix兼容的命令行标志解析能力，包括长短标志、内嵌命令、自定义help和usage等。

创建cmd包
cmd/root.go
```go
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

var name *string
var age *int

var mainCmd = &cobra.Command{
	Use:   "main [options]",
	Short: "short usage",
	Long:  ``,
	Run: func(cmd *cobra.Command, args []string) {
        fmt.Printf("My name is %s, My age is %d\n",
             viper.GetString("name"), viper.GetInt("age"))
	},
}

func Execute() {
	if err := mainCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
func init() {
	viper.SetEnvPrefix("CONFIG")
	viper.AutomaticEnv()

	flags := mainCmd.Flags()

	flags.StringP("name", "n", "", "your name")
	flags.Int("age", 0, "your age")

    // viper.BindFlags(flags) 代替下面两行

	viper.BindPFlag("name", flags.Lookup("name"))
	viper.BindPFlag("age", flags.Lookup("age"))

	viper.SetConfigName("config")
	viper.AddConfigPath("./")

	err := viper.ReadInConfig()
	if err != nil {
		fmt.Println("No config file found.Using build-in defaults.")
	}
}

func init2() {
}
```
main.go
```go
package main

import "eleztian/viper_test/viper_cobra/cmd"

func main() {
	cmd.Execute()
}
```
config.toml
```toml
Title = " config.toml"
name="zt"
age=20
```
运行结果如下：
```bash
$ ./main
My name is zt, My age is 20
$ ./main -n joke --age 30
My name is joke, My age is 30
$
```
有上面结果可以得知，viper+cobra的组合可以轻松实现下面的情况：
1. 程序内内置配置项的初始默认值。
2. 配置文件中的配置项值可以覆盖(override)程序内配置项的默认值。
3. 命令行选项和参数值具有最高优先级，可以override前两层的配置项值。

以上仅仅是自己的入门应用。