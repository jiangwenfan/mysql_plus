# MySQL Plus 库架构文档

## 项目概述
这是一个用于连接和查询MySQL & MariaDB数据库的Dart库。该库实现了完整的MySQL协议，支持连接管理、查询执行、预处理语句等功能。

## 主要特性
- 完整的MySQL协议实现
- 支持SSL连接
- 预处理语句支持
- 连接池管理
- 事务支持
- 流式查询结果处理

## 核心架构

### 入口文件 (mysql1.dart)
- 作为库的主入口点，导出所有公共API
- 主要导出的类：
  - `MySqlConnection`: 数据库连接主类
  - `Results`: 查询结果类
  - `Field`: 字段信息类
  - `Row`: 行数据类
  - `Blob`: 二进制数据类
  - 各种异常类

### 目录结构概览

```
lib/src/
├── auth/                    # 认证相关模块
├── handlers/                # 协议处理器
├── prepared_statements/     # 预处理语句
├── query/                   # 查询处理
├── results/                 # 结果处理
├── single_connection.dart   # 核心连接类
├── buffer.dart             # 数据缓冲区
├── buffered_socket.dart    # 缓冲Socket
├── constants.dart          # MySQL协议常量
└── 异常处理相关文件
```

## 核心组件说明

### SingleConnection (single_connection.dart)
- **功能**: 核心数据库连接类，负责管理与MySQL服务器的连接
- **主要职责**:
  - 建立和维护数据库连接
  - 处理连接设置和配置
  - 管理事务
  - 执行查询和预处理语句
  - 连接池管理

### Buffer & BufferedSocket
- **Buffer**: 用于读写MySQL协议数据包的底层缓冲区
- **BufferedSocket**: 在标准Socket基础上添加缓冲功能

### Constants (constants.dart)
- 定义MySQL协议相关的常量
- 包含包类型、客户端能力标志、字段类型等

## 开发指南

### 添加新功能的建议流程
1. 在相应目录下创建新的处理器或扩展现有功能
2. 如果是协议级别的新功能，在handlers目录下创建新的处理器
3. 更新相关的测试文件
4. 在主入口文件中导出新的公共API

### 调试建议
- 使用`debug_handler.dart`进行协议级别的调试
- 查看`test/`目录下的测试用例了解使用方式
- 启用日志记录查看详细的协议交互

### 代码风格
- 遵循Dart官方代码风格指南
- 使用有意义的中文注释
- 保持代码的模块化和可测试性

## 详细文档
每个子目录都有专门的README文档，详细说明该模块的功能和使用方法：
- [认证模块文档](src/auth/README.md)
- [处理器模块文档](src/handlers/README.md)
- [预处理语句文档](src/prepared_statements/README.md)
- [查询处理文档](src/query/README.md)
- [结果处理文档](src/results/README.md)