# Mock与测试替身

在软件测试中，测试替身（Test Double）是一种模拟真实组件行为的对象，用于隔离被测试的代码单元。Go语言提供了多种创建和使用测试替身的方法，帮助开发者编写更可靠、更独立的测试。

## 测试替身的类型

测试替身有几种不同的类型，每种类型有不同的用途：

1. **Dummy（虚设对象）**：仅用于填充参数列表，实际上不会被使用。
2. **Stub（存根）**：提供预定义的响应，不处理未预料到的调用。
3. **Spy（间谍）**：记录调用信息，供测试断言使用。
4. **Mock（模拟对象）**：预设期望的调用及其响应，可验证是否按预期使用。
5. **Fake（伪造对象）**：提供简化的实现，不适合生产环境但满足测试需求。

## 使用接口进行依赖注入

Go的接口是实现测试替身的基础。通过定义接口并注入依赖，可以轻松替换真实实现：

```go
// 定义接口
type UserRepository interface {
    GetByID(id int) (User, error)
    Save(user User) error
}

// 依赖注入
type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) UpdateUser(id int, name string) error {
    user, err := s.repo.GetByID(id)
    if err != nil {
        return err
    }
    
    user.Name = name
    return s.repo.Save(user)
}
```

## 手动创建Mock

最简单的方法是手动创建满足接口的Mock实现：

```go
// 手动创建Mock
type MockUserRepository struct {
    // 存储预设的响应
    users map[int]User
    // 跟踪调用信息
    GetByIDCalled bool
    GetByIDInput  int
    SaveCalled    bool
    SaveInput     User
}

// 构造函数
func NewMockUserRepository() *MockUserRepository {
    return &MockUserRepository{
        users: make(map[int]User),
    }
}

// 实现接口方法
func (m *MockUserRepository) GetByID(id int) (User, error) {
    m.GetByIDCalled = true
    m.GetByIDInput = id
    
    user, exists := m.users[id]
    if !exists {
        return User{}, errors.New("user not found")
    }
    return user, nil
}

func (m *MockUserRepository) Save(user User) error {
    m.SaveCalled = true
    m.SaveInput = user
    
    m.users[user.ID] = user
    return nil
}

// 添加用于测试的辅助方法
func (m *MockUserRepository) SetUser(user User) {
    m.users[user.ID] = user
}
```

使用手动创建的Mock：

```go
func TestUserService_UpdateUser(t *testing.T) {
    // 设置Mock
    mockRepo := NewMockUserRepository()
    mockRepo.SetUser(User{ID: 1, Name: "Old Name"})
    
    // 创建被测试的服务
    service := NewUserService(mockRepo)
    
    // 执行测试
    err := service.UpdateUser(1, "New Name")
    
    // 验证结果
    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
    
    // 验证Mock被正确调用
    if !mockRepo.GetByIDCalled {
        t.Error("Expected GetByID to be called")
    }
    
    if mockRepo.GetByIDInput != 1 {
        t.Errorf("Expected GetByID to be called with ID 1, got %d", mockRepo.GetByIDInput)
    }
    
    if !mockRepo.SaveCalled {
        t.Error("Expected Save to be called")
    }
    
    if mockRepo.SaveInput.Name != "New Name" {
        t.Errorf("Expected Save to be called with user name 'New Name', got %s", mockRepo.SaveInput.Name)
    }
}
```

## 使用测试替身框架

对于复杂场景，可以使用测试替身框架简化工作：

### testify/mock

[testify/mock](https://github.com/stretchr/testify) 是最流行的Go测试替身框架之一：

```go
import (
    "testing"
    
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/assert"
)

// 创建模拟对象
type MockUserRepo struct {
    mock.Mock
}

// 实现接口方法
func (m *MockUserRepo) GetByID(id int) (User, error) {
    // 记录方法调用
    args := m.Called(id)
    // 返回预设的值
    return args.Get(0).(User), args.Error(1)
}

func (m *MockUserRepo) Save(user User) error {
    args := m.Called(user)
    return args.Error(0)
}

func TestUserService_UpdateUser_WithTestify(t *testing.T) {
    // 创建Mock
    mockRepo := new(MockUserRepo)
    
    // 设置期望
    user := User{ID: 1, Name: "Old Name"}
    mockRepo.On("GetByID", 1).Return(user, nil)
    mockRepo.On("Save", mock.Anything).Return(nil)
    
    // 创建服务并测试
    service := NewUserService(mockRepo)
    err := service.UpdateUser(1, "New Name")
    
    // 断言结果
    assert.NoError(t, err)
    
    // 验证期望
    mockRepo.AssertExpectations(t)
    
    // 验证Save方法调用参数
    saveCall := mockRepo.Calls[1]
    savedUser := saveCall.Arguments[0].(User)
    assert.Equal(t, "New Name", savedUser.Name)
}
```

### gomock

[gomock](https://github.com/golang/mock) 是Google官方的Mock生成器：

```bash
# 安装gomock
go install go.uber.org/mock/mockgen@latest

# 为接口生成Mock
mockgen -source=user_repository.go -destination=mocks/mock_user_repository.go -package=mocks
```

生成的Mock可以这样使用：

```go
import (
    "testing"
    
    "go.uber.org/mock/gomock"
    "example.com/myapp/mocks"
)

func TestUserService_UpdateUser_WithGoMock(t *testing.T) {
    // 创建控制器
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    // 创建Mock
    mockRepo := mocks.NewMockUserRepository(ctrl)
    
    // 设置期望
    user := User{ID: 1, Name: "Old Name"}
    updatedUser := User{ID: 1, Name: "New Name"}
    
    // 期望GetByID被调用，并返回user和nil
    mockRepo.EXPECT().GetByID(1).Return(user, nil)
    
    // 期望Save被调用，参数匹配updatedUser，并返回nil
    mockRepo.EXPECT().Save(gomock.Any()).
        DoAndReturn(func(u User) error {
            if u.ID != 1 || u.Name != "New Name" {
                t.Errorf("Save called with unexpected user: %+v", u)
            }
            return nil
        })
    
    // 创建服务并测试
    service := NewUserService(mockRepo)
    err := service.UpdateUser(1, "New Name")
    
    // 断言结果
    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
}
```

## 创建简单的Fake对象

有时，使用简单的实现（Fake）比复杂的Mock更适合：

```go
// 内存中的用户仓库实现
type InMemoryUserRepository struct {
    users map[int]User
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{
        users: make(map[int]User),
    }
}

func (r *InMemoryUserRepository) GetByID(id int) (User, error) {
    user, exists := r.users[id]
    if !exists {
        return User{}, errors.New("user not found")
    }
    return user, nil
}

func (r *InMemoryUserRepository) Save(user User) error {
    r.users[user.ID] = user
    return nil
}

// 测试用例
func TestUserService_WithFake(t *testing.T) {
    // 创建Fake仓库
    fakeRepo := NewInMemoryUserRepository()
    fakeRepo.users[1] = User{ID: 1, Name: "Old Name"}
    
    // 创建服务
    service := NewUserService(fakeRepo)
    
    // 执行操作
    err := service.UpdateUser(1, "New Name")
    
    // 验证结果
    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
    
    // 验证用户已更新
    updatedUser := fakeRepo.users[1]
    if updatedUser.Name != "New Name" {
        t.Errorf("Expected user name to be 'New Name', got %s", updatedUser.Name)
    }
}
```

## HTTP服务测试

测试HTTP服务时，可以使用`httptest`包创建测试服务器：

```go
import (
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
)

func TestUserHandler(t *testing.T) {
    // 创建Mock仓库
    mockRepo := NewMockUserRepository()
    mockRepo.SetUser(User{ID: 1, Name: "Test User"})
    
    // 创建HTTP处理函数
    handler := NewUserHandler(mockRepo)
    
    // 创建测试请求
    requestBody := `{"name":"Updated User"}`
    req := httptest.NewRequest("PUT", "/users/1", strings.NewReader(requestBody))
    req.Header.Set("Content-Type", "application/json")
    
    // 创建响应记录器
    rec := httptest.NewRecorder()
    
    // 处理请求
    handler.ServeHTTP(rec, req)
    
    // 验证状态码
    if rec.Code != http.StatusOK {
        t.Errorf("Expected status code %d, got %d", http.StatusOK, rec.Code)
    }
    
    // 验证响应内容
    var response map[string]interface{}
    json.Unmarshal(rec.Body.Bytes(), &response)
    
    if response["success"] != true {
        t.Errorf("Expected success to be true")
    }
}
```

## 使用依赖注入容器

对于大型应用，可以使用依赖注入容器管理测试替身：

```go
import "github.com/google/wire"

// 定义Provider
func ProvideUserRepository() UserRepository {
    return NewDatabaseUserRepository()
}

func ProvideUserService(repo UserRepository) *UserService {
    return NewUserService(repo)
}

// 正常构建
var normalSet = wire.NewSet(
    ProvideUserRepository,
    ProvideUserService,
)

// 测试构建
func ProvideMockUserRepository() UserRepository {
    return NewMockUserRepository()
}

var testSet = wire.NewSet(
    ProvideMockUserRepository,
    ProvideUserService,
)
```

## 模拟时间

测试与时间相关的代码时，可以使用时间模拟技术：

```go
// 时间提供者接口
type TimeProvider interface {
    Now() time.Time
}

// 真实实现
type RealTimeProvider struct{}

func (p RealTimeProvider) Now() time.Time {
    return time.Now()
}

// 模拟实现
type MockTimeProvider struct {
    CurrentTime time.Time
}

func (p MockTimeProvider) Now() time.Time {
    return p.CurrentTime
}

// 使用时间提供者的服务
type ExpiryChecker struct {
    timeProvider TimeProvider
}

func NewExpiryChecker(provider TimeProvider) *ExpiryChecker {
    return &ExpiryChecker{timeProvider: provider}
}

func (c *ExpiryChecker) IsExpired(expiryDate time.Time) bool {
    return c.timeProvider.Now().After(expiryDate)
}

// 测试
func TestExpiryChecker(t *testing.T) {
    // 创建模拟时间
    mockTime := MockTimeProvider{
        CurrentTime: time.Date(2023, 5, 15, 0, 0, 0, 0, time.UTC),
    }
    
    checker := NewExpiryChecker(mockTime)
    
    // 测试未过期的情况
    future := time.Date(2023, 5, 16, 0, 0, 0, 0, time.UTC)
    if checker.IsExpired(future) {
        t.Error("Expected future date to not be expired")
    }
    
    // 测试已过期的情况
    past := time.Date(2023, 5, 14, 0, 0, 0, 0, time.UTC)
    if !checker.IsExpired(past) {
        t.Error("Expected past date to be expired")
    }
}
```

## 模拟文件系统

测试文件操作时，可以使用文件系统接口：

```go
// 文件系统接口
type FileSystem interface {
    ReadFile(filename string) ([]byte, error)
    WriteFile(filename string, data []byte, perm os.FileMode) error
    Stat(name string) (os.FileInfo, error)
}

// 真实实现
type RealFileSystem struct{}

func (fs RealFileSystem) ReadFile(filename string) ([]byte, error) {
    return os.ReadFile(filename)
}

func (fs RealFileSystem) WriteFile(filename string, data []byte, perm os.FileMode) error {
    return os.WriteFile(filename, data, perm)
}

func (fs RealFileSystem) Stat(name string) (os.FileInfo, error) {
    return os.Stat(name)
}

// 模拟实现
type MockFileSystem struct {
    files map[string][]byte
}

func NewMockFileSystem() *MockFileSystem {
    return &MockFileSystem{
        files: make(map[string][]byte),
    }
}

func (fs *MockFileSystem) ReadFile(filename string) ([]byte, error) {
    data, exists := fs.files[filename]
    if !exists {
        return nil, os.ErrNotExist
    }
    return data, nil
}

func (fs *MockFileSystem) WriteFile(filename string, data []byte, perm os.FileMode) error {
    fs.files[filename] = data
    return nil
}

func (fs *MockFileSystem) Stat(name string) (os.FileInfo, error) {
    if _, exists := fs.files[name]; !exists {
        return nil, os.ErrNotExist
    }
    return nil, nil // 简化实现
}

// 使用文件系统的服务
type ConfigService struct {
    fs FileSystem
}

func NewConfigService(fs FileSystem) *ConfigService {
    return &ConfigService{fs: fs}
}

func (s *ConfigService) LoadConfig(filename string) (Config, error) {
    data, err := s.fs.ReadFile(filename)
    if err != nil {
        return Config{}, err
    }
    
    var config Config
    err = json.Unmarshal(data, &config)
    return config, err
}

// 测试
func TestConfigService_LoadConfig(t *testing.T) {
    mockFS := NewMockFileSystem()
    
    // 设置模拟文件
    configData := []byte(`{"appName": "TestApp", "maxUsers": 100}`)
    mockFS.WriteFile("config.json", configData, 0644)
    
    service := NewConfigService(mockFS)
    
    // 测试加载配置
    config, err := service.LoadConfig("config.json")
    
    // 验证结果
    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
    
    if config.AppName != "TestApp" {
        t.Errorf("Expected AppName to be 'TestApp', got %s", config.AppName)
    }
    
    if config.MaxUsers != 100 {
        t.Errorf("Expected MaxUsers to be 100, got %d", config.MaxUsers)
    }
}
```

## 测试替身的最佳实践

1. **接口隔离原则**：创建小型、专注的接口，便于模拟。
2. **依赖注入**：通过构造函数或字段注入依赖，而不是直接创建。
3. **只模拟直接依赖**：避免过度模拟，只模拟被测试代码直接依赖的组件。
4. **避免模拟实现细节**：测试应关注行为，而非实现。
5. **保持Mock简单**：复杂的Mock难以维护，考虑使用Fake或集成测试。
6. **使用工具生成Mock**：对于复杂接口，使用gomock等工具生成Mock。
7. **模拟外部服务**：对于数据库、API等外部服务，总是使用测试替身。
8. **考虑使用Fake而非Mock**：对于复杂行为，Fake可能比Mock更简单可靠。
9. **验证交互**：确保Mock被正确调用，参数正确。
10. **避免过度断言**：只验证重要的交互，过多的断言使测试脆弱。

通过合理使用测试替身，可以编写更加隔离、稳定和高效的测试，同时避免对外部依赖的耦合。
