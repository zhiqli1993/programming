# Go语言Goroutine基础

## 📚 学习目标
掌握Go语言的轻量级协程Goroutine的基本概念、启动方式和调度机制，理解并发编程的核心思想。

---

## 1. Goroutine简介

### 1.1 什么是Goroutine
- Goroutine是Go语言的轻量级线程。
- Goroutine由Go运行时调度，而不是操作系统线程。
- 每个Goroutine的初始栈大小很小（约2KB），并且可以动态扩展。

---

## 2. Goroutine的启动

### 2.1 使用`go`关键字启动
```go
package main

import (
    "fmt"
    "time"
)

func printMessage(msg string) {
    for i := 0; i < 5; i++ {
        fmt.Println(msg)
        time.Sleep(time.Millisecond * 500)
    }
}

func main() {
    go printMessage("协程1")
    go printMessage("协程2")

    // 主协程等待
    time.Sleep(time.Second * 3)
    fmt.Println("主协程结束")
}
```

---

## 3. Goroutine的调度

### 3.1 G-M-P模型
- **Goroutine (G)**: 轻量级协程。
- **Machine (M)**: 操作系统线程。
- **Processor (P)**: 调度器的逻辑处理器。

### 3.2 Go调度器的工作原理
- Go调度器基于抢占式调度。
- Goroutine之间的切换是由Go运行时控制的。

---

## 4. Goroutine的同步问题

### 4.1 数据竞争
```go
package main

import (
    "fmt"
    "time"
)

var counter int

func increment() {
    for i := 0; i < 1000; i++ {
        counter++
    }
}

func main() {
    go increment()
    go increment()

    time.Sleep(time.Second)
    fmt.Println("最终计数器值:", counter) // 可能出现数据竞争
}
```

---

## 5. Goroutine的最佳实践

### 5.1 使用`sync.WaitGroup`等待协程完成
```go
package main

import (
    "fmt"
    "sync"
)

func printMessage(msg string, wg *sync.WaitGroup) {
    defer wg.Done() // 标记任务完成
    for i := 0; i < 5; i++ {
        fmt.Println(msg)
    }
}

func main() {
    var wg sync.WaitGroup

    wg.Add(2) // 添加两个任务
    go printMessage("协程1", &wg)
    go printMessage("协程2", &wg)

    wg.Wait() // 等待所有任务完成
    fmt.Println("所有协程完成")
}
```

---

## 6. 综合案例：简单并发任务处理
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("工人%d开始工作\n", id)
    time.Sleep(time.Second)
    fmt.Printf("工人%d完成工作\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }

    wg.Wait()
    fmt.Println("所有工人完成任务")
}
```

---

## 7. 学习检查点

- [ ] 理解Goroutine的基本概念
- [ ] 掌握Goroutine的启动方式
- [ ] 理解Go调度器的工作原理
- [ ] 能识别并解决数据竞争问题
- [ ] 能用`sync.WaitGroup`管理协程的同步

---

Goroutine是Go语言并发编程的核心，掌握这些基础将为后续的Channel通信和并发设计模式学习打下坚实基础。
