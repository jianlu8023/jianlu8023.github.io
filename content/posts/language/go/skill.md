+++
title = 'golang技巧'
date = '2025-04-28T17:42:18+08:00'
author = 'jianlu'
draft = false
description = "golang开发技巧"
aliases = ["golang", "skill"]
+++

<a id="top"></a>
---

目录

* [并发获取一个值](#1)
* [并发获取全部](#2)

----

## 并发获取一个值

<a id="1"></a>

```go
package main

import (
	"fmt"
	"log"
	"math/rand"
	"sync"
	"time"
)

func getOneResult(tasks []func() (interface{}, error)) (interface{}, error) {
	var (
		done       = make(chan struct{})
		once       sync.Once
		errChan    = make(chan error, len(tasks))
		resultChan = make(chan interface{}, 1)
		wg         sync.WaitGroup
		closeOnce  = func() {
			once.Do(func() {
				close(done) // 所有查询完成，关闭 done channel
			})
		}
	)
	
	defer close(errChan)
	defer close(resultChan)
	wg.Add(len(tasks))
	
	for _, task := range tasks {
		task := task // 避免闭包引用同一个变量
		go func() {
			defer wg.Done()
			select {
			case <-done:
				log.Println("task done")
			default:
				value, err := task()
				if err != nil {
					log.Println("task error:", err)
					errChan <- err
					return
				}
				select {
				case resultChan <- value:
					log.Println("task success")
					closeOnce()
				case <-done:
					// log.Println("task done")
				}
			}
		}()
	}
	
	// 等待所有 goroutine 完成
	wg.Wait()
	
	select {
	case r := <-resultChan:
		return r, nil
	default:
		// 收集所有错误
		var errs []error
		for i := 0; i < len(tasks); i++ {
			if err, ok := <-errChan; ok {
				errs = append(errs, err)
			}
		}
		return nil, fmt.Errorf("all tasks failed: %v", errs)
	}
}

func main() {
	tasks := []func() (interface{}, error){
		func() (interface{}, error) {
			duration := time.Duration(rand.Intn(6-3) + 3)
			log.Printf("task 1 sleep %s", duration)
			time.Sleep(duration * time.Second)
			return "task 1", nil
		},
		func() (interface{}, error) {
			duration := time.Duration(rand.Intn(6-3) + 3)
			log.Printf("task 2 sleep %s", duration)
			time.Sleep(duration * time.Second)
			return "task 2", nil
		},
		func() (interface{}, error) {
			duration := time.Duration(rand.Intn(6-3) + 3)
			log.Printf("task 3 sleep %s", duration)
			time.Sleep(duration * time.Second)
			return "task 3", nil
		},
	}
	
	now := time.Now()
	result, err := getOneResult(tasks)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
	sub := time.Now().Sub(now)
	log.Printf("get one use %s", sub)
}
```

[top](#top)

## 并发获取全部

<a id="2"></a>

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/rand"
	"sync"
	"time"
	
	"golang.org/x/sync/errgroup"
)

func main() {
	// 示例任务：模拟从不同的数据源获取数据
	task1 := func() (interface{}, error) {
		duration := time.Duration(rand.Intn(6-3) + 3)
		log.Printf("task 1 sleep %v seconds", duration)
		time.Sleep(duration * time.Second)
		return "Data from source 1", nil
	}
	
	task2 := func() (interface{}, error) {
		duration := time.Duration(rand.Intn(6-3) + 3)
		log.Printf("task 2 sleep %v seconds", duration)
		time.Sleep(duration * time.Second)
		return "Data from source 2", nil
	}
	
	task3 := func() (interface{}, error) {
		duration := time.Duration(rand.Intn(6-3) + 3)
		log.Printf("task 3 sleep %v seconds", duration)
		time.Sleep(duration * time.Second)
		return "Data from source 3", nil
	}
	
	tasks := []func() (interface{}, error){task1, task2, task3}
	
	// 并发获取所有可用的值
	results, err := getMoreResult(tasks)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Results:", results) // 输出: Results: [Data from source 2 Data from source 1 Data from source 3] (顺序可能不同)
	}
}

func getMoreResult(tasks []func() (interface{}, error)) ([]interface{}, error) {
	var (
		resultChan = make(chan interface{}, len(tasks))
		results    = make([]interface{}, 0, len(tasks))
		mu         sync.Mutex // 保护 results 切片
		eg, egCtx  = errgroup.WithContext(context.Background())
	)
	
	defer close(resultChan)
	
	// 启动 goroutine 执行任务
	for _, task := range tasks {
		task := task // 避免闭包问题
		eg.Go(func() error {
			select {
			case <-egCtx.Done():
				fmt.Println("任务被取消")
				return egCtx.Err()
			default:
				// 执行任务
				value, err := task()
				if err != nil {
					fmt.Println("任务执行失败")
					return err
				}
				
				select {
				case <-egCtx.Done():
					fmt.Println("任务被取消")
					return egCtx.Err()
				case resultChan <- value:
					fmt.Println("成功获取到值")
					return nil
				}
			}
		})
	}
	
	// 等待所有 goroutine 完成
	if err := eg.Wait(); err != nil {
		fmt.Println("一些任务执行失败")
		return nil, err
	}
	
	// 收集结果
	for i := 0; i < len(tasks); i++ {
		select {
		case result := <-resultChan:
			mu.Lock()
			results = append(results, result)
			mu.Unlock()
		default:
			// 如果没有值可读取，则跳过
			fmt.Println("No result received")
		}
	}
	
	return results, nil
}

```

[top](#top)
