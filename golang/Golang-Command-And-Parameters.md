---
title: [Golang Learning]  Command And Parameters
short: "The ways of getting the command line and parameters of golang."
date: 16.Dec.2017
tags:
    - c
    - golang
    - server
    - notes
---

# 标准库
1. ## os 
    
    在os包里面定义了一个Args， 类似于c语言里面的main参数Args.
    
    ```go
    var Args []string
    ```
    我们可以这样简单的获取命令行参数, 获取的命令行参数里包含了运行程序的这条指令，也就os.Args[0] 为这条指令本身。

    ```go
    import (
        "fmt"
        "os"
    )
    func main() {
        fmt.Println(os.Args)
    }
    ```
    这种方式的操作需要自己处理参数信息。
2. ## flag

    flag 包提供比os.Args更加方便的命令行处理方法。
    它将命令行参数分为非标志类参数(nonflagarguments)和标志类参数(flag arguments), 标志类比如-name=zt, 非标志类比如arg1 agr2。

    flag 处理标志类参数和使用os相似。flag.Args() []string。但是非标志类参数必须位于标志类参数之后。且这里的非标志参数不包含命令本身。
    标志类参数必须在flg.Parse()之前定义，否则出错。在flag.parse()之前的值都为默认值。

    ```go
    package main
 
    import (
    "flag"
    "fmt"
    )
    var (
        ok := flag.Bool("ok", false, "is ok")
        id := flag.Int("id", 0, "id")
        port := flag.String("port", ":8080", "http listen port")
        name string
    )

    func main() {
     
        flag.StringVar(&name, "name", "123", "name")

        flag.Parse()

        fmt.Println("ok:", *ok)
        fmt.Println("id:", *id)
        fmt.Println("port:", *port)
        fmt.Println("name:", name)
        fmt.Println("args:", flag.Args())
    }
    ```
    运行结果如下：
    ```bash
    $ go run flag.go -id=2 -name="golang" hello
    ok: false
    id: 2
    port: :8080
    name: golang
    hello
    ```
    由上例可以看出flag支持如下格式
    ```
    -id=1
    --id=1
    -id 1
    --id 1
    ```
    在使用中有两种方式，传入参数地址或返回参数地址。

    ```go
    flag.Narg()  // 返回非标志类参数个数
    flag.Nflag()  // 返回标志类参数个数
    ```

# 非标准库

1. ## [cobra 库 ](https://github.com/sp13/cobra)

    - ### 简介
    
         Cobra既是一个用来创建强大的现代CLI命令行的golang库，也是一个生成程序应用和命令行文件的程序。
        ![Cobra 演示程序](https://cloud.githubusercontent.com/assets/173412/10911369/84832a8e-8212-11e5-9f82-cc96660a4794.gif) 
    - ### cobra提供的功能

        - 简易的子命令行模式，如 app server， app fetch等等
        - 完全兼容posix命令行模式
        - 嵌套子命令subcommand
        - 支持全局，局部，串联flags
        - 使用Cobra很容易的生成应用程序和命令，使用cobra create appname和cobra add cmdname
        - 如果命令输入错误，将提供智能建议，如 app srver，将提示srver没有，是否是app server
        - 自动生成commands和flags的帮助信息
        - 自动生成详细的help信息，如app help
        - 自动识别-h，--help帮助flag
        - 自动生成应用程序在bash下命令自动完成功能
        - 自动生成应用程序的man手册
        - 命令行别名
        - 自定义help和usage信息
        - 可选的紧密集成的viper apps
    - ### 安装
        ```bash
        go get -v github.com/spf13/cobra/cobra
        ```
    - ### 使用cobra生产示例程序
        ```bash
        > $GOPATH/bin/cobra init demo
        Your Cobra application is ready at
        /home/ubuntu/go/src/demo.

        Give it a try by going there and running `go run main.go`.
        Add commands to it by running `cobra add [cmdname]`.
        ```
        生成的示例程序在[ $GOPATH/src/demo ]()目录下。
    - ### 实现一个没有子命令的CLI 程序
        新建一个包demo/imp
        ```go
        // imp/imp.go
        package imp

        import(
            "fmt"
        )

        func Show(name string, age int) {
            fmt.Printf("My name is %s, my age is %d\n", name, age)
        }
        ```
        // 修改cmd/root.go文件
        ```go
        // cmd/root.go
        // Copyright @ 2017 NAME HERE <EMAIL ADDRESS>
        //
        // Licensed under the Apache License, Version 2.0 (the "License");
        // you may not use this file except in compliance with the License.
        // You may obtain a copy of the License at
        //
        //     http://www.apache.org/licenses/LICENSE-2.0
        //
        // Unless required by applicable law or agreed to in writing, software
        // distributed under the License is distributed on an "AS IS" BASIS,
        // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        // See the License for the specific language governing permissions and
        // limitations under the License.

        package cmd

        import (
            "demo/imp"
            "fmt"
            "os"

            "github.com/spf13/cobra"
        )

        var cfgFile string

        var name string
        var age int

        // rootCmd represents the base command when called without any subcommands
        var rootCmd = &cobra.Command{
            Use:   "demo",
            Short: "A brief description of your application",
            Long: `A longer description that spans multiple lines and likely contains
            examples and usage of using your application. For example:

            Cobra is a CLI library for Go that empowers applications.
            This application is a tool to generate the needed files
            to quickly create a Cobra application.`,
            // Uncomment the following line if your bare application
            // has an action associated with it:
            Run: func(cmd *cobra.Command, args []string) {
                if len(name) == 0 {
                    cmd.Help()
                    return
                }
                imp.Show(name, age)
            },
        }

        // Execute adds all child commands to the root command and sets flags appropriately.
        // This is called by main.main(). It only needs to happen once to the rootCmd.
        func Execute() {
            if err := rootCmd.Execute(); err != nil {
                fmt.Println(err)
                os.Exit(1)
            }
        }

        func init() {
            // cobra.OnInitialize(initConfig)

            // Here you will define your flags and configuration settings.
            // Cobra supports persistent flags, which, if defined here,
            // will be global for your application.
            // rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.demo.yaml)")

            // Cobra also supports local flags, which will only run
            // when this action is called directly.
            //rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
            rootCmd.Flags().StringVarP(&name, "name", "n", "", "Please give your name")
            rootCmd.Flags().IntVar(&age, "age", 0, "Please give your age")
        }

        // initConfig reads in config file and ENV variables if set.
        /* func initConfig() { */
            // if cfgFile != "" {
            //     // Use config file from the flag.
            //     viper.SetConfigFile(cfgFile)
            // } else {
            //     // Find home directory.
            //     home, err := homedir.Dir()
            //     if err != nil {
            //         fmt.Println(err)
            //         os.Exit(1)
            //     }
            //
            //     // Search config in home directory with name ".demo" (without extension).
            //     viper.AddConfigPath(home)
            //     viper.SetConfigName(".demo")
            // }
            //
            // viper.AutomaticEnv() // read in environment variables that match
            //
            // // If a config file is found, read it in.
            // if err := viper.ReadInConfig(); err == nil {
            //     fmt.Println("Using config file:", viper.ConfigFileUsed())
            // }
        /* } */
        ```
        编译运行如下：
        ```bash
        $ ./main
        A longer description that spans multiple lines and likely contains
        examples and usage of using your application. For example:

        Cobra is a CLI library for Go that empowers applications.
        This application is a tool to generate the needed files
        to quickly create a Cobra application.

        Usage:
        demo [flags]

        Flags:
        --age int       Please give your age
        -h, --help          help for demo
        -n, --name string   Please give your name
        $ ./main -n zt --age 20
        My name is zt, my age is 20
        $ ./main --name zt --age 20
        My name is zt, my age is 20
        ```
    - ### 实现带有子命令的CLIs程序
        使用cobraa添加子命令
        ```bash
        $ ~/go/bin/cobra add test
        test created at /home/ubuntu/go/src/demo/cmd/test.go
        ```
        生成的test命令在$GOPATH/src/demo/test.go
        ```go
        // Copyright @ 2017 NAME HERE <EMAIL ADDRESS>
        //
        // Licensed under the Apache License, Version 2.0 (the "License");
        // you may not use this file except in compliance with the License.
        // You may obtain a copy of the License at
        //
        //     http://www.apache.org/licenses/LICENSE-2.0
        //
        // Unless required by applicable law or agreed to in writing, software
        // distributed under the License is distributed on an "AS IS" BASIS,
        // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        // See the License for the specific language governing permissions and
        // limitations under the License.

        package cmd

        import (
            "fmt"

            "github.com/spf13/cobra"
        )

        // testCmd represents the test command
        var testCmd = &cobra.Command{
            Use:   "test",
            Short: "A brief description of your command",
            Long: `A longer description that spans multiple lines and likely contains examples
            and usage of using your command. For example:

            Cobra is a CLI library for Go that empowers applications.
            This application is a tool to generate the needed files
            to quickly create a Cobra application.`,
            Run: func(cmd *cobra.Command, args []string) {
                fmt.Println("test called")
            },
        }

        func init() {
            rootCmd.AddCommand(testCmd)

            // Here you will define your flags and configuration settings.

            // Cobra supports Persistent Flags which will work for this command
            // and all subcommands, e.g.:
            // testCmd.PersistentFlags().String("foo", "", "A help for foo")

            // Cobra supports local flags which will only run when this command
            // is called directly, e.g.:
            // testCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
        }
        ```
        // 可以自己在func init中配置参数， 在run中配置命令执行内容。
        运行如下
        ```bash
        $ ./main test
        test called
        $ ./main test -h
        A longer description that spans multiple lines and likely contains examples
        and usage of using your command. For example:

        Cobra is a CLI library for Go that empowers applications.
        This application is a tool to generate the needed files
        to quickly create a Cobra application.

        Usage:
        demo test [flags]

        Flags:
        -h, --help   help for test
        $
        ```
        此外，cobra还具有错误提示功能
        ```bash
        $ ./main t
        Error: unknown command "t" for "demo"

        Did you mean this?
            test

        Run 'demo --help' for usage.
        unknown command "t" for "demo"

        Did you mean this?
            test
        ```