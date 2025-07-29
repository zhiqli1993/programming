# Go语言设计模式

## 📚 学习目标
掌握在Go语言中应用经典设计模式的方法和实践，理解Go语言特有的设计模式实现方式，以及如何利用Go语言的特性创建简洁、高效、可维护的代码结构。

## 📖 主要模式分类

Go语言设计模式可以分为三大类：

1. **[创建型模式](创建型模式.md)** - 处理对象创建机制，以适合情况的方式创建对象
2. **[结构型模式](结构型模式.md)** - 关注类和对象的组合，通过创建复杂结构提供更高层次的灵活性
3. **[行为型模式](行为型模式.md)** - 关注对象之间的通信、职责分配和算法封装

## 🎯 学习路径

### 第一阶段：创建型模式 (核心)
1. **单例模式** - 确保一个类只有一个实例
   - Go语言中的单例实现
   - sync.Once保证线程安全
   - 惰性初始化
   - 单例的测试方法

2. **工厂模式** - 对象创建的抽象
   - 简单工厂模式
   - 工厂方法模式
   - 抽象工厂模式
   - Go语言中的工厂函数

3. **建造者模式** - 分步骤构建复杂对象
   - 链式调用API设计
   - 选项模式(Functional Options)
   - 构建器与配置分离
   - 验证与默认值处理

4. **原型模式** - 对象拷贝与克隆
   - 深拷贝实现
   - 序列化与反序列化
   - 原型注册表
   - 原型继承

### 第二阶段：结构型模式 (重要)
5. **适配器模式** - 接口转换
   - 类适配器
   - 对象适配器
   - 双向适配器
   - 适配第三方库

6. **装饰器模式** - 动态添加职责
   - 中间件设计
   - HTTP处理器装饰
   - IO装饰器
   - 函数装饰器

7. **代理模式** - 控制对象访问
   - 远程代理
   - 虚拟代理
   - 保护代理
   - 缓存代理

8. **组合模式** - 部分-整体层次结构
   - 树形结构实现
   - 透明组合模式
   - 安全组合模式
   - 组合迭代器

9. **外观模式** - 统一接口
   - API封装
   - 子系统简化
   - 接口统一
   - 依赖隔离

### 第三阶段：行为型模式 (高级)
10. **观察者模式** - 一对多依赖关系
    - 事件系统
    - 发布-订阅机制
    - 异步观察者
    - 取消订阅机制

11. **策略模式** - 算法族与上下文分离
    - 策略接口
    - 上下文设计
    - 运行时策略选择
    - 函数式策略模式

12. **模板方法模式** - 算法骨架
    - 接口与默认实现
    - 钩子方法
    - 回调函数
    - 算法步骤固化

13. **责任链模式** - 处理请求链
    - HTTP中间件链
    - 管道模式实现
    - 动态链构建
    - 双向责任链

14. **状态模式** - 状态转换与行为
    - 状态机实现
    - 上下文与状态分离
    - 状态转换表
    - 事件驱动状态机

## 🎨 Go语言设计模式特色

### 语言特性影响
- **接口隐式实现** - 松耦合设计更自然
- **组合优于继承** - 更倾向于组合模式
- **函数是一等公民** - 函数式模式更简洁
- **并发原生支持** - 并发设计模式更丰富

### 常见应用场景
- **Web框架** - 大量使用中间件(装饰器)和上下文模式
- **配置管理** - 广泛应用建造者和单例模式
- **数据库访问** - 工厂模式和适配器模式常见
- **并发控制** - 特有的并发模式实现

## 🛠️ 实践项目

### 基础项目
1. **配置管理库** - 使用建造者模式和单例模式
2. **插件系统** - 使用工厂模式和策略模式
3. **日志库封装** - 使用装饰器模式和责任链模式
4. **缓存代理** - 使用代理模式和策略模式

### 进阶项目
1. **Web中间件框架** - 责任链模式与装饰器模式
2. **命令解析器** - 命令模式与组合模式
3. **工作流引擎** - 状态模式与观察者模式
4. **UI组件库** - 组合模式与访问者模式

## 📖 学习建议

### 学习顺序
1. **先理解原则** - 掌握SOLID等设计原则
2. **创建型模式** - 从对象创建模式开始
3. **结构型模式** - 学习对象组织方法
4. **行为型模式** - 深入对象交互模式

### 常见误区
- **过度设计** - 不是所有代码都需要模式
- **模式教条** - 灵活调整模式以适应Go特性
- **忽视简单性** - Go推崇简单直接的解决方案
- **忽略性能** - 某些模式可能引入性能开销

### 实践建议
- **阅读源码** - 学习标准库和知名项目的模式应用
- **重构代码** - 识别现有代码中的模式机会
- **写测试** - 通过测试验证模式的正确性
- **寻求反馈** - 代码审查中讨论设计选择

## 🎯 能力检查点

### 基础能力 (第1-2周)
- [ ] 理解并能实现基本创建型模式
- [ ] 能够识别代码中的设计模式
- [ ] 理解Go语言对经典设计模式的调整
- [ ] 能够选择合适的模式解决问题

### 进阶能力 (第3-4周)
- [ ] 熟练应用结构型模式组织代码
- [ ] 能够结合多种模式解决复杂问题
- [ ] 理解模式之间的关系和转换
- [ ] 能够评估模式应用的成本和收益

### 高级能力 (第5-8周)
- [ ] 精通行为型模式处理对象交互
- [ ] 能够创建自定义设计模式
- [ ] 理解并实现并发设计模式
- [ ] 能够在系统架构层面应用模式

## 🔧 实现示例

### 单例模式
```go
var (
    instance *Singleton
    once     sync.Once
)

func GetInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{}
    })
    return instance
}
```

### 策略模式
```go
type Strategy interface {
    Execute(data string) string
}

type Context struct {
    strategy Strategy
}

func (c *Context) SetStrategy(strategy Strategy) {
    c.strategy = strategy
}

func (c *Context) ExecuteStrategy(data string) string {
    return c.strategy.Execute(data)
}
```

### 装饰器模式
```go
type Component interface {
    Operation() string
}

type ConcreteComponent struct{}

func (c *ConcreteComponent) Operation() string {
    return "ConcreteComponent"
}

type Decorator struct {
    component Component
}

func (d *Decorator) Operation() string {
    return d.component.Operation() + " + Decorator"
}
```

## 🌟 Go特有模式

### 函数选项模式
```go
type Option func(*Config)

func WithTimeout(timeout time.Duration) Option {
    return func(c *Config) {
        c.Timeout = timeout
    }
}

func NewServer(addr string, options ...Option) *Server {
    config := defaultConfig()
    for _, option := range options {
        option(config)
    }
    return &Server{addr: addr, config: config}
}
```

### 上下文模式
```go
func ProcessRequest(ctx context.Context, req Request) (Response, error) {
    select {
    case <-ctx.Done():
        return Response{}, ctx.Err()
    case result := <-processAsync(req):
        return result, nil
    }
}
```

### 错误处理模式
```go
type Result struct {
    Value string
    Err   error
}

func operation() Result {
    // 执行操作并返回结果和错误
    return Result{Value: "data", Err: nil}
}

// 使用
result := operation()
if result.Err != nil {
    // 处理错误
}
```

## 🔗 学习资源

### 官方资源
- [Go标准库源码](https://golang.org/src/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go Proverbs](https://go-proverbs.github.io/)

### 推荐书籍
- 《Go设计模式》
- 《Head First设计模式》(需调整为Go实现)
- 《Clean Architecture》
- 《Go语言编程模式实践》

### 在线资源
- [Go Patterns](https://github.com/tmrts/go-patterns)
- [Go语言设计模式](https://github.com/senghoo/golang-design-pattern)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Go设计模式实例](https://refactoring.guru/design-patterns/go)

掌握Go语言设计模式，不仅能让您写出更优雅、可维护的代码，还能提升系统架构设计能力，是成为Go语言高级开发者的必备技能。
