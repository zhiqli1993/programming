# HTTP网络编程

Go语言的`net/http`包提供了强大而简洁的HTTP客户端和服务器实现，是构建网络应用的核心组件。

## HTTP服务器

### 基本HTTP服务器

```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, Go HTTP!")
}

func main() {
    http.HandleFunc("/hello", helloHandler)
    http.ListenAndServe(":8080", nil)
}
```

### 使用ServeMux路由

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/api/users", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Users API")
    })
    
    mux.HandleFunc("/api/products", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Products API")
    })
    
    http.ListenAndServe(":8080", mux)
}
```

### 中间件模式

```go
package main

import (
    "log"
    "net/http"
    "time"
)

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        log.Printf("Started %s %s", r.Method, r.URL.Path)
        
        next.ServeHTTP(w, r)
        
        log.Printf("Completed %s %s in %v", r.Method, r.URL.Path, time.Since(start))
    })
}

func main() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/api/data", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Data endpoint"))
    })
    
    wrappedMux := loggingMiddleware(mux)
    http.ListenAndServe(":8080", wrappedMux)
}
```

### 静态文件服务

```go
package main

import "net/http"

func main() {
    fs := http.FileServer(http.Dir("./static"))
    http.Handle("/static/", http.StripPrefix("/static/", fs))
    http.ListenAndServe(":8080", nil)
}
```

## HTTP客户端

### 基本GET请求

```go
package main

import (
    "fmt"
    "io"
    "net/http"
)

func main() {
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("Error reading body:", err)
        return
    }
    
    fmt.Println("Status:", resp.Status)
    fmt.Println("Body:", string(body))
}
```

### POST请求

```go
package main

import (
    "bytes"
    "fmt"
    "io"
    "net/http"
    "strings"
)

func main() {
    // 使用字符串
    resp1, err := http.Post(
        "https://api.example.com/users",
        "application/json",
        strings.NewReader(`{"name":"John","email":"john@example.com"}`),
    )
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    defer resp1.Body.Close()
    
    // 使用bytes.Buffer
    data := []byte(`{"name":"Alice","email":"alice@example.com"}`)
    resp2, err := http.Post(
        "https://api.example.com/users",
        "application/json",
        bytes.NewBuffer(data),
    )
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    defer resp2.Body.Close()
}
```

### 自定义HTTP客户端

```go
package main

import (
    "net/http"
    "time"
)

func main() {
    client := &http.Client{
        Timeout: 10 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            MaxIdleConnsPerHost: 10,
            IdleConnTimeout:     30 * time.Second,
        },
    }
    
    req, _ := http.NewRequest("GET", "https://api.example.com/data", nil)
    req.Header.Add("Authorization", "Bearer token123")
    
    resp, err := client.Do(req)
    if err != nil {
        // 处理错误
        return
    }
    defer resp.Body.Close()
    
    // 处理响应
}
```

## HTTP服务器性能优化

### 保持连接

```go
package main

import "net/http"

func main() {
    server := &http.Server{
        Addr:         ":8080",
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }
    server.ListenAndServe()
}
```

### 使用HTTP/2

```go
package main

import "net/http"

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", nil)
}

func handler(w http.ResponseWriter, r *http.Request) {
    // HTTP/2会自动启用，无需额外配置
    w.Write([]byte("Hello from HTTP/2 server"))
}
```

## 测试HTTP服务器

```go
package main

import (
    "io"
    "net/http"
    "net/http/httptest"
    "testing"
)

func HelloHandler(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "Hello, world!")
}

func TestHelloHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/hello", nil)
    w := httptest.NewRecorder()
    
    HelloHandler(w, req)
    
    resp := w.Result()
    body, _ := io.ReadAll(resp.Body)
    
    if resp.StatusCode != http.StatusOK {
        t.Errorf("Expected status OK; got %v", resp.Status)
    }
    
    if string(body) != "Hello, world!" {
        t.Errorf("Expected 'Hello, world!'; got %v", string(body))
    }
}
```

## 安全最佳实践

1. 始终验证并清理用户输入
2. 使用HTTPS (TLS)
3. 设置适当的超时值
4. 实现速率限制
5. 添加安全相关的HTTP头
6. 避免在URL中包含敏感信息
7. 使用上下文(context)进行请求取消
