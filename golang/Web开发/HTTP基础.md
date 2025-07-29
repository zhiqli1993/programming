# HTTP基础

## 概述

HTTP（超文本传输协议）是Web应用程序中最常用的应用层协议，Go语言提供了强大而灵活的HTTP支持。本文档将介绍在Go中处理HTTP协议的核心概念和实践。

## HTTP协议核心概念

### 请求与响应模型

HTTP协议基于请求-响应模型：
- **客户端**（通常是浏览器）发送请求
- **服务器**处理请求并返回响应

### HTTP方法

常用的HTTP方法包括：
```
GET    - 获取资源
POST   - 提交数据
PUT    - 更新资源
DELETE - 删除资源
PATCH  - 部分更新资源
HEAD   - 获取头信息
OPTIONS- 获取支持的方法
```

### HTTP状态码

HTTP状态码分为五类：
- **1xx** - 信息性响应
- **2xx** - 成功响应（如200 OK）
- **3xx** - 重定向响应
- **4xx** - 客户端错误（如404 Not Found）
- **5xx** - 服务器错误

### HTTP头部

HTTP头部包含请求和响应的元数据：
```
Content-Type  - 指定内容类型
Content-Length- 内容长度
Authorization - 认证信息
User-Agent    - 客户端标识
Cookie        - 客户端存储的会话数据
```

## Go中的HTTP客户端

### 基本GET请求

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    // 发起GET请求
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        fmt.Println("请求错误:", err)
        return
    }
    defer resp.Body.Close()

    // 读取响应体
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("读取响应错误:", err)
        return
    }

    // 输出状态码和响应内容
    fmt.Println("状态码:", resp.StatusCode)
    fmt.Println("响应体:", string(body))
}
```

### 自定义HTTP客户端

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func main() {
    // 创建自定义客户端
    client := &http.Client{
        Timeout: 10 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            MaxIdleConnsPerHost: 10,
            IdleConnTimeout:     90 * time.Second,
        },
    }

    // 使用自定义客户端发起请求
    resp, err := client.Get("https://api.example.com/data")
    if err != nil {
        fmt.Println("请求错误:", err)
        return
    }
    defer resp.Body.Close()

    // 处理响应...
}
```

### POST请求与JSON数据

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func main() {
    // 准备请求数据
    data := map[string]interface{}{
        "name":  "张三",
        "email": "zhangsan@example.com",
        "age":   30,
    }
    
    // 转换为JSON
    jsonData, err := json.Marshal(data)
    if err != nil {
        fmt.Println("JSON编码错误:", err)
        return
    }
    
    // 创建请求
    req, err := http.NewRequest("POST", "https://api.example.com/users", bytes.NewBuffer(jsonData))
    if err != nil {
        fmt.Println("创建请求错误:", err)
        return
    }
    
    // 设置头部
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer token123")
    
    // 发送请求
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println("发送请求错误:", err)
        return
    }
    defer resp.Body.Close()
    
    // 处理响应...
}
```

## Go中的HTTP服务器

### 创建基本HTTP服务器

```go
package main

import (
    "fmt"
    "net/http"
)

// 处理函数
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, 世界!")
}

func main() {
    // 注册路由
    http.HandleFunc("/hello", helloHandler)
    
    // 启动服务器
    fmt.Println("服务器启动在 :8080...")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("服务器启动失败:", err)
    }
}
```

### 自定义ServeMux路由

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    // 创建自定义ServeMux
    mux := http.NewServeMux()
    
    // 注册路由处理函数
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "首页")
    })
    
    mux.HandleFunc("/about", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "关于我们")
    })
    
    mux.HandleFunc("/users/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "用户: %s", r.URL.Path[7:])
    })
    
    // 启动带有自定义ServeMux的服务器
    server := &http.Server{
        Addr:    ":8080",
        Handler: mux,
    }
    
    fmt.Println("服务器启动在 :8080...")
    err := server.ListenAndServe()
    if err != nil {
        fmt.Println("服务器启动失败:", err)
    }
}
```

### 静态文件服务

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    // 创建文件服务器
    fs := http.FileServer(http.Dir("./static"))
    
    // 注册静态文件处理
    http.Handle("/static/", http.StripPrefix("/static/", fs))
    
    // 主页
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "访问 /static/ 目录查看静态文件")
    })
    
    fmt.Println("服务器启动在 :8080...")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("服务器启动失败:", err)
    }
}
```

## HTTP中间件实现

### 日志中间件

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

// 日志中间件
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 请求前记录
        start := time.Now()
        log.Printf("开始 %s %s", r.Method, r.URL.Path)
        
        // 调用下一个处理器
        next.ServeHTTP(w, r)
        
        // 请求后记录
        log.Printf("完成 %s %s in %v", r.Method, r.URL.Path, time.Since(start))
    })
}

func main() {
    // 创建处理函数
    helloHandler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, 世界!")
    })
    
    // 应用中间件
    http.Handle("/hello", loggingMiddleware(helloHandler))
    
    fmt.Println("服务器启动在 :8080...")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("服务器启动失败:", err)
    }
}
```

### 认证中间件

```go
package main

import (
    "fmt"
    "net/http"
)

// 认证中间件
func authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 检查授权头
        token := r.Header.Get("Authorization")
        if token != "Bearer valid-token" {
            http.Error(w, "未授权", http.StatusUnauthorized)
            return
        }
        
        // 已授权，继续处理
        next.ServeHTTP(w, r)
    })
}

func main() {
    // 受保护的处理函数
    protectedHandler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "这是受保护的资源!")
    })
    
    // 应用认证中间件
    http.Handle("/protected", authMiddleware(protectedHandler))
    
    // 公开的处理函数
    http.HandleFunc("/public", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "这是公开资源!")
    })
    
    fmt.Println("服务器启动在 :8080...")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("服务器启动失败:", err)
    }
}
```

## 请求解析

### URL参数获取

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/search", func(w http.ResponseWriter, r *http.Request) {
        // 获取URL查询参数
        query := r.URL.Query().Get("q")
        page := r.URL.Query().Get("page")
        
        if page == "" {
            page = "1" // 默认值
        }
        
        fmt.Fprintf(w, "搜索: %s, 页码: %s", query, page)
    })
    
    fmt.Println("服务器启动在 :8080...")
    http.ListenAndServe(":8080", nil)
}
```

### 表单解析

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
        if r.Method != "POST" {
            http.Error(w, "仅支持POST方法", http.StatusMethodNotAllowed)
            return
        }
        
        // 解析表单数据
        err := r.ParseForm()
        if err != nil {
            http.Error(w, "解析表单失败", http.StatusBadRequest)
            return
        }
        
        // 获取表单值
        username := r.FormValue("username")
        password := r.FormValue("password")
        
        // 简单验证
        if username == "admin" && password == "password" {
            fmt.Fprintf(w, "登录成功，欢迎 %s!", username)
        } else {
            http.Error(w, "用户名或密码错误", http.StatusUnauthorized)
        }
    })
    
    fmt.Println("服务器启动在 :8080...")
    http.ListenAndServe(":8080", nil)
}
```

### JSON请求解析

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

// 用户结构体
type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

func main() {
    http.HandleFunc("/api/users", func(w http.ResponseWriter, r *http.Request) {
        if r.Method != "POST" {
            http.Error(w, "仅支持POST方法", http.StatusMethodNotAllowed)
            return
        }
        
        // 检查内容类型
        if r.Header.Get("Content-Type") != "application/json" {
            http.Error(w, "仅接受 application/json", http.StatusUnsupportedMediaType)
            return
        }
        
        // 解析JSON
        var user User
        decoder := json.NewDecoder(r.Body)
        err := decoder.Decode(&user)
        if err != nil {
            http.Error(w, "无效的JSON", http.StatusBadRequest)
            return
        }
        
        // 处理数据
        fmt.Printf("接收到用户: %+v\n", user)
        
        // 返回JSON响应
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "status":  "success",
            "message": fmt.Sprintf("用户 %s 已创建", user.Name),
            "id":      123, // 假设的ID
        })
    })
    
    fmt.Println("服务器启动在 :8080...")
    http.ListenAndServe(":8080", nil)
}
```

## Cookie与Session管理

### Cookie操作

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func main() {
    // 设置Cookie的处理函数
    http.HandleFunc("/set-cookie", func(w http.ResponseWriter, r *http.Request) {
        // 创建Cookie
        cookie := http.Cookie{
            Name:     "session_id",
            Value:    "abc123",
            Path:     "/",
            HttpOnly: true,
            MaxAge:   3600, // 1小时
            // Secure: true, // 仅HTTPS
            // Domain: "example.com",
        }
        
        // 设置Cookie
        http.SetCookie(w, &cookie)
        
        fmt.Fprintln(w, "Cookie已设置!")
    })
    
    // 读取Cookie的处理函数
    http.HandleFunc("/get-cookie", func(w http.ResponseWriter, r *http.Request) {
        // 读取Cookie
        cookie, err := r.Cookie("session_id")
        if err != nil {
            if err == http.ErrNoCookie {
                fmt.Fprintln(w, "没有找到Cookie")
                return
            }
            http.Error(w, "服务器错误", http.StatusInternalServerError)
            return
        }
        
        fmt.Fprintf(w, "Cookie值: %s", cookie.Value)
    })
    
    // 删除Cookie的处理函数
    http.HandleFunc("/delete-cookie", func(w http.ResponseWriter, r *http.Request) {
        // 创建过期Cookie
        cookie := http.Cookie{
            Name:    "session_id",
            Value:   "",
            Path:    "/",
            MaxAge:  -1,
            Expires: time.Unix(0, 0),
        }
        
        // 设置过期Cookie
        http.SetCookie(w, &cookie)
        
        fmt.Fprintln(w, "Cookie已删除!")
    })
    
    fmt.Println("服务器启动在 :8080...")
    http.ListenAndServe(":8080", nil)
}
```

### 简单Session实现

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

// 会话管理器
type SessionManager struct {
    sessions map[string]map[string]interface{} // 会话存储
    mu       sync.RWMutex                     // 读写锁
}

// 创建新会话管理器
func NewSessionManager() *SessionManager {
    return &SessionManager{
        sessions: make(map[string]map[string]interface{}),
    }
}

// 创建新会话
func (sm *SessionManager) CreateSession() string {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    // 简化处理: 使用时间戳作为会话ID
    sessionID := fmt.Sprintf("%d", time.Now().UnixNano())
    sm.sessions[sessionID] = make(map[string]interface{})
    
    return sessionID
}

// 获取会话
func (sm *SessionManager) GetSession(sessionID string) (map[string]interface{}, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    
    session, ok := sm.sessions[sessionID]
    return session, ok
}

// 删除会话
func (sm *SessionManager) DeleteSession(sessionID string) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    delete(sm.sessions, sessionID)
}

// 全局会话管理器
var globalSessions = NewSessionManager()

func main() {
    // 登录处理
    http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
        username := r.FormValue("username")
        password := r.FormValue("password")
        
        // 验证用户名和密码 (示例中简化处理)
        if username == "admin" && password == "123456" {
            // 创建新会话
            sessionID := globalSessions.CreateSession()
            
            // 存储会话数据
            session, _ := globalSessions.GetSession(sessionID)
            session["username"] = username
            session["loginTime"] = time.Now()
            
            // 设置Cookie
            cookie := http.Cookie{
                Name:     "session_id",
                Value:    sessionID,
                HttpOnly: true,
                Path:     "/",
                MaxAge:   3600, // 1小时
            }
            http.SetCookie(w, &cookie)
            
            fmt.Fprintf(w, "登录成功，欢迎 %s!", username)
        } else {
            http.Error(w, "用户名或密码错误", http.StatusUnauthorized)
        }
    })
    
    // 受保护的资源
    http.HandleFunc("/profile", func(w http.ResponseWriter, r *http.Request) {
        // 获取会话ID
        cookie, err := r.Cookie("session_id")
        if err != nil {
            http.Redirect(w, r, "/login", http.StatusFound)
            return
        }
        
        // 获取会话数据
        session, ok := globalSessions.GetSession(cookie.Value)
        if !ok {
            http.Redirect(w, r, "/login", http.StatusFound)
            return
        }
        
        // 显示用户信息
        username := session["username"].(string)
        loginTime := session["loginTime"].(time.Time)
        
        fmt.Fprintf(w, "用户资料: %s\n", username)
        fmt.Fprintf(w, "登录时间: %s\n", loginTime.Format("2006-01-02 15:04:05"))
    })
    
    // 注销
    http.HandleFunc("/logout", func(w http.ResponseWriter, r *http.Request) {
        cookie, err := r.Cookie("session_id")
        if err == nil {
            // 删除服务器会话
            globalSessions.DeleteSession(cookie.Value)
            
            // 清除客户端Cookie
            expiredCookie := http.Cookie{
                Name:   "session_id",
                Value:  "",
                Path:   "/",
                MaxAge: -1,
            }
            http.SetCookie(w, &expiredCookie)
        }
        
        fmt.Fprintln(w, "已注销")
    })
    
    fmt.Println("服务器启动在 :8080...")
    http.ListenAndServe(":8080", nil)
}
```

## 总结

本文档介绍了Go语言中HTTP编程的基础知识，包括：

1. HTTP协议的基本概念
2. Go中实现HTTP客户端
3. Go中构建HTTP服务器
4. 中间件的实现与应用
5. 请求解析与处理
6. Cookie与Session管理

掌握这些基础知识后，你可以进一步学习Web框架（如Gin、Echo）以及更高级的Web开发主题。

## 推荐阅读

- [Go标准库http包文档](https://golang.org/pkg/net/http/)
- [HTTP MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
