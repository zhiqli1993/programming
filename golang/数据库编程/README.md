# Go语言数据库编程

## 📚 学习目标
掌握使用Go语言与各类数据库交互的核心技能，包括关系型数据库和NoSQL数据库的连接、查询、事务处理和性能优化。

## 🎯 学习路径

### 第一阶段：关系型数据库 (核心)
1. **标准库database/sql** - 原生数据库交互
   - 数据库连接与连接池
   - 执行查询与更新
   - 预处理语句
   - 事务处理

2. **MySQL与PostgreSQL** - 主流关系型数据库
   - 驱动配置
   - CRUD操作
   - 批量操作
   - 数据类型映射

### 第二阶段：ORM框架 (重要)
3. **GORM使用** - 主流ORM框架
   - 模型定义与关联
   - 查询构建器
   - 关联查询
   - 事务管理

4. **其他ORM工具** - 多样化选择
   - SQLx
   - XORM
   - 轻量级解决方案
   - 性能对比

### 第三阶段：NoSQL数据库 (扩展)
5. **Redis与Go** - 键值存储
   - 连接与配置
   - 数据类型操作
   - 缓存策略

6. **MongoDB与Go** - 文档数据库
   - CRUD操作
   - 聚合查询
   - 索引优化

### 第四阶段：高级主题 (进阶)
7. **分库分表与性能优化** - 扩展能力
   - 分片策略
   - 连接池优化
   - 查询性能优化

8. **数据访问模式** - 架构设计
   - 仓储模式
   - 数据访问层设计
   - 多数据源集成

## 🎨 实践项目

1. **个人博客系统** - MySQL+GORM实现
2. **用户认证服务** - PostgreSQL+Redis组合
3. **商品库存系统** - 事务与并发控制
4. **订单处理系统** - 分布式事务处理

## 📖 学习资源

### 官方资源
- [Go database/sql 包文档](https://golang.org/pkg/database/sql/)
- [GORM官方文档](https://gorm.io/docs/)
- [Go-Redis文档](https://redis.uptrace.dev/)

### 推荐书籍
- 《Go数据库编程》
- 《高性能MySQL》
- 《Redis实战》

### 在线资源
- [Go Web编程 - 数据库篇](https://github.com/astaxie/build-web-application-with-golang)
- [GORM教程](https://gorm.io/docs/)
- [Go数据库示例](https://github.com/go-sql-driver/mysql/wiki/Examples)

掌握Go语言数据库编程，能够帮助您构建高性能、可靠的数据密集型应用，满足各类业务场景的需求。
