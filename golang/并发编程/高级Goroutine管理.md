# Go语言高级Goroutine管理

## 📚 学习目标
掌握Go语言中Goroutine的高级管理技术，包括生命周期控制、限制和调度优化，以及处理常见的并发陷阱。

---

## 1. Goroutine生命周期管理

### 1.1 优雅终止Goroutine
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("工作者 %d 收到取消信号，正在退出...\n", id)
            return
        default:
            fmt.Printf("工作者 %d 正在工作\n", id)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    // 创建可取消的上下文
    ctx, cancel := context.WithCancel(context.Background())
    
    // 启动多个goroutine
    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }
    
    // 让goroutines运行5秒
    time.Sleep(5 * time.Second)
    
    // 发送取消信号
    fmt.Println("发送取消信号...")
    cancel()
    
    // 给goroutines时间退出
    time.Sleep(2 * time.Second)
    fmt.Println("程序退出")
}
```

### 1.2 处理Panic
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func safeWorker(id int, wg *sync.WaitGroup) {
    // 确保wg.Done()被调用
    defer wg.Done()
    
    // 捕获可能的panic
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("工作者 %d 恢复自panic: %v\n", id, r)
        }
    }()
    
    // 模拟工作和潜在panic
    if id == 2 {
        fmt.Printf("工作者 %d 即将panic\n", id)
        panic("模拟错误")
    }
    
    fmt.Printf("工作者 %d 正常完成\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    // 启动5个worker
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go safeWorker(i, &wg)
    }
    
    // 等待所有worker完成
    wg.Wait()
    fmt.Println("所有工作者已完成，即使有panic发生")
}
```

---

## 2. Goroutine数量控制

### 2.1 使用信号量限制并发
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// 定义信号量结构体
type Semaphore struct {
    capacity int
    taken    int
    mu       sync.Mutex
    cond     *sync.Cond
}

// 创建新信号量
func NewSemaphore(capacity int) *Semaphore {
    s := &Semaphore{capacity: capacity}
    s.cond = sync.NewCond(&s.mu)
    return s
}

// 获取信号量
func (s *Semaphore) Acquire() {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    for s.taken >= s.capacity {
        s.cond.Wait() // 等待直到有可用资源
    }
    
    s.taken++
}

// 释放信号量
func (s *Semaphore) Release() {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    s.taken--
    s.cond.Signal() // 通知等待的goroutine
}

func worker(id int, sem *Semaphore, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("工作者 %d 尝试获取资源\n", id)
    sem.Acquire()
    defer sem.Release()
    
    fmt.Printf("工作者 %d 获得资源，开始工作\n", id)
    time.Sleep(2 * time.Second) // 模拟工作
    fmt.Printf("工作者 %d 完成工作，释放资源\n", id)
}

func main() {
    const totalWorkers = 10
    const concurrencyLimit = 3
    
    // 创建信号量，限制最大并发为3
    sem := NewSemaphore(concurrencyLimit)
    var wg sync.WaitGroup
    
    for i := 1; i <= totalWorkers; i++ {
        wg.Add(1)
        go worker(i, sem, &wg)
    }
    
    wg.Wait()
    fmt.Println("所有工作者完成")
}
```

### 2.2 使用带缓冲的Channel限制并发
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, limit chan struct{}, wg *sync.WaitGroup) {
    defer wg.Done()
    
    // 获取令牌
    limit <- struct{}{}
    fmt.Printf("工作者 %d 开始工作\n", id)
    
    // 模拟工作
    time.Sleep(2 * time.Second)
    
    fmt.Printf("工作者 %d 完成工作\n", id)
    
    // 释放令牌
    <-limit
}

func main() {
    const totalWorkers = 10
    const concurrencyLimit = 3
    
    // 创建限制通道
    limit := make(chan struct{}, concurrencyLimit)
    var wg sync.WaitGroup
    
    for i := 1; i <= totalWorkers; i++ {
        wg.Add(1)
        go worker(i, limit, &wg)
    }
    
    wg.Wait()
    fmt.Println("所有工作者完成")
}
```

---

## 3. Goroutine调度与性能

### 3.1 使用GOMAXPROCS控制CPU使用
```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func cpuIntensiveTask(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("任务 %d 开始在P=%d上运行\n", id, runtime.GOMAXPROCS(0))
    
    start := time.Now()
    // CPU密集型计算
    counter := 0
    for i := 0; i < 1000000000; i++ {
        counter++
    }
    
    elapsed := time.Since(start)
    fmt.Printf("任务 %d 完成，耗时：%v\n", id, elapsed)
}

func main() {
    // 获取CPU核心数
    numCPU := runtime.NumCPU()
    fmt.Printf("系统CPU核心数: %d\n", numCPU)
    
    // 设置可用的P数量
    runtime.GOMAXPROCS(numCPU)
    fmt.Printf("GOMAXPROCS设置为: %d\n", runtime.GOMAXPROCS(0))
    
    var wg sync.WaitGroup
    
    // 启动与CPU核心数量相同的goroutine
    for i := 1; i <= numCPU; i++ {
        wg.Add(1)
        go cpuIntensiveTask(i, &wg)
    }
    
    wg.Wait()
    fmt.Println("所有CPU密集型任务完成")
}
```

### 3.2 混合IO和CPU工作负载
```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
    "runtime"
    "sync"
    "time"
)

func cpuTask(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("CPU任务 %d 开始\n", id)
    start := time.Now()
    
    // CPU密集型工作
    for i := 0; i < 100000000; i++ {
        _ = i * i
    }
    
    elapsed := time.Since(start)
    fmt.Printf("CPU任务 %d 完成，耗时：%v\n", id, elapsed)
}

func ioTask(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("IO任务 %d 开始\n", id)
    start := time.Now()
    
    // IO密集型工作
    resp, err := http.Get("https://golang.org")
    if err == nil {
        defer resp.Body.Close()
        _, _ = ioutil.ReadAll(resp.Body)
    }
    
    elapsed := time.Since(start)
    fmt.Printf("IO任务 %d 完成，耗时：%v\n", id, elapsed)
}

func main() {
    numCPU := runtime.NumCPU()
    fmt.Printf("系统CPU核心数: %d\n", numCPU)
    
    var wg sync.WaitGroup
    
    // 启动CPU密集型任务
    for i := 1; i <= numCPU; i++ {
        wg.Add(1)
        go cpuTask(i, &wg)
    }
    
    // 启动更多IO密集型任务
    for i := 1; i <= numCPU*4; i++ {
        wg.Add(1)
        go ioTask(i, &wg)
    }
    
    wg.Wait()
    fmt.Println("所有任务完成")
}
```

---

## 4. Goroutine泄漏检测与预防

### 4.1 常见的Goroutine泄漏场景
```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func leakyFunction() {
    ch := make(chan int) // 无缓冲通道
    
    // 这个goroutine将永远阻塞，因为没有人从通道接收
    go func() {
        val := 1
        fmt.Println("Goroutine尝试发送值", val)
        ch <- val // 将永远阻塞在这里
        fmt.Println("这行永远不会执行")
    }()
    
    // 不从通道接收，也不关闭通道
    fmt.Println("函数返回，但goroutine泄漏了")
}

func properFunction() {
    ch := make(chan int)
    
    go func() {
        val := 1
        fmt.Println("Goroutine尝试发送值", val)
        ch <- val
        fmt.Println("发送成功")
    }()
    
    // 从通道接收值
    result := <-ch
    fmt.Println("接收到值:", result)
}

func timeoutFunction() {
    ch := make(chan int)
    
    go func() {
        val := 1
        fmt.Println("Goroutine尝试发送值", val)
        ch <- val
        fmt.Println("发送成功")
    }()
    
    // 使用select实现超时
    select {
    case result := <-ch:
        fmt.Println("接收到值:", result)
    case <-time.After(2 * time.Second):
        fmt.Println("操作超时")
    }
}

func printGoroutineCount(label string) {
    fmt.Printf("%s: goroutine数量 = %d\n", label, runtime.NumGoroutine())
}

func main() {
    printGoroutineCount("初始状态")
    
    leakyFunction()
    runtime.GC() // 尝试触发GC
    printGoroutineCount("泄漏函数后")
    
    properFunction()
    runtime.GC()
    printGoroutineCount("正确函数后")
    
    timeoutFunction()
    runtime.GC()
    printGoroutineCount("超时函数后")
    
    // 等待一段时间，观察goroutine数量
    time.Sleep(3 * time.Second)
    printGoroutineCount("等待后")
}
```

### 4.2 使用Context预防泄漏
```go
package main

import (
    "context"
    "fmt"
    "runtime"
    "time"
)

func leakyWorker() {
    // 启动一个永不退出的goroutine
    go func() {
        for {
            time.Sleep(time.Second)
            // 永远不会结束的工作
        }
    }()
}

func controlledWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("工作者收到取消信号，退出")
                return
            default:
                time.Sleep(time.Second)
                // 模拟工作
            }
        }
    }()
}

func main() {
    printGoroutineCount("初始状态")
    
    // 泄漏的goroutine
    leakyWorker()
    printGoroutineCount("泄漏工作者启动后")
    
    // 使用context控制的goroutine
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    controlledWorker(ctx)
    printGoroutineCount("受控工作者启动后")
    
    // 等待context超时
    time.Sleep(4 * time.Second)
    printGoroutineCount("context超时后")
    
    // 垃圾回收
    runtime.GC()
    printGoroutineCount("GC后")
}

func printGoroutineCount(label string) {
    fmt.Printf("%s: goroutine数量 = %d\n", label, runtime.NumGoroutine())
}
```

---

## 5. Goroutine实战：实现超时控制

### 5.1 超时控制模式
```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "time"
)

// 模拟长时间运行的操作
func longRunningOperation(ctx context.Context) (string, error) {
    // 创建一个结果通道
    resultCh := make(chan string)
    
    go func() {
        // 模拟随机耗时操作
        delay := time.Duration(rand.Intn(5)+1) * time.Second
        fmt.Printf("操作将花费 %v\n", delay)
        time.Sleep(delay)
        
        resultCh <- "操作结果"
    }()
    
    // 等待结果或超时
    select {
    case result := <-resultCh:
        return result, nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

func main() {
    // 设置随机数种子
    rand.Seed(time.Now().UnixNano())
    
    // 创建一个有3秒超时的上下文
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    fmt.Println("开始长时间操作，超时设置为3秒")
    
    // 执行操作并等待结果
    result, err := longRunningOperation(ctx)
    
    if err != nil {
        fmt.Printf("操作失败: %v\n", err)
    } else {
        fmt.Printf("操作成功: %s\n", result)
    }
}
```

### 5.2 并发超时重试模式
```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "time"
)

// 模拟可能失败或超时的服务调用
func callService(ctx context.Context) (string, error) {
    // 模拟50%的失败率
    if rand.Float32() < 0.5 {
        time.Sleep(time.Duration(rand.Intn(200)) * time.Millisecond)
        return "", fmt.Errorf("服务暂时不可用")
    }
    
    // 模拟服务耗时
    select {
    case <-time.After(time.Duration(rand.Intn(500)) * time.Millisecond):
        return "服务响应", nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

// 带有重试的服务调用
func callWithRetry(maxRetries int, timeout time.Duration) (string, error) {
    for attempt := 1; attempt <= maxRetries; attempt++ {
        // 创建超时上下文
        ctx, cancel := context.WithTimeout(context.Background(), timeout)
        
        fmt.Printf("尝试 %d/%d...\n", attempt, maxRetries)
        result, err := callService(ctx)
        
        // 释放上下文资源
        cancel()
        
        if err == nil {
            return result, nil
        }
        
        fmt.Printf("尝试 %d 失败: %v\n", attempt, err)
        
        // 最后一次尝试失败，返回错误
        if attempt == maxRetries {
            return "", fmt.Errorf("达到最大重试次数: %w", err)
        }
        
        // 退避策略: 随机延迟避免惊群效应
        backoff := time.Duration(rand.Intn(100)) * time.Millisecond
        fmt.Printf("等待 %v 后重试\n", backoff)
        time.Sleep(backoff)
    }
    
    return "", fmt.Errorf("不应该到达这里")
}

func main() {
    // 设置随机数种子
    rand.Seed(time.Now().UnixNano())
    
    // 最多重试5次，每次超时300毫秒
    result, err := callWithRetry(5, 300*time.Millisecond)
    
    if err != nil {
        fmt.Printf("最终操作失败: %v\n", err)
    } else {
        fmt.Printf("操作成功: %s\n", result)
    }
}
```

---

## 6. 学习检查点

- [ ] 理解goroutine的生命周期管理方式
- [ ] 掌握处理goroutine panic的技术
- [ ] 能够使用信号量和channel限制goroutine数量
- [ ] 理解GOMAXPROCS对性能的影响
- [ ] 能够识别和处理goroutine泄漏问题
- [ ] 掌握使用context实现超时控制

---

高级的Goroutine管理是构建健壮、高性能并发系统的关键。通过对Goroutine生命周期、数量和调度的精细控制，可以充分发挥Go语言的并发优势，同时避免常见的并发陷阱。
