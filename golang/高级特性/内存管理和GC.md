# Go语言内存管理和GC

## 📚 学习目标
掌握Go语言的内存管理机制，理解垃圾回收的工作原理和性能优化方法。

---

## 1. 内存分配

### 1.1 栈分配 vs 堆分配
```go
package main

import "fmt"

func stackAllocation() int {
    x := 42 // 栈分配
    return x
}

func heapAllocation() *int {
    x := 42 // 堆分配
    return &x
}

func main() {
    fmt.Printf("栈分配: %d\n", stackAllocation())
    fmt.Printf("堆分配: %d\n", *heapAllocation())
}
```

---

## 2. 逃逸分析

### 2.1 什么是逃逸分析
- 逃逸分析是Go编译器用于决定变量分配位置的技术。
- 如果变量在函数返回后仍然被引用，则分配到堆，否则分配到栈。

### 2.2 查看逃逸分析结果
```bash
# 使用以下命令查看逃逸分析结果
go build -gcflags="-m" main.go
```

---

## 3. 垃圾回收

### 3.1 垃圾回收的工作原理
- Go语言使用**三色标记法**进行垃圾回收。
- 垃圾回收器会标记活动对象并清除不再使用的对象。

### 3.2 手动触发垃圾回收
```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Println("手动触发垃圾回收")
    runtime.GC()
}
```

---

## 4. 内存优化

### 4.1 使用对象池
```go
package main

import (
    "fmt"
    "sync"
)

var pool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024) // 分配1KB的内存
    },
}

func main() {
    buf := pool.Get().([]byte) // 从对象池获取
    fmt.Println("使用对象池分配内存")

    pool.Put(buf) // 归还对象池
}
```

### 4.2 减少垃圾回收压力
- 使用对象池减少频繁的内存分配和释放。
- 尽量避免短生命周期的对象。

---

## 5. 综合案例：内存管理优化
```go
package main

import (
    "fmt"
    "sync"
)

type Cache struct {
    data map[string]string
    mux  sync.RWMutex
}

func (c *Cache) Get(key string) string {
    c.mux.RLock()
    defer c.mux.RUnlock()
    return c.data[key]
}

func (c *Cache) Set(key, value string) {
    c.mux.Lock()
    defer c.mux.Unlock()
    c.data[key] = value
}

func main() {
    cache := Cache{data: make(map[string]string)}

    var wg sync.WaitGroup
    wg.Add(2)

    go func() {
        defer wg.Done()
        cache.Set("name", "Go语言")
    }()

    go func() {
        defer wg.Done()
        fmt.Println("读取缓存:", cache.Get("name"))
    }()

    wg.Wait()
}
```

---

## 6. 学习检查点

- [ ] 理解栈分配和堆分配的区别
- [ ] 掌握逃逸分析的作用和使用方法
- [ ] 理解垃圾回收的工作原理
- [ ] 能用对象池优化内存分配
- [ ] 能用内存管理优化程序性能

---

内存管理和垃圾回收是Go语言的核心特性之一，掌握这些基础将显著提升程序的性能和稳定性。
