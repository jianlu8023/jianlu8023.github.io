+++
title = '长安链相关开发注意事项'
date = '2025-06-25T11:20:47+08:00'
author = 'jianlu'
draft = false
description = "长安链相关开发注意事项"
aliases = ["blockchain", "chainmaker", "tips"]
+++

## 使用sdk 订阅时注意事项

```go
package main

import (
	chainmakercommon "chainmaker.org/chainmaker/pb-go/v2/common"
	chainmakersdkgo "chainmaker.org/chainmaker/sdk-go/v2"
	"context"
	"log"
	"os"
	"os/signal"
	"sync"
	"sync/atomic"
	"syscall"
	"time"
)

var (
	innerSub atomic.Pointer[Subscribe]
	once     sync.Once
)

type Subscribe struct {
	startBlock    int64
	endBlock      int64
	chaincodeName string
	topicName     string
	start         atomic.Bool
	exit          chan struct{}
	timerExit     chan struct{}
	sync.Mutex
	needRestart atomic.Bool
	sdk         *chainmakersdkgo.ChainClient
}

func (s *Subscribe) SetStartBlock(startBlock int64) {
	s.startBlock = startBlock
}

func (s *Subscribe) SetEndBlock(endBlock int64) {
	s.endBlock = endBlock
}

func (s *Subscribe) Restart() {
	s.killCurrentSub()
	s.needRestart.Store(true)
}

func (s *Subscribe) killCurrentSub() {
	defer func() {
		log.Printf("[killCurrentSub] kill current sub success, release lock ...")
		s.Unlock()
	}()
	s.Lock()
	select {
	case s.exit <- struct{}{}:
		log.Printf("[Stop] call Stop(), send a signal to exit channel...")
	default:
		log.Printf("[Stop] subscribe contract %s topic %s exit channel is full or closed", s.chaincodeName, s.topicName)
	}
}

func NewSubscribe(startBlock, endBlock int64, chaincodeName, topicName string, sdk *chainmakersdkgo.ChainClient) *Subscribe {
	once.Do(func() {
		sub := Subscribe{
			startBlock:    startBlock,
			endBlock:      endBlock,
			chaincodeName: chaincodeName,
			topicName:     topicName,
			start:         atomic.Bool{},
			exit:          make(chan struct{}, 1),
			timerExit:     make(chan struct{}, 1),
			sdk:           sdk,
			needRestart:   atomic.Bool{},
		}
		sub.start.Store(false)
		sub.needRestart.Store(false)
		log.Printf("[NewSubscribe] start timerChecker ...")
		go sub.timerChecker(5)
		innerSub.Store(&sub)
	})
	return innerSub.Load()
}

func (s *Subscribe) Start(ctx context.Context) error {
	defer func() {
		log.Printf("[Start] finish start subscribe,release lock ...")
		s.Unlock()
	}()
	s.Lock()
	log.Printf("[Start] get lock, start subscribe ...")
	if s.start.Load() {
		log.Printf("[Start] already start subscribe ...")
		return nil
	}
	event, err := s.sdk.SubscribeBlock(ctx, s.startBlock, s.endBlock, false, false)
	if err != nil {
		log.Printf("[Start] subscribe chainmaker block failed error %v", err)
		return err
	}
	
	go func() {
		for {
			select {
			case _, ok := <-s.exit:
				if ok {
					log.Printf("[Start] exit subscribe ...")
					return
				}
			case e, ok := <-event:
				if !ok {
					// 防止订阅过程中出现报错的情况
					log.Printf("[Start] event channel closed ...")
					s.needRestart.Store(true)
					return
				}
				if e == nil {
					log.Printf("[Start] event is nil ...")
					continue
				}
				
				// topicEvent, ok := e.(*chainmakercommon.ContractEventInfoList)
				// tx,ok:=e.(*chainmakercommon.Transaction)
				blockInfo, ok := e.(*chainmakercommon.BlockInfo)
				if !ok {
					log.Printf("[Start] can not convert event to *chainmakercommon.BlockInfo, raw event %v", e)
					continue
				}
				log.Printf("[Start] receive new block %v", blockInfo)
				// 具体代码
			}
		}
	}()
	
	log.Printf("[Start] subscribe chainmaker block success setting start = true ...")
	s.start.Store(true)
	return nil
}

func (s *Subscribe) timerChecker(interval int64) {
	ticker := time.NewTicker(time.Duration(interval) * time.Second)
	for {
		select {
		case <-ticker.C:
			log.Printf("[timerChecker] check need restart ...")
			if s.needRestart.Load() {
				s.start.Store(false)
				// 可增加重设startBlock的代码
				// s.SetStartBlock(999)
				if err := s.Start(context.Background()); err != nil {
					log.Printf("[timerChecker] restart subscribe failed error %s", err)
				} else {
					log.Printf("[timerChecker] restart subscribe success ...")
					s.needRestart.Store(false)
				}
			}
		case _, ok := <-s.timerExit:
			if ok {
				log.Printf("[timerChecker] recevice exit signal ...")
				return
			}
		}
	}
}

func (s *Subscribe) Stop() {
	defer func() {
		close(s.exit)
		close(s.timerExit)
		if rec := recover(); rec != nil {
			log.Printf("[Stop] stop subscribe contract %s topic %s event panic, err: %s", s.chaincodeName, s.topicName, rec)
		}
	}()
	select {
	case s.exit <- struct{}{}:
		log.Printf("[Stop] call Stop(), send a signal to exit channel...")
	default:
		log.Printf("[Stop] subscribe contract %s topic %s exit channel is full or closed", s.chaincodeName, s.topicName)
	}
	select {
	case s.timerExit <- struct{}{}:
		log.Printf("[Stop] call Stop(), send a signal to timerExit channel...")
	default:
		log.Printf("[Stop] subscribe contract %s topic %s timerExit channel is full or closed", s.chaincodeName, s.topicName)
	}
}

func StopSubscribe() {
	sub := innerSub.Load()
	if sub != nil {
		sub.Stop()
		sub.start.Store(false)
	}
}

func main() {
	defer StopSubscribe()
	if err := NewSubscribe(0, -1, "fact", "fact_tx", nil).
		Start(context.Background()); err != nil {
		log.Printf("[main] start subscribe failed error %v", err)
		return
	}
	
	exit := make(chan os.Signal, 1)
	signal.Notify(exit, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)
	select {
	case <-exit:
		log.Printf("[main] exit signal received")
		return
	}
}

```
