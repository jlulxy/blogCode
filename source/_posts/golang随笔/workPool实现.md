---
title: workPool实现
date: 2020-01-03 19:07:12
tags: [goalng,workpool]
---
### 随手写了个workpool放上来

```
package taskPool

import (
	"golib/errors"
	"sync/atomic"
)

type Task struct {
	Handler func(v ...interface{})
	Params[]interface{}
}

type Pool struct {
	capacity  uint64
	runningWorkersNum  uint64
	state  int64
	taskC  chan *Task
	closeC chan bool
}

var ErrInvalidPoolkCap = errors.New("invalid pool cap")
const (
	RUNNING = 1
	STOPED  = 0
)



func NewPool (capacity uint64)(*Pool,error){
	if capacity <= 0 {
		return nil,ErrInvalidPoolkCap
	}
	return &Pool{
		capacity:capacity,
		state:RUNNING,
		taskC:make(chan *Task,capacity),
		closeC:make(chan bool),
	},nil
}

func (p *Pool) addRunningkNum(){
	atomic.AddUint64(&p.runningWorkersNum,1)
}
func (p *Pool) reduceRunningNum(){
	atomic.AddUint64(&p.runningWorkersNum,1)
}
func (p *Pool) GetRunningNum()uint64{
	return  atomic.LoadUint64(&p.runningWorkersNum)
}

func (p *Pool) GetCap() uint64{
	return  atomic.LoadUint64(&p.capacity)
}
func (p *Pool) run(){
	p.addRunningkNum()
	go func() {
		defer func() {
			p.reduceRunningNum()
		}()
		for {
			select {
				case  task := <- p.taskC:
					task.Handler(task.Params)
			     case <-p.closeC:
				return
			}
		}
	}()
}

func (p *Pool) Put (task *Task)error{
	if p.state == STOPED{
		return ErrInvalidPoolkCap
	}
	//容量不满再开一个携程处理
	if p.runningWorkersNum < p.GetCap(){
		p.run()
	}
	//放入任务
	p.taskC <- task
	return nil
}

func (p *Pool) Clsoe(){
	p.state = STOPED //没有并发操作
	//等待任务消费完毕
	for len(p.taskC) > 0{
	}
	p.closeC <- true
	close(p.taskC)
}

```