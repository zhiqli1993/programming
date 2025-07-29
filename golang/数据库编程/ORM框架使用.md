# Go语言ORM框架使用

## 概述

对象关系映射(ORM)是一种编程技术，将关系型数据库中的数据表映射到面向对象编程语言中的对象。Go语言有多种ORM框架，能够简化数据库操作，提高开发效率。本文档将介绍几种常用的Go ORM框架的使用方法和最佳实践。

## GORM

GORM是Go语言中最流行的ORM框架之一，提供了丰富的功能和友好的API。

### 安装与设置

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql    # MySQL驱动
go get -u gorm.io/driver/postgres # PostgreSQL驱动
go get -u gorm.io/driver/sqlite   # SQLite驱动
```

### 建立连接

```go
package main

import (
	"log"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

func main() {
	// 连接MySQL
	dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info), // 设置日志级别
	})
	if err != nil {
		log.Fatalf("无法连接数据库: %v", err)
	}

	// 获取底层SQL连接以设置连接池
	sqlDB, err := db.DB()
	if err != nil {
		log.Fatalf("获取数据库连接失败: %v", err)
	}

	// 设置连接池参数
	sqlDB.SetMaxIdleConns(10)
	sqlDB.SetMaxOpenConns(100)
}
```

### 模型定义

GORM使用结构体标签来映射Go结构体与数据库表。

```go
type User struct {
	ID        uint           `gorm:"primaryKey"`
	Name      string         `gorm:"size:100;not null"`
	Email     string         `gorm:"uniqueIndex;size:100"`
	Age       int            `gorm:"default:18"`
	CreatedAt time.Time      `gorm:"autoCreateTime"`
	UpdatedAt time.Time      `gorm:"autoUpdateTime"`
	DeletedAt gorm.DeletedAt `gorm:"index"` // 软删除支持
}

// 自定义表名
func (User) TableName() string {
	return "users"
}
```

### 自动迁移

GORM可以自动创建/更新数据库表结构。

```go
// 自动迁移表结构
err := db.AutoMigrate(&User{}, &Product{}, &Order{})
if err != nil {
	log.Fatalf("自动迁移失败: %v", err)
}
```

### 基本CRUD操作

#### 创建记录

```go
// 创建单条记录
user := User{Name: "张三", Email: "zhangsan@example.com", Age: 25}
result := db.Create(&user)
if result.Error != nil {
	log.Fatalf("创建用户失败: %v", result.Error)
}
log.Printf("创建用户成功，ID: %d, 影响行数: %d", user.ID, result.RowsAffected)

// 批量创建
users := []User{
	{Name: "李四", Email: "lisi@example.com", Age: 30},
	{Name: "王五", Email: "wangwu@example.com", Age: 28},
}
result = db.Create(&users)
log.Printf("批量创建成功，影响行数: %d", result.RowsAffected)
```

#### 查询记录

```go
// 查询单条记录
var user User
result := db.First(&user, 1) // 查询ID为1的用户
if result.Error != nil {
	log.Fatalf("查询用户失败: %v", result.Error)
}
log.Printf("查询结果: %v", user)

// 条件查询
var anotherUser User
result = db.Where("name = ?", "张三").First(&anotherUser)
if result.Error != nil {
	log.Fatalf("条件查询失败: %v", result.Error)
}

// 查询多条记录
var users []User
result = db.Where("age > ?", 20).Limit(10).Find(&users)
log.Printf("查询到%d条记录", len(users))

// 使用结构体查询
result = db.Where(&User{Name: "张三"}).Find(&users)

// 使用map查询
result = db.Where(map[string]interface{}{"name": "张三", "age": 25}).Find(&users)

// 使用IN查询
result = db.Where("name IN ?", []string{"张三", "李四"}).Find(&users)

// 选择特定字段
result = db.Select("name", "email").Find(&users)

// 排序
result = db.Order("age desc, name").Find(&users)

// 分页
result = db.Offset(10).Limit(5).Find(&users)

// 分组与聚合
type Result struct {
	Age   int
	Count int
}
var results []Result
db.Model(&User{}).Select("age, count(*) as count").Group("age").Find(&results)
```

#### 更新记录

```go
// 更新单个字段
result := db.Model(&User{}).Where("id = ?", 1).Update("name", "新名字")

// 更新多个字段
result = db.Model(&User{}).Where("id = ?", 1).Updates(
	map[string]interface{}{
		"name": "新名字",
		"age":  26,
	},
)

// 使用结构体更新（注意：仅更新非零值字段）
user := User{
	Name: "新名字",
	Age:  26,
}
result = db.Model(&User{}).Where("id = ?", 1).Updates(user)

// 使用Select更新零值字段
result = db.Model(&User{}).Select("name", "age").Where("id = ?", 1).Updates(user)

// 批量更新
result = db.Model(&User{}).Where("age < ?", 20).Update("age", gorm.Expr("age + ?", 1))
```

#### 删除记录

```go
// 删除单条记录
result := db.Delete(&User{}, 1) // 删除ID为1的用户

// 条件删除
result = db.Where("age < ?", 18).Delete(&User{})

// 软删除 (需要结构体有DeletedAt gorm.DeletedAt字段)
// 以上删除操作默认为软删除，会更新DeletedAt字段

// 永久删除
result = db.Unscoped().Delete(&User{}, 1)
```

### 关联关系

GORM支持多种关联关系：一对一、一对多、多对多。

#### 一对一关系

```go
type User struct {
	gorm.Model
	Name    string
	Profile Profile // has one
}

type Profile struct {
	gorm.Model
	UserID   uint
	Address  string
	Phone    string
}

// 创建关联
user := User{
	Name: "张三",
	Profile: Profile{
		Address: "北京市",
		Phone:   "12345678901",
	},
}
db.Create(&user) // 会自动创建Profile并设置关联

// 预加载关联
var userWithProfile User
db.Preload("Profile").First(&userWithProfile, 1)
```

#### 一对多关系

```go
type User struct {
	gorm.Model
	Name    string
	Posts   []Post // has many
}

type Post struct {
	gorm.Model
	Title   string
	Content string
	UserID  uint
}

// 创建关联
user := User{
	Name: "张三",
	Posts: []Post{
		{Title: "第一篇文章", Content: "内容..."},
		{Title: "第二篇文章", Content: "内容..."},
	},
}
db.Create(&user)

// 预加载关联
var userWithPosts User
db.Preload("Posts").First(&userWithPosts, 1)

// 关联查询
var posts []Post
db.Model(&User{ID: 1}).Association("Posts").Find(&posts)
```

#### 多对多关系

```go
type User struct {
	gorm.Model
	Name   string
	Roles  []Role `gorm:"many2many:user_roles;"`
}

type Role struct {
	gorm.Model
	Name  string
	Users []User `gorm:"many2many:user_roles;"`
}

// 创建关联
user := User{
	Name: "张三",
	Roles: []Role{
		{Name: "管理员"},
		{Name: "编辑"},
	},
}
db.Create(&user)

// 预加载关联
var userWithRoles User
db.Preload("Roles").First(&userWithRoles, 1)

// 添加关联
var user User
var role Role
db.First(&user, 1)
db.First(&role, 3) // 假设有ID为3的角色
db.Model(&user).Association("Roles").Append(&role)

// 删除关联
db.Model(&user).Association("Roles").Delete(&role)

// 清空关联
db.Model(&user).Association("Roles").Clear()

// 关联计数
count := db.Model(&user).Association("Roles").Count()
```

### 事务处理

```go
// 方法1：手动事务
tx := db.Begin()
// 注意：使用tx而不是db来执行操作
user := User{Name: "事务用户"}
if err := tx.Create(&user).Error; err != nil {
	tx.Rollback()
	log.Fatalf("创建用户失败: %v", err)
}

if err := tx.Create(&Profile{UserID: user.ID, Address: "事务地址"}).Error; err != nil {
	tx.Rollback()
	log.Fatalf("创建资料失败: %v", err)
}

tx.Commit()

// 方法2：事务闭包
err := db.Transaction(func(tx *gorm.DB) error {
	user := User{Name: "事务用户2"}
	if err := tx.Create(&user).Error; err != nil {
		return err
	}

	if err := tx.Create(&Profile{UserID: user.ID, Address: "事务地址2"}).Error; err != nil {
		return err
	}

	return nil
})

if err != nil {
	log.Fatalf("事务执行失败: %v", err)
}
```

### 钩子方法

GORM支持模型生命周期的钩子方法。

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
	// 创建前执行
	u.Name = strings.TrimSpace(u.Name)
	if u.Name == "" {
		err = errors.New("用户名不能为空")
	}
	return
}

func (u *User) AfterCreate(tx *gorm.DB) (err error) {
	// 创建后执行
	return tx.Create(&Log{Action: "创建用户", UserID: u.ID}).Error
}

// 其他钩子: BeforeSave, AfterSave, BeforeUpdate, AfterUpdate, BeforeDelete, AfterDelete, AfterFind
```

### 原生SQL

```go
// 执行原生SQL
var users []User
db.Raw("SELECT * FROM users WHERE age > ?", 20).Scan(&users)

// 执行原生SQL而不绑定到模型
db.Exec("UPDATE users SET name = ? WHERE id = ?", "新名字", 1)

// 使用SQL生成器
db.Table("users").Select("AVG(age) as avg_age").Row().Scan(&avgAge)

// 命名参数
db.Where("name = @name AND age > @age", map[string]interface{}{
	"name": "张三",
	"age":  20,
}).Find(&users)
```

### 性能优化

```go
// 禁用默认事务
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	SkipDefaultTransaction: true,
})

// 预编译模式
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	PrepareStmt: true,
})

// 批量插入
var users = make([]User, 1000)
// 填充users数据...
// 设置批量插入大小
db.CreateInBatches(users, 100) // 每次插入100条

// 选择指定字段以减少数据传输
db.Select("name", "age").Find(&users)

// 使用索引
db.Clauses(hints.UseIndex("idx_user_name")).Find(&users)

// 查询缓存
// GORM不直接支持缓存，需要自行实现或使用第三方缓存库
```

## SQLx

SQLx是database/sql包的一个扩展，提供了更便捷的API，同时保持轻量级。

### 安装与设置

```bash
go get -u github.com/jmoiron/sqlx
```

### 基本使用

```go
package main

import (
	"log"

	"github.com/jmoiron/sqlx"
	_ "github.com/go-sql-driver/mysql"
)

// 定义结构体
type User struct {
	ID    int    `db:"id"`
	Name  string `db:"name"`
	Email string `db:"email"`
	Age   int    `db:"age"`
}

func main() {
	// 连接数据库
	db, err := sqlx.Connect("mysql", "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		log.Fatalf("无法连接数据库: %v", err)
	}
	defer db.Close()

	// 设置连接池
	db.SetMaxOpenConns(100)
	db.SetMaxIdleConns(10)

	// 查询单行数据
	user := User{}
	err = db.Get(&user, "SELECT * FROM users WHERE id = ?", 1)
	if err != nil {
		log.Fatalf("查询失败: %v", err)
	}
	log.Printf("用户: %+v", user)

	// 查询多行数据
	users := []User{}
	err = db.Select(&users, "SELECT * FROM users WHERE age > ?", 20)
	if err != nil {
		log.Fatalf("查询失败: %v", err)
	}
	log.Printf("查询到%d个用户", len(users))

	// 命名参数
	namedUsers := []User{}
	err = db.Select(&namedUsers, "SELECT * FROM users WHERE age > :minAge", map[string]interface{}{
		"minAge": 20,
	})
	if err != nil {
		log.Fatalf("命名参数查询失败: %v", err)
	}

	// 插入数据
	result, err := db.Exec("INSERT INTO users (name, email, age) VALUES (?, ?, ?)", "张三", "zhangsan@example.com", 25)
	if err != nil {
		log.Fatalf("插入失败: %v", err)
	}
	id, _ := result.LastInsertId()
	log.Printf("插入成功，ID: %d", id)

	// 命名参数插入
	_, err = db.NamedExec("INSERT INTO users (name, email, age) VALUES (:name, :email, :age)",
		map[string]interface{}{
			"name":  "李四",
			"email": "lisi@example.com",
			"age":   30,
		})
	if err != nil {
		log.Fatalf("命名参数插入失败: %v", err)
	}

	// 事务
	tx, err := db.Beginx()
	if err != nil {
		log.Fatalf("开始事务失败: %v", err)
	}
	
	_, err = tx.Exec("INSERT INTO users (name, email, age) VALUES (?, ?, ?)", "事务用户", "tx@example.com", 28)
	if err != nil {
		tx.Rollback()
		log.Fatalf("事务内插入失败: %v", err)
	}
	
	err = tx.Commit()
	if err != nil {
		log.Fatalf("提交事务失败: %v", err)
	}

	// 预处理语句
	stmt, err := db.Preparex("SELECT * FROM users WHERE id = ?")
	if err != nil {
		log.Fatalf("预处理失败: %v", err)
	}
	defer stmt.Close()
	
	var preparedUser User
	err = stmt.Get(&preparedUser, 1)
	if err != nil {
		log.Fatalf("预处理查询失败: %v", err)
	}
}
```

### 批量操作

```go
// 批量插入
users := []User{
	{Name: "批量1", Email: "batch1@example.com", Age: 25},
	{Name: "批量2", Email: "batch2@example.com", Age: 26},
	{Name: "批量3", Email: "batch3@example.com", Age: 27},
}

query := "INSERT INTO users (name, email, age) VALUES (:name, :email, :age)"

// 方法1：单独执行命名查询
for _, user := range users {
	_, err := db.NamedExec(query, user)
	if err != nil {
		log.Printf("插入失败: %v", err)
	}
}

// 方法2：事务中批量执行
tx, _ := db.Beginx()
for _, user := range users {
	_, err := tx.NamedExec(query, user)
	if err != nil {
		tx.Rollback()
		log.Fatalf("事务插入失败: %v", err)
	}
}
tx.Commit()
```

## XORM

XORM是另一个流行的Go ORM框架，提供丰富的查询API和详细的日志支持。

### 安装与设置

```bash
go get -u xorm.io/xorm
```

### 基本使用

```go
package main

import (
	"log"
	"time"

	_ "github.com/go-sql-driver/mysql"
	"xorm.io/xorm"
	"xorm.io/xorm/names"
)

// 定义模型
type User struct {
	Id        int64     `xorm:"pk autoincr"`
	Name      string    `xorm:"varchar(100) notnull"`
	Email     string    `xorm:"varchar(100) unique"`
	Age       int       `xorm:"default 18"`
	CreatedAt time.Time `xorm:"created"`
	UpdatedAt time.Time `xorm:"updated"`
	DeletedAt time.Time `xorm:"deleted"`
}

func main() {
	// 创建引擎
	engine, err := xorm.NewEngine("mysql", "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		log.Fatalf("创建引擎失败: %v", err)
	}
	defer engine.Close()

	// 配置
	engine.SetMaxOpenConns(100)
	engine.SetMaxIdleConns(10)
	engine.SetMapper(names.GonicMapper{}) // 名称映射策略
	engine.ShowSQL(true)                 // 显示SQL语句

	// 同步结构
	err = engine.Sync2(new(User))
	if err != nil {
		log.Fatalf("同步结构失败: %v", err)
	}

	// 插入数据
	user := &User{
		Name:  "张三",
		Email: "zhangsan@example.com",
		Age:   25,
	}
	affected, err := engine.Insert(user)
	if err != nil {
		log.Fatalf("插入失败: %v", err)
	}
	log.Printf("插入成功，影响行数: %d, ID: %d", affected, user.Id)

	// 批量插入
	users := []User{
		{Name: "李四", Email: "lisi@example.com", Age: 30},
		{Name: "王五", Email: "wangwu@example.com", Age: 28},
	}
	affected, err = engine.Insert(&users)
	if err != nil {
		log.Fatalf("批量插入失败: %v", err)
	}
	log.Printf("批量插入成功，影响行数: %d", affected)

	// 查询单条记录
	user = &User{}
	has, err := engine.ID(1).Get(user)
	if err != nil {
		log.Fatalf("查询失败: %v", err)
	}
	if has {
		log.Printf("查询结果: %+v", user)
	} else {
		log.Println("未找到记录")
	}

	// 条件查询
	user = &User{}
	has, err = engine.Where("name = ?", "张三").Get(user)
	if err != nil {
		log.Fatalf("条件查询失败: %v", err)
	}

	// 查询多条记录
	var users2 []User
	err = engine.Where("age > ?", 20).Limit(10, 0).Find(&users2)
	if err != nil {
		log.Fatalf("查询多条记录失败: %v", err)
	}
	log.Printf("查询到%d条记录", len(users2))

	// 更新记录
	user.Name = "新名字"
	affected, err = engine.ID(user.Id).Update(user)
	if err != nil {
		log.Fatalf("更新失败: %v", err)
	}
	log.Printf("更新成功，影响行数: %d", affected)

	// 更新指定字段
	affected, err = engine.ID(1).Cols("name").Update(&User{Name: "新名字2"})
	if err != nil {
		log.Fatalf("更新字段失败: %v", err)
	}

	// 删除记录
	affected, err = engine.ID(2).Delete(&User{})
	if err != nil {
		log.Fatalf("删除失败: %v", err)
	}
	log.Printf("删除成功，影响行数: %d", affected)

	// 事务
	session := engine.NewSession()
	defer session.Close()

	err = session.Begin()
	if err != nil {
		log.Fatalf("开始事务失败: %v", err)
	}

	user1 := User{Name: "事务用户1", Email: "tx1@example.com", Age: 31}
	user2 := User{Name: "事务用户2", Email: "tx2@example.com", Age: 32}

	if _, err = session.Insert(&user1); err != nil {
		session.Rollback()
		log.Fatalf("事务插入失败: %v", err)
	}

	if _, err = session.Insert(&user2); err != nil {
		session.Rollback()
		log.Fatalf("事务插入失败: %v", err)
	}

	err = session.Commit()
	if err != nil {
		log.Fatalf("提交事务失败: %v", err)
	}
}
```

## ORM比较与选择

### 功能对比

| 特性 | GORM | SQLx | XORM |
|------|------|------|------|
| 类型 | 全功能ORM | SQL辅助工具 | 全功能ORM |
| 自动迁移 | ✓ | ✗ | ✓ |
| 关联关系 | ✓ | ✗ | 部分支持 |
| 钩子方法 | ✓ | ✗ | ✓ |
| 软删除 | ✓ | ✗ | ✓ |
| 事务支持 | ✓ | ✓ | ✓ |
| 预加载 | ✓ | ✗ | ✓ |
| 复杂查询 | ✓ | ✓ | ✓ |
| 命名参数 | ✓ | ✓ | ✓ |
| 性能 | 中等 | 高 | 中等 |
| 易用性 | 高 | 中等 | 中等 |
| 文档质量 | 高 | 中等 | 中等 |
| 社区活跃度 | 高 | 中等 | 中等 |

### 框架选择建议

1. **选择GORM**：
   - 当需要完整的ORM功能
   - 需要自动迁移和复杂关联关系
   - 优先考虑开发效率
   - 需要充分的文档和社区支持

2. **选择SQLx**：
   - 当需要更接近原生SQL的体验
   - 对性能要求较高
   - 不需要ORM的自动迁移和关联功能
   - 希望学习曲线较平缓

3. **选择XORM**：
   - 当GORM不满足需求
   - 需要更灵活的查询API
   - 需要详细的SQL日志支持
   - 已有XORM使用经验

## ORM最佳实践

### 1. 合理使用事务

```go
// GORM事务最佳实践
err := db.Transaction(func(tx *gorm.DB) error {
	// 在事务中执行多个操作
	if err := tx.Create(&user).Error; err != nil {
		// 返回任何错误都会回滚事务
		return err
	}
	
	if err := tx.Create(&order).Error; err != nil {
		return err
	}
	
	// 返回nil提交事务
	return nil
})
```

### 2. 避免N+1查询问题

```go
// 错误做法 - 导致N+1查询
var users []User
db.Find(&users)
for _, user := range users {
	var posts []Post
	db.Where("user_id = ?", user.ID).Find(&posts)
	// 处理每个用户的文章...
}

// 正确做法 - 使用预加载
var users []User
db.Preload("Posts").Find(&users)
for _, user := range users {
	// 直接访问user.Posts，不会产生额外查询
}

// 或者使用连接查询
type UserWithPostCount struct {
	User
	PostCount int
}

var usersWithCount []UserWithPostCount
db.Model(&User{}).
	Select("users.*, COUNT(posts.id) as post_count").
	Joins("LEFT JOIN posts ON posts.user_id = users.id").
	Group("users.id").
	Find(&usersWithCount)
```

### 3. 批量操作优化

```go
// GORM批量插入优化
var users []User
// 填充大量用户数据...
// 设置合理的批量大小，避免单次操作数据过多
db.CreateInBatches(users, 100) // 每批100条
```

### 4. 合理使用索引

```go
// 在模型中定义索引
type Product struct {
	gorm.Model
	Code  string `gorm:"index:idx_code,unique"`
	Price uint   `gorm:"index:idx_price"`
}

// 查询时使用索引提示
db.Clauses(hints.UseIndex("idx_code")).Where("code = ?", "abc").Find(&products)
```

### 5. 使用连接池

```go
// GORM连接池
sqlDB, err := db.DB()
// 设置连接池上限
sqlDB.SetMaxOpenConns(100)
// 设置空闲连接数
sqlDB.SetMaxIdleConns(20)
// 设置连接最大存活时间
sqlDB.SetConnMaxLifetime(time.Hour)
```

### 6. 懒加载与预加载

```go
// 预加载 - 适合已知需要的关联数据
db.Preload("Orders").Preload("Profile").Find(&users)

// 懒加载 - 按需加载关联数据
// 注意：可能导致N+1问题，谨慎使用
db.First(&user, 1)
db.Model(&user).Association("Orders").Find(&orders)
```

### 7. 错误处理

```go
// 检查记录是否存在
result := db.First(&user, 1)
if result.Error != nil {
	if errors.Is(result.Error, gorm.ErrRecordNotFound) {
		// 处理记录不存在的情况
	} else {
		// 处理其他错误
	}
}

// 错误包装
if err := db.First(&user, 1).Error; err != nil {
	return fmt.Errorf("查询用户失败: %w", err)
}
```

### 8. 查询优化

```go
// 只查询需要的字段
db.Select("id", "name", "age").Find(&users)

// 避免使用Find后立即Count，应分别查询
var count int64
db.Model(&User{}).Where("age > ?", 20).Count(&count)
db.Where("age > ?", 20).Limit(10).Find(&users)
```

### 9. 模型设计

```go
// 使用组合而非继承
type Model struct {
	ID        uint           `gorm:"primarykey"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt gorm.DeletedAt `gorm:"index"`
}

type User struct {
	Model
	Name  string
	Email string
}

// 实现接口
type Taggable interface {
	SetTags(tags []string)
	GetTags() []string
}

type Post struct {
	Model
	Title string
	Tags  datatypes.JSON
}

func (p *Post) SetTags(tags []string) {
	p.Tags, _ = json.Marshal(tags)
}

func (p *Post) GetTags() []string {
	var tags []string
	json.Unmarshal(p.Tags, &tags)
	return tags
}
```

## 总结

Go语言提供了多种ORM框架选择，每种框架都有其优缺点和适用场景：

1. **GORM**：功能最全面的ORM框架，提供自动迁移、关联关系、钩子方法等丰富功能，适合大多数项目。

2. **SQLx**：轻量级SQL辅助库，提供了比原生database/sql更便捷的API，同时保持接近原生的性能，适合对性能要求较高的项目。

3. **XORM**：另一个全功能ORM框架，提供丰富的查询API和详细的日志支持，是GORM的良好替代品。

选择合适的ORM框架应考虑项目需求、团队经验、性能要求等因素。无论使用哪种框架，都应遵循最佳实践，如合理使用事务、避免N+1查询问题、优化批量操作等，以构建高效、可维护的数据库应用。

## 推荐阅读

- [GORM官方文档](https://gorm.io/docs/)
- [SQLx GitHub](https://github.com/jmoiron/sqlx)
- [XORM官方文档](https://xorm.io/docs)
- [数据库性能优化指南](https://use-the-index-luke.com/)
- [SQL反模式](https://www.oreilly.com/library/view/sql-antipatterns/9781680500073/)
