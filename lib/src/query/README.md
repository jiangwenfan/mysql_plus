# Query 查询处理模块文档

## 模块概述
查询处理模块负责处理MySQL的标准SQL查询操作，包括SELECT、INSERT、UPDATE、DELETE等语句的执行和结果处理。与预处理语句不同，这里处理的是直接的SQL文本查询。

## 文件结构
```
query/
├── query_stream_handler.dart      # 查询流处理器
├── standard_data_packet.dart      # 标准数据包处理
└── result_set_header_packet.dart  # 结果集头包处理
```

## 核心组件

### QueryStreamHandler (query_stream_handler.dart)
- **功能**: 处理标准SQL查询的执行和结果流处理
- **职责**:
  - 发送COM_QUERY命令
  - 管理查询执行状态
  - 处理流式结果返回
  - 支持大结果集的内存友好处理

**查询执行流程**:
1. 创建查询请求包
2. 发送SQL文本到服务器
3. 接收结果集头包
4. 接收字段定义包
5. 接收数据行包
6. 处理EOF或OK包

**状态管理**:
```dart
class QueryStreamHandler {
  static const int STATE_HEADER_PACKET = 0;    // 等待头包
  static const int STATE_FIELD_PACKETS = 1;    // 接收字段定义
  static const int STATE_ROW_PACKETS = 2;      // 接收数据行
}
```

### StandardDataPacket (standard_data_packet.dart)
- **功能**: 处理查询结果的标准数据包格式
- **职责**:
  - 解析文本格式的数据行
  - 处理NULL值标记
  - 字符串数据解码
  - 类型转换和验证

**数据包格式**:
- 文本协议格式（与二进制格式相对）
- 每个字段值作为字符串传输
- NULL值用特殊标记表示
- 支持所有MySQL数据类型

### ResultSetHeaderPacket (result_set_header_packet.dart)
- **功能**: 解析查询结果集的头部信息
- **包含信息**:
  - 结果集中的列数
  - 额外信息字段
  - 后续包的数量预告

## 查询处理流程

### 1. 查询请求
```
客户端 → 服务器: COM_QUERY + SQL文本
```

### 2. 结果集响应
```
服务器 → 客户端: 结果集头包 (列数)
服务器 → 客户端: N个字段定义包
服务器 → 客户端: EOF包 (字段定义结束)
服务器 → 客户端: M个数据行包
服务器 → 客户端: EOF包 (数据结束) 或 OK包
```

### 3. 流式处理
```dart
// 支持大结果集的流式处理
await for (var row in results) {
  // 逐行处理，内存占用可控
  processRow(row);
}
```

## 数据类型处理

### 文本格式转换
标准查询使用文本协议，所有数据都以字符串形式传输：

```dart
// 数字类型转换
var intValue = int.parse(row['id']);
var doubleValue = double.parse(row['price']);

// 日期时间转换  
var dateValue = DateTime.parse(row['created_at']);

// 布尔值转换
var boolValue = row['is_active'] == '1';

// NULL值处理
var nullableValue = row['optional_field']; // 可能为null
```

### 支持的查询类型

#### SELECT查询
```dart
var results = await conn.query('SELECT * FROM users WHERE age > 18');
await for (var row in results) {
  print('用户: ${row['name']}, 年龄: ${row['age']}');
}
```

#### INSERT查询
```dart
var result = await conn.query(
  "INSERT INTO users (name, email) VALUES ('张三', 'zhang@example.com')"
);
print('插入ID: ${result.insertId}');
print('影响行数: ${result.affectedRows}');
```

#### UPDATE查询
```dart
var result = await conn.query(
  "UPDATE users SET email = 'new@example.com' WHERE id = 1"
);
print('更新行数: ${result.affectedRows}');
```

#### DELETE查询
```dart
var result = await conn.query('DELETE FROM users WHERE age < 18');
print('删除行数: ${result.affectedRows}');
```

## 性能考虑

### 内存管理
```dart
// 大结果集处理 - 流式读取
var results = await conn.query('SELECT * FROM large_table');
var count = 0;
await for (var row in results) {
  count++;
  // 避免将所有行存储在内存中
  if (count % 1000 == 0) {
    print('已处理 $count 行');
  }
}
```

### 查询优化
1. **使用索引**: 确保WHERE条件使用了适当的索引
2. **限制结果**: 使用LIMIT子句控制返回行数
3. **选择性查询**: 只查询需要的列，避免SELECT *
4. **批量操作**: 考虑使用批量INSERT/UPDATE

## 与预处理语句的比较

| 特性 | 标准查询 | 预处理语句 |
|------|----------|------------|
| SQL注入防护 | 需要手动转义 | 自动防护 |
| 性能 | 每次解析SQL | 一次准备多次执行 |
| 传输协议 | 文本格式 | 二进制格式 |
| 使用场景 | 一次性查询 | 重复执行查询 |
| 参数绑定 | 字符串拼接 | 类型安全绑定 |

## 安全注意事项

### SQL注入防护
标准查询容易受到SQL注入攻击，需要谨慎处理用户输入：

```dart
// 危险做法 - 直接拼接用户输入
var userInput = "'; DROP TABLE users; --";
await conn.query("SELECT * FROM users WHERE name = '$userInput'");

// 安全做法 - 使用预处理语句
var query = await conn.prepare('SELECT * FROM users WHERE name = ?');
await query.execute([userInput]);

// 或者手动转义（不推荐）
var escaped = conn.quoteValue(userInput);
await conn.query("SELECT * FROM users WHERE name = $escaped");
```

### 数据验证
```dart
// 输入验证
bool isValidEmail(String email) {
  return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
}

if (isValidEmail(userEmail)) {
  await conn.query("INSERT INTO users (email) VALUES ('$userEmail')");
} else {
  throw ArgumentError('无效的邮箱格式');
}
```

## 错误处理

### 常见错误类型
```dart
try {
  var results = await conn.query('SELECT * FROM non_existent_table');
} catch (e) {
  if (e is MySqlException) {
    switch (e.errorNumber) {
      case 1146: // Table doesn't exist
        print('表不存在');
        break;
      case 1064: // SQL syntax error
        print('SQL语法错误');
        break;
      default:
        print('数据库错误: ${e.message}');
    }
  }
}
```

### 超时处理
```dart
try {
  var results = await conn.query('SELECT * FROM huge_table')
      .timeout(Duration(seconds: 30));
} catch (e) {
  if (e is TimeoutException) {
    print('查询超时');
  }
}
```

## 调试技巧

### SQL日志记录
```dart
// 启用查询日志
Logger.root.level = Level.FINEST;
Logger.root.onRecord.listen((record) {
  if (record.loggerName.contains('Query')) {
    print('执行SQL: ${record.message}');
  }
});
```

### 查询分析
```dart
// 分析查询执行计划
var explainResults = await conn.query('EXPLAIN SELECT * FROM users WHERE age > 25');
await for (var row in explainResults) {
  print('执行计划: ${row.values}');
}
```

## 最佳实践
1. **参数化查询**: 优先使用预处理语句
2. **错误处理**: 妥善处理各种异常情况
3. **资源管理**: 及时关闭连接和释放资源
4. **性能监控**: 记录慢查询并进行优化
5. **安全第一**: 永远不要直接拼接用户输入到SQL中