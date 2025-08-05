# MySQL Plus 源码目录文档

## 目录概述
`src/` 目录包含了MySQL Plus库的所有核心实现代码，按功能模块组织成不同的子目录和文件。

## 目录结构
```
src/
├── auth/                    # 认证模块
├── handlers/                # 协议处理器模块
├── prepared_statements/     # 预处理语句模块
├── query/                   # 查询处理模块
├── results/                 # 结果处理模块
├── single_connection.dart   # 核心连接类
├── buffer.dart             # 数据缓冲区
├── buffered_socket.dart    # 缓冲Socket
├── constants.dart          # MySQL协议常量
├── blob.dart              # 二进制数据处理
├── mysql_client_error.dart # 客户端错误
├── mysql_exception.dart    # MySQL异常
└── mysql_protocol_error.dart # 协议错误
```

## 核心文件说明

### single_connection.dart
- **功能**: 核心数据库连接类
- **职责**: 管理与MySQL服务器的连接、执行查询、处理事务
- **主要类**: `MySqlConnection`, `ConnectionSettings`, `TransactionContext`

### buffer.dart
- **功能**: MySQL协议数据包的读写缓冲区
- **职责**: 提供二进制数据的序列化和反序列化功能
- **特性**: 支持多种数据类型的编码/解码

### buffered_socket.dart
- **功能**: 带缓冲功能的Socket包装器
- **职责**: 在标准Socket基础上提供缓冲I/O功能
- **优势**: 提高网络通信效率

### constants.dart
- **功能**: 定义MySQL协议相关常量
- **内容**: 包类型、命令类型、字段类型、客户端能力标志等
- **作用**: 确保协议实现的正确性和一致性

### blob.dart
- **功能**: 处理MySQL的二进制大对象(BLOB)数据
- **职责**: 提供BLOB数据的读写和转换功能
- **支持类型**: TINYBLOB, BLOB, MEDIUMBLOB, LONGBLOB

## 异常处理

### mysql_client_error.dart
- **功能**: 客户端错误定义
- **用途**: 处理客户端逻辑错误和配置错误

### mysql_exception.dart
- **功能**: MySQL服务器异常封装
- **用途**: 包装服务器返回的错误信息

### mysql_protocol_error.dart
- **功能**: 协议级别错误
- **用途**: 处理协议解析和通信错误

## 模块间关系

### 依赖关系图
```
single_connection.dart
├── auth/ (认证模块)
├── handlers/ (处理器模块)
├── prepared_statements/ (预处理语句)
├── query/ (查询处理)
├── results/ (结果处理)
├── buffer.dart
├── buffered_socket.dart
└── constants.dart
```

### 数据流向
```
用户API调用
    ↓
single_connection.dart (连接管理)
    ↓
handlers/ (协议处理器)
    ↓
buffer.dart (数据序列化)
    ↓
buffered_socket.dart (网络通信)
    ↓
MySQL服务器
    ↓
results/ (结果处理)
    ↓
用户应用
```

## 开发指南

### 添加新功能
1. 确定功能所属的模块
2. 在相应目录下创建新文件或扩展现有文件
3. 更新相关的处理器和常量定义
4. 在main library文件中导出新的公共API

### 修改现有功能
1. 找到对应的模块和文件
2. 理解现有的实现逻辑
3. 保持向后兼容性
4. 更新相关测试

### 调试技巧
1. 启用详细日志记录
2. 使用debug_handler查看协议交互
3. 检查buffer和socket的数据流
4. 验证常量和类型定义

## 性能考虑

### 关键路径优化
- `buffer.dart`: 数据序列化性能
- `buffered_socket.dart`: 网络I/O效率
- `results/`: 结果处理性能
- `prepared_statements/`: 预处理语句缓存

### 内存管理
- 及时释放大结果集
- 避免不必要的数据复制
- 合理使用缓冲区大小
- 正确关闭连接和释放资源

## 安全考虑

### 输入验证
- 所有用户输入必须经过验证
- 使用预处理语句防止SQL注入
- 正确处理特殊字符和编码

### 连接安全
- 支持SSL/TLS加密连接
- 安全的认证机制
- 密码和敏感信息的安全处理

## 测试策略

### 单元测试
- 每个模块都有对应的单元测试
- 测试覆盖核心功能和边界情况
- 使用mock对象隔离依赖

### 集成测试
- 测试完整的数据库交互流程
- 验证不同MySQL版本的兼容性
- 性能和压力测试

## 文档链接
- [认证模块详细文档](auth/README.md)
- [处理器模块详细文档](handlers/README.md)
- [预处理语句详细文档](prepared_statements/README.md)
- [查询处理详细文档](query/README.md)
- [结果处理详细文档](results/README.md)