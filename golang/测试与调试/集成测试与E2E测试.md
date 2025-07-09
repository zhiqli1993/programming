# 集成测试与E2E测试

相比于单元测试关注代码的最小单元，集成测试和端到端（E2E）测试验证系统组件如何协同工作，以及完整的用户场景是否按预期运行。这些测试对于确保系统作为一个整体正常工作至关重要。

## 集成测试基础

### 什么是集成测试

集成测试验证多个组件或服务协同工作的能力。它们通常包括：

- 测试组件间的通信
- 验证数据流通过多个层级
- 确保不同部分的系统能够正确集成

### 与单元测试的区别

| 单元测试 | 集成测试 |
|---------|----------|
| 关注最小的代码单元 | 关注组件间的交互 |
| 通常使用Mock/Stub隔离依赖 | 通常使用真实或接近真实的依赖 |
| 运行快速 | 相对运行较慢 |
| 容易定位问题 | 问题定位可能更复杂 |
| 代码覆盖率高 | 场景覆盖更全面 |

## Go中的集成测试策略

### 使用测试标记区分测试类型

使用构建标记可以区分单元测试和集成测试：

```go
// integration_test.go
//go:build integration

package mypackage_test

import "testing"

func TestDatabaseIntegration(t *testing.T) {
    // 集成测试代码
}
```

运行集成测试：

```bash
go test -tags=integration ./...
```

### 测试辅助函数

创建辅助函数简化集成测试的设置：

```go
// test_helpers.go
package mypackage_test

import (
    "database/sql"
    "testing"
    
    _ "github.com/lib/pq"
)

// 测试数据库辅助结构
type TestDB struct {
    DB     *sql.DB
    t      *testing.T
    tables []string
}

// 设置测试数据库
func SetupTestDB(t *testing.T) *TestDB {
    db, err := sql.Open("postgres", "postgres://user:pass@localhost/testdb?sslmode=disable")
    if err != nil {
        t.Fatalf("连接测试数据库失败: %v", err)
    }
    
    return &TestDB{
        DB:     db,
        t:      t,
        tables: []string{},
    }
}

// 创建测试表
func (tdb *TestDB) CreateTable(tableName, schema string) {
    _, err := tdb.DB.Exec("CREATE TABLE IF NOT EXISTS " + tableName + " " + schema)
    if err != nil {
        tdb.t.Fatalf("创建表失败: %v", err)
    }
    tdb.tables = append(tdb.tables, tableName)
}

// 清理测试数据
func (tdb *TestDB) Cleanup() {
    for _, table := range tdb.tables {
        _, err := tdb.DB.Exec("DROP TABLE IF EXISTS " + table)
        if err != nil {
            tdb.t.Logf("清理表 %s 失败: %v", table, err)
        }
    }
    tdb.DB.Close()
}
```

### 使用测试容器

[testcontainers-go](https://github.com/testcontainers/testcontainers-go) 库提供了在测试中使用Docker容器的能力：

```go
package integration_test

import (
    "context"
    "database/sql"
    "testing"
    
    _ "github.com/lib/pq"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/wait"
)

func TestWithPostgres(t *testing.T) {
    ctx := context.Background()
    
    // 设置Postgres容器
    postgresContainer, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        "postgres:13",
            ExposedPorts: []string{"5432/tcp"},
            Env: map[string]string{
                "POSTGRES_USER":     "testuser",
                "POSTGRES_PASSWORD": "testpass",
                "POSTGRES_DB":       "testdb",
            },
            WaitingFor: wait.ForLog("database system is ready to accept connections"),
        },
        Started: true,
    })
    if err != nil {
        t.Fatalf("启动Postgres容器失败: %v", err)
    }
    
    // 确保测试结束时清理容器
    defer postgresContainer.Terminate(ctx)
    
    // 获取容器映射的端口
    port, err := postgresContainer.MappedPort(ctx, "5432")
    if err != nil {
        t.Fatalf("获取端口失败: %v", err)
    }
    
    host, err := postgresContainer.Host(ctx)
    if err != nil {
        t.Fatalf("获取主机失败: %v", err)
    }
    
    // 连接到容器中的数据库
    connStr := fmt.Sprintf("postgres://testuser:testpass@%s:%s/testdb?sslmode=disable", host, port.Port())
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        t.Fatalf("连接数据库失败: %v", err)
    }
    defer db.Close()
    
    // 运行集成测试...
    err = db.Ping()
    if err != nil {
        t.Fatalf("无法ping数据库: %v", err)
    }
    
    // 执行其他测试...
}
```

## 常见集成测试场景

### 数据库集成测试

```go
func TestUserRepository(t *testing.T) {
    if testing.Short() {
        t.Skip("跳过集成测试")
    }
    
    // 设置测试数据库
    db := SetupTestDB(t)
    defer db.Cleanup()
    
    db.CreateTable("users", `(
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TIMESTAMP DEFAULT NOW()
    )`)
    
    // 创建仓库实例
    repo := NewUserRepository(db.DB)
    
    // 测试创建用户
    user := User{Name: "Test User", Email: "test@example.com"}
    createdUser, err := repo.Create(user)
    if err != nil {
        t.Fatalf("创建用户失败: %v", err)
    }
    
    if createdUser.ID == 0 {
        t.Error("预期创建的用户有ID")
    }
    
    // 测试读取用户
    retrievedUser, err := repo.GetByID(createdUser.ID)
    if err != nil {
        t.Fatalf("获取用户失败: %v", err)
    }
    
    if retrievedUser.Name != user.Name || retrievedUser.Email != user.Email {
        t.Errorf("用户数据不匹配: 预期 %+v, 实际 %+v", user, retrievedUser)
    }
}
```

### API集成测试

```go
func TestAPIIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("跳过集成测试")
    }
    
    // 设置测试数据库
    db := SetupTestDB(t)
    defer db.Cleanup()
    
    // 初始化服务和路由
    userRepo := NewUserRepository(db.DB)
    userService := NewUserService(userRepo)
    router := SetupRouter(userService)
    
    // 创建测试服务器
    server := httptest.NewServer(router)
    defer server.Close()
    
    // 测试用户创建API
    createUserPayload := `{"name":"Test User","email":"test@example.com"}`
    resp, err := http.Post(
        server.URL+"/users",
        "application/json",
        strings.NewReader(createUserPayload),
    )
    if err != nil {
        t.Fatalf("请求失败: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusCreated {
        t.Errorf("预期状态码 %d, 实际 %d", http.StatusCreated, resp.StatusCode)
    }
    
    // 解析响应
    var createResponse struct {
        ID int `json:"id"`
    }
    json.NewDecoder(resp.Body).Decode(&createResponse)
    
    // 测试获取用户API
    getUserResp, err := http.Get(server.URL + fmt.Sprintf("/users/%d", createResponse.ID))
    if err != nil {
        t.Fatalf("请求失败: %v", err)
    }
    defer getUserResp.Body.Close()
    
    if getUserResp.StatusCode != http.StatusOK {
        t.Errorf("预期状态码 %d, 实际 %d", http.StatusOK, getUserResp.StatusCode)
    }
    
    var user User
    json.NewDecoder(getUserResp.Body).Decode(&user)
    
    if user.Name != "Test User" || user.Email != "test@example.com" {
        t.Errorf("用户数据不匹配: %+v", user)
    }
}
```

### 微服务集成测试

```go
func TestMicroserviceIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("跳过集成测试")
    }
    
    ctx := context.Background()
    
    // 启动依赖的服务容器
    authServiceContainer := startServiceContainer(ctx, t, "auth-service:latest", 8080)
    defer authServiceContainer.Terminate(ctx)
    
    paymentServiceContainer := startServiceContainer(ctx, t, "payment-service:latest", 8081)
    defer paymentServiceContainer.Terminate(ctx)
    
    // 获取服务URL
    authServiceURL := getServiceURL(ctx, t, authServiceContainer, 8080)
    paymentServiceURL := getServiceURL(ctx, t, paymentServiceContainer, 8081)
    
    // 配置测试客户端
    client := NewAPIClient(WithAuthService(authServiceURL), WithPaymentService(paymentServiceURL))
    
    // 执行跨服务的集成测试
    token, err := client.Authenticate("testuser", "password")
    if err != nil {
        t.Fatalf("认证失败: %v", err)
    }
    
    // 使用认证令牌进行支付
    paymentResult, err := client.ProcessPayment(token, 100.0, "USD", "4111111111111111")
    if err != nil {
        t.Fatalf("支付处理失败: %v", err)
    }
    
    if !paymentResult.Success {
        t.Errorf("支付应该成功: %+v", paymentResult)
    }
}

func startServiceContainer(ctx context.Context, t *testing.T, image string, port int) testcontainers.Container {
    container, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        image,
            ExposedPorts: []string{fmt.Sprintf("%d/tcp", port)},
            WaitingFor:   wait.ForHTTP("/health").WithPort(nat.Port(fmt.Sprintf("%d/tcp", port))),
        },
        Started: true,
    })
    if err != nil {
        t.Fatalf("启动容器失败 %s: %v", image, err)
    }
    return container
}

func getServiceURL(ctx context.Context, t *testing.T, container testcontainers.Container, port int) string {
    mappedPort, err := container.MappedPort(ctx, nat.Port(fmt.Sprintf("%d/tcp", port)))
    if err != nil {
        t.Fatalf("获取映射端口失败: %v", err)
    }
    
    host, err := container.Host(ctx)
    if err != nil {
        t.Fatalf("获取主机失败: %v", err)
    }
    
    return fmt.Sprintf("http://%s:%s", host, mappedPort.Port())
}
```

## E2E测试

### 什么是E2E测试

端到端测试验证整个应用程序从用户界面到数据存储的完整流程，模拟真实用户行为。

### E2E测试框架

在Go生态系统中，可以使用多种框架进行E2E测试：

#### Playwright/Selenium与Go

使用Playwright或Selenium测试Web应用程序：

```go
package e2e_test

import (
    "context"
    "testing"
    "time"
    
    "github.com/mxschmitt/playwright-go"
)

func TestUserSignup(t *testing.T) {
    // 启动Playwright
    pw, err := playwright.Run()
    if err != nil {
        t.Fatalf("无法启动Playwright: %v", err)
    }
    defer pw.Stop()
    
    // 启动浏览器
    browser, err := pw.Chromium.Launch()
    if err != nil {
        t.Fatalf("无法启动浏览器: %v", err)
    }
    defer browser.Close()
    
    // 创建页面
    page, err := browser.NewPage()
    if err != nil {
        t.Fatalf("无法创建页面: %v", err)
    }
    
    // 导航到应用程序
    if _, err = page.Goto("http://localhost:3000"); err != nil {
        t.Fatalf("导航失败: %v", err)
    }
    
    // 填写注册表单
    if err = page.Fill("#email", "test@example.com"); err != nil {
        t.Fatalf("填写邮箱失败: %v", err)
    }
    
    if err = page.Fill("#password", "securepassword"); err != nil {
        t.Fatalf("填写密码失败: %v", err)
    }
    
    if err = page.Fill("#confirm-password", "securepassword"); err != nil {
        t.Fatalf("确认密码失败: %v", err)
    }
    
    // 点击注册按钮
    if err = page.Click("#signup-button"); err != nil {
        t.Fatalf("点击注册按钮失败: %v", err)
    }
    
    // 等待重定向到仪表板
    if err = page.WaitForURL("**/dashboard"); err != nil {
        t.Fatalf("等待重定向失败: %v", err)
    }
    
    // 验证欢迎消息
    welcomeText, err := page.TextContent("#welcome-message")
    if err != nil {
        t.Fatalf("获取欢迎消息失败: %v", err)
    }
    
    if !strings.Contains(welcomeText, "test@example.com") {
        t.Errorf("欢迎消息应包含用户邮箱, 实际: %s", welcomeText)
    }
}
```

#### API E2E测试

对RESTful或GraphQL API进行端到端测试：

```go
func TestUserJourney(t *testing.T) {
    if testing.Short() {
        t.Skip("跳过E2E测试")
    }
    
    // 设置测试环境
    apiURL := "http://localhost:8080/api"
    if envURL := os.Getenv("API_URL"); envURL != "" {
        apiURL = envURL
    }
    
    client := &http.Client{Timeout: 10 * time.Second}
    
    // 第1步: 用户注册
    registerPayload := `{"email":"e2e@example.com","password":"secure123"}`
    registerResp, err := client.Post(
        apiURL+"/register",
        "application/json",
        strings.NewReader(registerPayload),
    )
    if err != nil {
        t.Fatalf("注册请求失败: %v", err)
    }
    defer registerResp.Body.Close()
    
    if registerResp.StatusCode != http.StatusCreated {
        t.Fatalf("注册应返回201状态码, 实际: %d", registerResp.StatusCode)
    }
    
    var registerResult struct {
        UserID string `json:"user_id"`
    }
    json.NewDecoder(registerResp.Body).Decode(&registerResult)
    
    // 第2步: 用户登录
    loginPayload := `{"email":"e2e@example.com","password":"secure123"}`
    loginResp, err := client.Post(
        apiURL+"/login",
        "application/json",
        strings.NewReader(loginPayload),
    )
    if err != nil {
        t.Fatalf("登录请求失败: %v", err)
    }
    defer loginResp.Body.Close()
    
    if loginResp.StatusCode != http.StatusOK {
        t.Fatalf("登录应返回200状态码, 实际: %d", loginResp.StatusCode)
    }
    
    var loginResult struct {
        Token string `json:"token"`
    }
    json.NewDecoder(loginResp.Body).Decode(&loginResult)
    
    // 第3步: 创建资源
    createResourceReq, _ := http.NewRequest(
        "POST",
        apiURL+"/resources",
        strings.NewReader(`{"name":"E2E Test Resource","type":"test"}`),
    )
    createResourceReq.Header.Set("Authorization", "Bearer "+loginResult.Token)
    createResourceReq.Header.Set("Content-Type", "application/json")
    
    createResourceResp, err := client.Do(createResourceReq)
    if err != nil {
        t.Fatalf("创建资源请求失败: %v", err)
    }
    defer createResourceResp.Body.Close()
    
    if createResourceResp.StatusCode != http.StatusCreated {
        t.Fatalf("创建资源应返回201状态码, 实际: %d", createResourceResp.StatusCode)
    }
    
    var resource struct {
        ID string `json:"id"`
    }
    json.NewDecoder(createResourceResp.Body).Decode(&resource)
    
    // 第4步: 获取资源
    getResourceReq, _ := http.NewRequest("GET", apiURL+"/resources/"+resource.ID, nil)
    getResourceReq.Header.Set("Authorization", "Bearer "+loginResult.Token)
    
    getResourceResp, err := client.Do(getResourceReq)
    if err != nil {
        t.Fatalf("获取资源请求失败: %v", err)
    }
    defer getResourceResp.Body.Close()
    
    if getResourceResp.StatusCode != http.StatusOK {
        t.Fatalf("获取资源应返回200状态码, 实际: %d", getResourceResp.StatusCode)
    }
    
    var getResourceResult struct {
        ID   string `json:"id"`
        Name string `json:"name"`
        Type string `json:"type"`
    }
    json.NewDecoder(getResourceResp.Body).Decode(&getResourceResult)
    
    if getResourceResult.Name != "E2E Test Resource" || getResourceResult.Type != "test" {
        t.Errorf("资源数据不匹配: %+v", getResourceResult)
    }
}
```

### 使用Docker Compose进行E2E测试

```go
package e2e_test

import (
    "context"
    "fmt"
    "net/http"
    "os"
    "os/exec"
    "testing"
    "time"
)

func TestMain(m *testing.M) {
    // 设置
    ctx := context.Background()
    
    fmt.Println("启动测试环境...")
    upCmd := exec.CommandContext(ctx, "docker-compose", "-f", "docker-compose.test.yml", "up", "-d")
    upCmd.Stdout = os.Stdout
    upCmd.Stderr = os.Stderr
    
    if err := upCmd.Run(); err != nil {
        fmt.Printf("启动Docker Compose失败: %v\n", err)
        os.Exit(1)
    }
    
    // 等待服务就绪
    waitForService("http://localhost:8080/health", 30*time.Second)
    
    // 运行测试
    exitCode := m.Run()
    
    // 清理
    fmt.Println("清理测试环境...")
    downCmd := exec.CommandContext(ctx, "docker-compose", "-f", "docker-compose.test.yml", "down")
    downCmd.Stdout = os.Stdout
    downCmd.Stderr = os.Stderr
    downCmd.Run()
    
    os.Exit(exitCode)
}

func waitForService(url string, timeout time.Duration) {
    deadline := time.Now().Add(timeout)
    for time.Now().Before(deadline) {
        resp, err := http.Get(url)
        if err == nil && resp.StatusCode == http.StatusOK {
            resp.Body.Close()
            return
        }
        if resp != nil {
            resp.Body.Close()
        }
        time.Sleep(1 * time.Second)
    }
    fmt.Printf("服务在 %s 未就绪\n", timeout)
    os.Exit(1)
}

func TestEndToEnd(t *testing.T) {
    // 此时所有服务已启动并就绪
    // 执行E2E测试...
}
```

## 集成测试与E2E测试的最佳实践

1. **避免过度测试**：只测试关键路径和边界情况，不要尝试测试所有可能的场景。

2. **隔离测试环境**：使用专用的测试数据库和服务实例，避免影响生产环境。

3. **使用合适的测试数据**：创建代表真实世界情况的测试数据。

4. **管理测试速度**：使用并行测试和高效的设置/拆卸来减少执行时间。

5. **自动化测试环境**：使用容器和配置脚本自动化测试环境设置。

6. **明确测试范围**：区分单元测试、集成测试和E2E测试的责任范围。

7. **稳定测试**：避免脆弱的测试，例如依赖于特定时间或外部服务的测试。

8. **清理资源**：测试完成后清理所有创建的资源，特别是在共享环境中。

9. **监控测试时间**：定期分析并优化长时间运行的测试。

10. **明确失败原因**：测试失败时提供清晰的错误消息和上下文。

11. **定期执行**：将E2E和集成测试纳入CI/CD流程，但可能不需要在每次提交时运行。

12. **维护分析**：分析测试失败模式，识别并解决系统中的弱点。

集成测试和E2E测试是测试金字塔的重要组成部分，它们与单元测试一起提供了全面的测试覆盖，确保应用程序在各个层面上都能正确运行。
