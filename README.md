
```

## golang 实现多个worker去解决大量的任务

- 可以设置worker在队列空的时候结束
- 设置单worker每秒最大执行job数量
- 提供友好的协程同步控制机制
- 可以灵活的自定义job

## 使用方式

```
```
package main

import (
	"bytes"
	"fmt"
	"runtime"
	"strconv"
	"github.com/loticket/gworker"
	"sync"
	"time"
)

func GetGID() uint64 {
	b := make([]byte, 64)
	b = b[:runtime.Stack(b, false)]
	b = bytes.TrimPrefix(b, []byte("goroutine "))
	b = b[:bytes.IndexByte(b, ' ')]
	n, _ := strconv.ParseUint(string(b), 10, 64)
	return n
}

type MyJob struct {
	data string
}

func (j *MyJob) GetName() string {
	return "my_job"
}

func (j *MyJob) RetryCount() int {
	return 1
}

func (j *MyJob) Delay() time.Duration {
	return 0
}

func NewMyJob(data string) *MyJob {
	return &MyJob{
		data: data,
	}
}

func (j *MyJob) Handle() error {
	fmt.Printf("GID:%d ,%s\n", GetGID(), j.data)
	return nil
}

func main() {

	var runTotal = 0
	var total = 1000
	mutex := sync.Mutex{}
	pool := gworker.NewWorkerPool(nil, time.Second*5, 10,
		func(err error, job gworker.Job) {

		},
		func(worker gworker.Worker, job gworker.Jobber) {
			mutex.Lock()
			runTotal++
			mutex.Unlock()
		})

	pool.PreSecondDealNum(1)
	pool.Run()
	for i := 1; i <= total; i++ {
		fmt.Println("add job")
		job := NewMyJob(fmt.Sprintf("id: %d", i))
		pool.Push(job)
	}

	pool.Stop()
	if runTotal != total {
		fmt.Println("push job num %d not equal run job num %d", total, runTotal)
	}

}

```

## 函数方法

### workerPool

```
    //push job  into workerPool
    Push(job Job) error
    //开始运行wokerPool
   	Run()
   	//停止 ,这里可能会阻塞
   	Stop() 
   	//回收一个空闲的worker
   	RecycleWorker(worker Worker)
   	//获取状态
   	Status() uint
   	//获取出错执行的函数
   	GetErrorHandle() ErrorHandle
   	//设置出错回调函数(panic时候触发)
   	SetErrorHandle(ErrorHandle)
   	//设置每秒处理任务数量
   	PreSecondDealNum(num int)
    //workerPool 是否已经停止运行
    IsStop() bool
    //设置任务执行完成后的回调函数
	SetJobRunOverHandle(JobRunOverHandle)
    //获取任务执行完成回调
	GetJobRunOverHandle() JobRunOverHandle
```








