---
title: "[NSQ Learning]  Woker Pool"
short: "Learning NSQ about Worker Pool,  manages a pool of worker that process channels concurrently."
date: 24.Jan.2018
tags:
    - golang
    - server
    - nsq
    - notes
---

在nsq中有这么两个函数nsqd\nsqd.go\queueScanLoop nsqd\nsqd.go：queueScanLoop、resizePool、queueScanWorker。它使用一个worker pool 来处理任务。

大致如下：

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type WorkPool struct {
	waitGroup   WaitGroupWrapper
	exitChan    chan struct{}
	poolSize    int
	workPoolMax int
}

func (rb *WorkPool) resizePool(num int, workCh chan int, responseCh chan bool, closeCh chan struct{}) {
	fmt.Println("resizePool:", num)
	idealPoolSize := num
	if idealPoolSize < 1 {
		idealPoolSize = 1
	} else if idealPoolSize > rb.workPoolMax {
		idealPoolSize = rb.workPoolMax
	}
	for {
		if idealPoolSize == rb.poolSize {
			break
		} else if idealPoolSize < rb.poolSize {
			// contract
			closeCh <- struct{}{}
			rb.poolSize--
		} else {
			// expand
			rb.waitGroup.Wrap(func() { // 再开一个worker
				rb.worker(workCh, responseCh, closeCh)
			})
			rb.poolSize++
		}
	}
}

func (rb *WorkPool) worker(workCh chan int, responseCh chan bool, closeCh chan struct{}) {
	fmt.Println("New Worker")
	for {
		result := false
		select {
		case c := <-workCh:
			// do work
			if c%2 == 0 {
				result = true
			}
			responseCh <- result
		case <-closeCh:
			fmt.Println("Delete Worker")
			return
		}
	}
}

func main() {
	workCh := make(chan int, 20) // default 20
	responseCh := make(chan bool, 20)
	closeCh := make(chan struct{})

	workTicker := time.NewTicker(1 * time.Second)
	refreshTicker := time.NewTicker(2 * time.Second)

	rb := WorkPool{workPoolMax: 20, exitChan:make(chan struct{})}
	num := 10
	rb.resizePool(num, workCh, responseCh, closeCh)
	rb.waitGroup.Wrap(func() {
		for {
			select {
			case <-workTicker.C:
			case <-refreshTicker.C:
				num = rand.Intn(5)+1
				rb.resizePool(num, workCh, responseCh, closeCh)
				continue
			case <-rb.exitChan:
				goto exit
			}
			for _, i := range UniqRands(num, 10) {
				workCh <- i
			}
			for i := 0; i < num; i++ {
				fmt.Println(">response:", <-responseCh)
			}
		}
	exit:
		close(closeCh)
	})
	time.Sleep(10 * time.Second)
	close(rb.exitChan)
	fmt.Println("Stopping...")
	workTicker.Stop()
	refreshTicker.Stop()
	rb.waitGroup.Wait()
	close(workCh)
	close(responseCh)
	fmt.Println("Stop.")
}

func UniqRands(l int, n int) []int {
	set := make(map[int]struct{})
	nums := make([]int, 0, l)
	for {
		num := rand.Intn(n) // 0-n的随机数
		if _, ok := set[num]; !ok {
			set[num] = struct{}{}
			nums = append(nums, num)
		}
		if len(nums) == l {
			goto exit
		}
	}
exit:
	return nums
}

```

运行结果：

```bash
resizePool: 10
New Worker
New Worker
New Worker
New Worker
New Worker
New Worker
New Worker
New Worker
New Worker
New Worker
>response: false
>response: false
>response: true
>response: true
>response: true
>response: true
>response: false
>response: false
>response: false
>response: true
>response: true
>response: true
>response: true
>response: true
>response: false
>response: false
>response: false
>response: false
>response: false
>response: true
resizePool: 4
Delete Worker
Delete Worker
Delete Worker
Delete Worker
Delete Worker
Delete Worker
>response: true
>response: false
>response: true
>response: true
>response: true
>response: true
>response: false
>response: false
resizePool: 1
Delete Worker
Delete Worker
Delete Worker
>response: false
>response: false
resizePool: 3
New Worker
New Worker
>response: false
>response: false
>response: false
>response: true
>response: false
>response: false
resizePool: 4
New Worker
>response: false
>response: false
>response: true
>response: true
>response: true
>response: true
>response: false
>response: false
resizePool: 1
Delete Worker
Delete Worker
Delete Worker
Stopping...
Delete Worker
Stop.
```

这是一个worker Pool 的简单实现，可以充分利用cpu时间进行处理任务。又可以将works 池数量控制在一个可控的范围内。