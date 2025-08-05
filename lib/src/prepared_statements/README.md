# Prepared Statements 预处理语句模块文档

## 模块概述
预处理语句模块实现了MySQL的预处理语句功能，提供了高效、安全的SQL执行方式。预处理语句可以防止SQL注入攻击，并且对于重复执行的查询具有更好的性能。

## 文件结构
```
prepared_statements/
├── prepared_query.dart           # 预处理查询对象
├── prepare_handler.dart          # 语句准备处理器
├── prepare_ok_packet.dart       # 准备成功响应包
├── execute_query_handler.dart   # 查询执行处理器
├── binary_data_packet.dart      # 二进制数据包处理
└── close_statement_handler.dart # 语句关闭处理器
```

## 核心组件

### PreparedQuery (prepared_query.dart)
- **功能**: 预处理查询的主要接口类
- **职责**:
  - 封装预处理语句的执行逻辑
  - 管理语句ID和参数
  - 提供执行和关闭接口

**主要方法**:
```dart
class PreparedQuery {
  // 执行预处理语句
  Future<Results> execute(List parameters);
  
  // 关闭预处理语句
  Future<void> deallocate();
}
```

### PrepareHandler (prepare_handler.dart)
- **功能**: 处理预处理语句的准备阶段
- **职责**:
  - 发送PREPARE命令到服务器
  - 解析服务器返回的语句元数据
  - 获取语句ID和参数信息

**工作流程**:
1. 发送COM_STMT_PREPARE命令
2. 接收PrepareOK包
3. 接收参数元数据
4. 接收结果集元数据

### ExecuteQueryHandler (execute_query_handler.dart)
- **功能**: 执行预处理语句
- **职责**:
  - 发送COM_STMT_EXECUTE命令
  - 处理参数绑定和编码
  - 接收和解析执行结果

**参数处理**:
- 支持所有MySQL数据类型
- 自动类型检测和转换
- NULL值处理
- 二进制数据处理

### BinaryDataPacket (binary_data_packet.dart)
- **功能**: 处理预处理语句的二进制数据包
- **职责**:
  - 二进制格式的数据编码/解码
  - 高效的数据传输
  - 类型安全的数据转换

**支持的数据类型**:
- 整数类型: TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT
- 浮点类型: FLOAT, DOUBLE, DECIMAL
- 字符串类型: VARCHAR, TEXT, CHAR
- 日期时间类型: DATE, TIME, DATETIME, TIMESTAMP
- 二进制类型: BINARY, VARBINARY, BLOB

### PrepareOkPacket (prepare_ok_packet.dart)
- **功能**: 解析预处理语句准备成功的响应包
- **包含信息**:
  - 语句ID
  - 参数数量
  - 结果列数量
  - 警告数量

### CloseStatementHandler (close_statement_handler.dart)
- **功能**: 关闭预处理语句，释放服务器资源
- **职责**:
  - 发送COM_STMT_CLOSE命令
  - 清理客户端资源
  - 从语句缓存中移除

## 预处理语句生命周期

### 1. 准备阶段 (Prepare)
```
客户端 → 服务器: COM_STMT_PREPARE + SQL语句
服务器 → 客户端: PrepareOK包 + 参数元数据 + 结果元数据
```

### 2. 执行阶段 (Execute)
```
客户端 → 服务器: COM_STMT_EXECUTE + 语句ID + 参数值
服务器 → 客户端: 结果集数据 (二进制格式)
```

### 3. 关闭阶段 (Close)
```
客户端 → 服务器: COM_STMT_CLOSE + 语句ID
```

## 使用示例

### 基本使用
```dart
// 准备预处理语句
var query = await conn.prepare('SELECT * FROM users WHERE id = ?');

// 执行语句
var results = await query.execute([userId]);

// 处理结果
await for (var row in results) {
  print('User: ${row['name']}');
}

// 关闭语句
await query.deallocate();
```

### 批量操作
```dart
var insertQuery = await conn.prepare(
  'INSERT INTO users (name, email) VALUES (?, ?)'
);

// 批量插入
for (var user in users) {
  await insertQuery.execute([user.name, user.email]);
}

await insertQuery.deallocate();
```

### 参数类型处理
```dart
var query = await conn.prepare('''
  INSERT INTO posts (title, content, created_at, is_published) 
  VALUES (?, ?, ?, ?)
''');

await query.execute([
  'Hello World',          // 字符串
  'This is content...',   // 文本
  DateTime.now(),         // 日期时间
  true,                   // 布尔值
]);
```

## 开发指南

### 参数绑定最佳实践
1. **类型匹配**: 确保Dart类型与MySQL字段类型兼容
2. **NULL处理**: 使用`null`值表示MySQL的NULL
3. **性能优化**: 重复使用预处理语句而不是每次都准备

### 错误处理
```dart
try {
  var query = await conn.prepare('SELECT * FROM invalid_table');
  var results = await query.execute([]);
} catch (e) {
  if (e is MySqlException) {
    print('SQL错误: ${e.message}');
  }
}
```

### 内存管理
- 及时调用`deallocate()`释放服务器资源
- 避免创建过多的预处理语句
- 考虑使用语句池进行管理

## 性能优化

### 语句重用
```dart
// 好的做法：重用预处理语句
var query = await conn.prepare('SELECT * FROM users WHERE age > ?');
for (var age in [18, 25, 30]) {
  var results = await query.execute([age]);
  // 处理结果...
}
await query.deallocate();

// 不好的做法：重复准备语句
for (var age in [18, 25, 30]) {
  var query = await conn.prepare('SELECT * FROM users WHERE age > ?');
  var results = await query.execute([age]);
  await query.deallocate();
}
```

### 参数优化
- 使用二进制传输减少数据大小
- 避免频繁的类型转换
- 合理设置批处理大小

## 安全考虑

### SQL注入防护
预处理语句天然防止SQL注入攻击：
```dart
// 安全：参数会被正确转义
var query = await conn.prepare('SELECT * FROM users WHERE name = ?');
await query.execute([userInput]); // userInput不会被解释为SQL代码

// 危险：直接字符串拼接
await conn.query('SELECT * FROM users WHERE name = "$userInput"');
```

### 数据验证
- 在执行前验证参数值
- 检查参数数量匹配
- 验证数据类型兼容性

## 调试技巧
1. **启用SQL日志**: 查看实际发送的SQL语句
2. **检查参数绑定**: 确认参数值正确传递
3. **验证语句格式**: 确保SQL语法正确
4. **监控资源使用**: 避免语句泄漏