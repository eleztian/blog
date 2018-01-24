---
title: "[NSQ Learning]  Start And End A Service Of Golang"
short: "Learning Nsq to use go-svc and waitgroup to stat and end a service of golang. it's easy to manage goroutine."
date: 24.Jan.2018
tags:
    - golang
    - server
    - nsq
    - notes
---
在nsq中使用go-svc: [https://github.com/judwhite/go-svc](https://github.com/judwhite/go-svc)来启动程序和结束程序。

它的实现方式如下：

```go
package main
 
import (
    "fmt"
    "os"
    "os/signal"
    "syscall"
)
 
func main() {
    c := make(chan os.Signal, 1)
    sig := []os.Signal{syscall.SIGINT, syscall.SIGTERM}
    fmt.Println("start")
    signal.Notify(c, sig...)
    fmt.Println("processA")
 
    //程序会阻塞在这里, 等待系统信号量
    s := <-c
    fmt.Println("processB")
    fmt.Println("Got signal:", s)
}
```

程序ctrl+c后结果如下：

```bash
processB
Got signal: interrupt
```

结合nsq里面goroutine的管理方式，可以向如下使用：

## waitgroupwraper.go

```go
import "sync"

type WaitGroupWrapper struct {
	sync.WaitGroup
}

func (w *WaitGroupWrapper)Wrap( fb func())  {
	w.Add(1)
	go func() {
		fb()
		w.Done()
	}()
}
```

## server.go

```go
import (
	"log"
	"time"
)

type server struct {
	data chan int

	exitChan  chan struct{}
	waitGroup WaitGroupWrapper
}

func (s *server) start() {
	s.data = make(chan int)
	s.exitChan = make(chan struct{})
	s.waitGroup.Wrap(func() {
		s.startSender()
	})
	s.waitGroup.Wrap(func() {
		s.startReceiver()
	})
}

func (s *server) stop() error {
    // 发出结束信号
    close(s.exitChan)
    // 等待所有goroutine结束。
	s.waitGroup.Wait()
	return nil
}

func (s *server) startSender() {
	ticker := time.NewTicker(20 * time.Millisecond)
	count := 1
	for {
		select {
		case <-ticker.C:
			select {
			case s.data <- count:
				count++
			case <-s.exitChan:
				// if the other goroutine exits there'll be no one to receive on the data chan,
				// and this goroutine could block. you can simulate this by putting a time.Sleep
				// inside startReceiver's receive on s.data and a log.Println here
				return
			}
		case <-s.exitChan:
			return
		}
	}
}

func (s *server) startReceiver() {
	for {
		select {
		case n := <-s.data:
			log.Printf("%d\n", n)
		case <-s.exitChan:
			return
		}
	}
}
```

## main.go

```go
import (
	"log"
	"os"
	"path/filepath"

	"syscall"

	"github.com/judwhite/go-svc/svc"
)

// implements svc.Service
type program struct {
	LogFile *os.File
	svr     *server
}

func main() {
	prg := program{
		svr: &server{},
	}
	defer func() {
		if prg.LogFile != nil {
			prg.LogFile.Close()
		}
	}()

	// call svc.Run to start your program/service
	// svc.Run will call Init, Start, and Stop
	if err := svc.Run(&prg, syscall.SIGINT, syscall.SIGTERM); err != nil {
		log.Fatal(err)
	}
}

func (p *program) Init(env svc.Environment) error {
	log.Printf("is win service? %v\n", env.IsWindowsService())

	// write to "example.log" when running as a Windows Service
	if env.IsWindowsService() {
		dir, err := filepath.Abs(filepath.Dir(os.Args[0]))
		if err != nil {
			return err
		}

		logPath := filepath.Join(dir, "example.log")

		f, err := os.OpenFile(logPath, os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
		if err != nil {
			return err
		}

		p.LogFile = f

		log.SetOutput(f)
	}

	return nil
}

func (p *program) Start() error {
	log.Printf("Starting...\n")
	go p.svr.start()
	return nil
}

func (p *program) Stop() error {
	log.Printf("Stopping...\n")
	if err := p.svr.stop(); err != nil {
		return err
	}
	log.Printf("Stopped.\n")
	return nil
}

```

## 运行结果

```bash
2018/01/24 10:03:52 is win service? false
2018/01/24 10:03:52 Starting...
2018/01/24 10:03:52 1
2018/01/24 10:03:52 2
2018/01/24 10:03:52 3
2018/01/24 10:03:52 4
2018/01/24 10:03:52 5
2018/01/24 10:03:52 6
2018/01/24 10:03:52 7
...
2018/01/24 10:03:55 188
2018/01/24 10:03:55 189
^c
2018/01/24 10:03:55 Stopping...
2018/01/24 10:03:55 Stopped.
```

如上的实现使得goroutine的管理变得简单。