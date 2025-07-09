# Go语言Context上下文

## 📚 学习目标
深入理解Context包的设计思想和使用方法，学会在并发程序中进行请求级别的控制和管理。

---

## 1. Context基础概念

### 1.1 什么是Context
Context（上下文）是Go语言用于在API边界和进程之间传递截止时间、取消信号和其他请求范围值的一种机制。

### 1.2 Context接口定义
```go
type Context interface {
    // Done返回一个通道，当Context被取消或超时时关闭
    Done() <-chan struct{}
    
    // Err返回Context被取消的原因
    Err() error
    
    // Deadline返回Context的截止时间
    Deadline() (deadline time.Time, ok bool)
    
    // Value返回与key关联的值
    Value(key interface{}) interface{}
}
```

### 1.3 Context设计理念
- Context是不可变的（immutable）
- Context可以衍生出子Context
- Context用于传递请求范围的值，不用于传递可选参数
- 同一个请求的不同阶段和Goroutine应共享同一个Context
- Context安全地被多个Goroutine同时使用

---

## 2. Context类型

### 2.1 根Context
```go
package main

import (
    "context"
    "fmt"
)

func main() {
    // 创建一个background context
    bgCtx := context.Background()
    fmt.Printf("Background: %v\n", bgCtx)
    
    // 创建一个todo context
    todoCtx := context.TODO()
    fmt.Printf("TODO: %v\n", todoCtx)
}
```

### 2.2 可取消Context
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 创建一个可取消的context
    ctx, cancel := context.WithCancel(context.Background())
    
    // 启动一个goroutine执行任务
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("任务被取消，退出")
                return
            default:
                fmt.Println("任务执行中...")
                time.Sleep(time.Second)
            }
        }
    }()
    
    // 等待5秒后取消任务
    time.Sleep(5 * time.Second)
    cancel()
    
    // 等待goroutine退出
    time.Sleep(time.Second)
}
```

### 2.3 超时Context
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 创建一个5秒超时的context
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("操作超时，退出")
                return
            default:
                fmt.Println("操作执行中...")
                time.Sleep(time.Second)
            }
        }
    }()
    
    // 等待超时发生
    <-ctx.Done()
    fmt.Println("主程序退出，错误:", ctx.Err())
}
```

### 2.4 截止时间Context
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 设置截止时间为5秒后
    deadline := time.Now().Add(5 * time.Second)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("截止时间到，退出")
                return
            default:
                fmt.Println("操作执行中...")
                time.Sleep(time.Second)
            }
        }
    }()
    
    // 等待截止时间到达
    <-ctx.Done()
    fmt.Println("主程序退出，错误:", ctx.Err())
}
```

### 2.5 值Context
```go
package main

import (
    "context"
    "fmt"
)

// 定义key类型
type key string

func main() {
    // 创建带值的context
    ctx := context.WithValue(context.Background(), key("user"), "alice")
    ctx = context.WithValue(ctx, key("role"), "admin")
    
    // 获取值
    user := ctx.Value(key("user")).(string)
    role := ctx.Value(key("role")).(string)
    
    fmt.Printf("用户: %s, 角色: %s\n", user, role)
    
    // 在函数中使用
    processRequest(ctx)
}

func processRequest(ctx context.Context) {
    // 从context中获取值
    if user, ok := ctx.Value(key("user")).(string); ok {
        fmt.Printf("处理来自用户 %s 的请求\n", user)
    }
    
    // 检查角色
    if role, ok := ctx.Value(key("role")).(string); ok && role == "admin" {
        fmt.Println("执行管理员操作")
    }
}
```

---

## 3. Context传播

### 3.1 父子Context关系
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 创建根context
    rootCtx := context.Background()
    
    // 创建第一个子context
    child1, cancel1 := context.WithTimeout(rootCtx, 5*time.Second)
    defer cancel1()
    
    // 创建第二个子context
    child2, cancel2 := context.WithCancel(child1)
    defer cancel2()
    
    go func() {
        <-child2.Done()
        fmt.Println("child2 已完成:", child2.Err())
    }()
    
    go func() {
        <-child1.Done()
        fmt.Println("child1 已完成:", child1.Err())
    }()
    
    // 取消child1，会级联取消child2
    time.Sleep(2 * time.Second)
    fmt.Println("取消child1")
    cancel1()
    
    // 等待goroutine完成
    time.Sleep(time.Second)
}
```

### 3.2 取消信号传播
```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func worker(ctx context.Context, id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("工人%d开始工作\n", id)
    
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("工人%d收到取消信号: %v\n", id, ctx.Err())
            return
        default:
            // 模拟工作
            fmt.Printf("工人%d正在工作\n", id)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    // 创建可取消的context
    ctx, cancel := context.WithCancel(context.Background())
    
    // 启动3个工人
    var wg sync.WaitGroup
    wg.Add(3)
    for i := 1; i <= 3; i++ {
        go worker(ctx, i, &wg)
    }
    
    // 3秒后发送取消信号
    time.Sleep(3 * time.Second)
    fmt.Println("主程序发送取消信号")
    cancel()
    
    // 等待所有工人退出
    wg.Wait()
    fmt.Println("所有工人已退出")
}
```

### 3.3 超时和截止时间传播
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func executeWithTimeout(ctx context.Context) {
    // 创建衍生的子context，超时时间为2秒
    timeoutCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()
    
    // 执行一些可能超时的操作
    go func() {
        // 检查是否从父context继承了值
        if userID, ok := ctx.Value("userID").(string); ok {
            fmt.Printf("子操作: 用户ID为 %s\n", userID)
        }
        
        // 模拟耗时操作
        select {
        case <-timeoutCtx.Done():
            fmt.Printf("子操作: 上下文完成: %v\n", timeoutCtx.Err())
        case <-time.After(3 * time.Second):
            fmt.Println("子操作: 工作完成")
        }
    }()
    
    // 等待子操作完成或超时
    <-timeoutCtx.Done()
    fmt.Printf("executeWithTimeout: 上下文完成: %v\n", timeoutCtx.Err())
}

func main() {
    // 创建根context，并添加值
    rootCtx := context.WithValue(context.Background(), "userID", "user-123")
    
    // 执行带超时的操作
    executeWithTimeout(rootCtx)
}
```

### 3.4 值传递机制
```go
package main

import (
    "context"
    "fmt"
)

// 为避免冲突，通常使用自定义类型作为key
type userIDKey struct{}
type authTokenKey struct{}

func enrichContext(ctx context.Context) context.Context {
    // 添加用户ID
    ctx = context.WithValue(ctx, userIDKey{}, "user-123")
    
    // 添加认证令牌
    ctx = context.WithValue(ctx, authTokenKey{}, "token-abc")
    
    return ctx
}

func processWithAuth(ctx context.Context) {
    // 获取用户ID
    userID, ok := ctx.Value(userIDKey{}).(string)
    if !ok {
        fmt.Println("未找到用户ID")
        return
    }
    
    // 获取认证令牌
    token, ok := ctx.Value(authTokenKey{}).(string)
    if !ok {
        fmt.Println("未找到认证令牌")
        return
    }
    
    fmt.Printf("处理来自用户 %s 的请求，令牌: %s\n", userID, token)
    
    // 创建子context并传递给其他函数
    subOperation(ctx)
}

func subOperation(ctx context.Context) {
    // 子操作也可以访问context中的值
    if userID, ok := ctx.Value(userIDKey{}).(string); ok {
        fmt.Printf("子操作: 用户ID为 %s\n", userID)
    }
}

func main() {
    // 创建基础context
    ctx := context.Background()
    
    // 丰富context
    ctx = enrichContext(ctx)
    
    // 使用context进行处理
    processWithAuth(ctx)
}
```

---

## 4. Context与HTTP请求

### 4.1 HTTP请求中的Context
```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
)

func slowHandler(w http.ResponseWriter, r *http.Request) {
    // 获取请求中的context
    ctx := r.Context()
    
    // 模拟慢操作
    select {
    case <-time.After(5 * time.Second):
        fmt.Fprintln(w, "操作完成")
    case <-ctx.Done():
        // 客户端取消请求
        fmt.Println("请求被取消")
        return
    }
}

func main() {
    http.HandleFunc("/slow", slowHandler)
    
    fmt.Println("服务器启动在 :8080")
    http.ListenAndServe(":8080", nil)
}
```

### 4.2 使用Context控制HTTP客户端
```go
package main

import (
    "context"
    "fmt"
    "io/ioutil"
    "net/http"
    "time"
)

func fetchURL(ctx context.Context, url string) ([]byte, error) {
    // 创建带有context的请求
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    // 发送请求
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    // 读取响应
    return ioutil.ReadAll(resp.Body)
}

func main() {
    // 创建超时context
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    // 尝试获取可能超时的URL
    data, err := fetchURL(ctx, "https://httpbin.org/delay/3")
    if err != nil {
        fmt.Printf("请求失败: %v\n", err)
        return
    }
    
    fmt.Printf("收到 %d 字节的数据\n", len(data))
}
```

---

## 5. Context最佳实践

### 5.1 不要将Context存储在结构体中
Context应该作为函数的第一个参数传递，不应该存储在结构体中，除非该结构体表示一个请求。

```go
// 正确的做法
func ProcessRequest(ctx context.Context, data []byte) error {
    // ...
}

// 错误的做法
type Service struct {
    ctx context.Context
    // ...
}
```

### 5.2 Context参数命名为ctx
按照惯例，Context参数通常命名为ctx：

```go
func DoSomething(ctx context.Context, arg Arg) error {
    // ...
}
```

### 5.3 Context应该是第一个参数
Context通常作为函数的第一个参数传递：

```go
// 正确的做法
func FetchData(ctx context.Context, id string) (*Data, error) {
    // ...
}

// 不推荐的做法
func FetchData(id string, ctx context.Context) (*Data, error) {
    // ...
}
```

### 5.4 不要传递nil Context
如果不确定使用哪个Context，使用context.TODO()：

```go
// 正确的做法
func DoSomething() {
    ctx := context.TODO()
    ProcessRequest(ctx, data)
}

// 错误的做法
func DoSomething() {
    ProcessRequest(nil, data) // 不要传递nil
}
```

### 5.5 Context值只用于请求范围的数据
Context的Value应该只用于传递请求特定的数据，不要用于传递可选参数：

```go
// 正确的做法 - 传递请求ID或跟踪信息
ctx = context.WithValue(ctx, RequestIDKey, requestID)

// 错误的做法 - 传递函数配置
ctx = context.WithValue(ctx, "timeout", 30) // 应该使用函数参数
```

### 5.6 Context取消是建议性的
Context的取消是建议性的，某些操作可能无法立即取消：

```go
func longOperation(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            // 尝试清理资源
            return ctx.Err()
        default:
            // 执行一小部分工作
            err := doChunk()
            if err != nil {
                return err
            }
        }
    }
}
```

---

## 6. 综合案例：并发请求控制

### 6.1 同时请求多个API并控制超时
```go
package main

import (
    "context"
    "errors"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// 模拟API请求
func fetchAPI(ctx context.Context, apiID int) (string, error) {
    // 随机延迟，模拟不同API的响应时间
    delay := time.Duration(rand.Intn(5)+1) * time.Second
    
    select {
    case <-ctx.Done():
        return "", ctx.Err()
    case <-time.After(delay):
        return fmt.Sprintf("API-%d的响应", apiID), nil
    }
}

// 并发请求多个API
func fetchAllAPIs(ctx context.Context, timeout time.Duration) ([]string, error) {
    // 创建子context，设置超时
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()
    
    var results []string
    var mu sync.Mutex
    var wg sync.WaitGroup
    
    // 跟踪错误
    errors := make(chan error, 3)
    
    // 发起3个并发请求
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(apiID int) {
            defer wg.Done()
            
            result, err := fetchAPI(ctx, apiID)
            if err != nil {
                select {
                case errors <- fmt.Errorf("API-%d错误: %w", apiID, err):
                default:
                    // 错误通道已满，忽略
                }
                return
            }
            
            mu.Lock()
            results = append(results, result)
            mu.Unlock()
        }(i)
    }
    
    // 等待所有请求完成或超时
    done := make(chan struct{})
    go func() {
        wg.Wait()
        close(done)
    }()
    
    // 等待完成或超时
    select {
    case <-done:
        return results, nil
    case <-ctx.Done():
        return results, ctx.Err()
    case err := <-errors:
        return results, err
    }
}

func main() {
    // 设置随机数种子
    rand.Seed(time.Now().UnixNano())
    
    // 创建基础context
    ctx := context.Background()
    
    // 添加请求ID
    ctx = context.WithValue(ctx, "requestID", "req-123")
    
    fmt.Println("开始并发请求API...")
    
    // 请求多个API，总超时为4秒
    results, err := fetchAllAPIs(ctx, 4*time.Second)
    
    if err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            fmt.Println("请求超时")
        } else {
            fmt.Printf("请求失败: %v\n", err)
        }
    }
    
    fmt.Printf("收到 %d 个API响应\n", len(results))
    for _, result := range results {
        fmt.Println(result)
    }
}
```

### 6.2 带Context的工作池
```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// 任务结构
type Task struct {
    ID int
}

// 结果结构
type Result struct {
    TaskID int
    Value  string
}

// 工作池
func workerPool(ctx context.Context, tasks <-chan Task, results chan<- Result, numWorkers int) {
    var wg sync.WaitGroup
    
    // 启动指定数量的工作者
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(ctx, tasks, results, i, &wg)
    }
    
    // 等待所有工作者完成
    go func() {
        wg.Wait()
        close(results)
    }()
}

// 工作者函数
func worker(ctx context.Context, tasks <-chan Task, results chan<- Result, workerID int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for {
        select {
        case task, ok := <-tasks:
            if !ok {
                // 任务通道已关闭
                return
            }
            
            // 处理任务
            result, err := processTask(ctx, task, workerID)
            if err != nil {
                fmt.Printf("工作者%d处理任务%d失败: %v\n", workerID, task.ID, err)
                continue
            }
            
            // 发送结果
            select {
            case results <- result:
                // 结果已发送
            case <-ctx.Done():
                fmt.Printf("工作者%d: 上下文取消，停止发送结果\n", workerID)
                return
            }
            
        case <-ctx.Done():
            fmt.Printf("工作者%d: 上下文取消，退出\n", workerID)
            return
        }
    }
}

// 处理单个任务
func processTask(ctx context.Context, task Task, workerID int) (Result, error) {
    fmt.Printf("工作者%d开始处理任务%d\n", workerID, task.ID)
    
    // 模拟随机处理时间
    processingTime := time.Duration(rand.Intn(500)+100) * time.Millisecond
    
    select {
    case <-time.After(processingTime):
        result := Result{
            TaskID: task.ID,
            Value:  fmt.Sprintf("任务%d的结果，由工作者%d处理", task.ID, workerID),
        }
        fmt.Printf("工作者%d完成任务%d\n", workerID, task.ID)
        return result, nil
        
    case <-ctx.Done():
        return Result{}, ctx.Err()
    }
}

func main() {
    // 设置随机数种子
    rand.Seed(time.Now().UnixNano())
    
    // 创建可取消的context
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    // 创建任务和结果通道
    tasks := make(chan Task, 10)
    results := make(chan Result)
    
    // 启动工作池，5个工作者
    go workerPool(ctx, tasks, results, 5)
    
    // 发送10个任务
    go func() {
        for i := 1; i <= 10; i++ {
            tasks <- Task{ID: i}
        }
        close(tasks)
    }()
    
    // 收集结果
    var collected []Result
    for result := range results {
        collected = append(collected, result)
        fmt.Printf("收到结果: %s\n", result.Value)
    }
    
    fmt.Printf("总共处理了 %d 个任务\n", len(collected))
}
```

---

## 7. 学习检查点

- [ ] 理解Context的基本概念和设计理念
- [ ] 掌握各种Context类型及其使用场景
- [ ] 能正确处理Context的传播和取消信号
- [ ] 理解Context值的传递机制和最佳实践
- [ ] 能在HTTP客户端和服务器中使用Context
- [ ] 能设计基于Context的并发控制系统

---

Context是Go语言处理请求范围值、取消信号和截止时间的标准方式，掌握它对于编写健壮的并发程序至关重要。通过正确使用Context，你可以更好地控制并发操作，提高程序的可维护性和可靠性。
