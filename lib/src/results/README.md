# Results 结果处理模块文档

## 模块概述
结果处理模块负责处理MySQL查询返回的结果数据，提供了统一的接口来访问查询结果的字段定义、行数据和元数据信息。

## 文件结构
```
results/
├── field.dart          # 字段定义和元数据
├── row.dart            # 行数据访问接口
└── results_impl.dart   # 结果集实现
```

## 核心组件

### Field (field.dart)
- **功能**: 表示查询结果中的字段（列）定义
- **职责**:
  - 存储字段元数据信息
  - 定义字段类型和属性
  - 提供字段信息查询接口

**字段属性**:
```dart
class Field {
  String? catalog;      // 数据库目录名
  String? db;          // 数据库名
  String? table;       // 表名
  String? orgTable;    // 原始表名
  String? name;        // 字段名
  String? orgName;     // 原始字段名
  int? characterSet;   // 字符集
  int? length;         // 字段长度
  int? type;          // 字段类型
  int? flags;         // 字段标志
  int? decimals;      // 小数位数
  dynamic defaultValue; // 默认值
}
```

**字段类型常量**:
- `FIELD_TYPE_DECIMAL`: 十进制数
- `FIELD_TYPE_TINY`: TINYINT
- `FIELD_TYPE_SHORT`: SMALLINT
- `FIELD_TYPE_LONG`: INT
- `FIELD_TYPE_FLOAT`: FLOAT
- `FIELD_TYPE_DOUBLE`: DOUBLE
- `FIELD_TYPE_NULL`: NULL
- `FIELD_TYPE_TIMESTAMP`: TIMESTAMP
- `FIELD_TYPE_LONGLONG`: BIGINT
- `FIELD_TYPE_INT24`: MEDIUMINT
- `FIELD_TYPE_DATE`: DATE
- `FIELD_TYPE_TIME`: TIME
- `FIELD_TYPE_DATETIME`: DATETIME
- `FIELD_TYPE_YEAR`: YEAR
- `FIELD_TYPE_NEWDATE`: NEWDATE
- `FIELD_TYPE_VARCHAR`: VARCHAR
- `FIELD_TYPE_BIT`: BIT
- `FIELD_TYPE_JSON`: JSON
- `FIELD_TYPE_NEWDECIMAL`: NEW DECIMAL
- `FIELD_TYPE_ENUM`: ENUM
- `FIELD_TYPE_SET`: SET
- `FIELD_TYPE_TINY_BLOB`: TINYBLOB
- `FIELD_TYPE_MEDIUM_BLOB`: MEDIUMBLOB
- `FIELD_TYPE_LONG_BLOB`: LONGBLOB
- `FIELD_TYPE_BLOB`: BLOB
- `FIELD_TYPE_VAR_STRING`: VAR_STRING
- `FIELD_TYPE_STRING`: STRING
- `FIELD_TYPE_GEOMETRY`: GEOMETRY

### ResultRow (row.dart)
- **功能**: 表示查询结果中的单行数据
- **职责**:
  - 提供多种数据访问方式
  - 支持索引和名称访问
  - 处理数据类型转换

**数据访问方式**:
```dart
abstract class ResultRow {
  List<dynamic>? values;           // 按索引访问的值列表
  
  // 按字段名访问
  dynamic operator [](String key);
  
  // 按索引访问  
  dynamic byIndex(int index);
  
  // 转换为Map
  Map<String, dynamic> toMap();
  
  // 获取所有值
  List<dynamic>? toList();
}
```

### ResultsImpl (results_impl.dart)
- **功能**: 查询结果集的具体实现
- **职责**:
  - 管理查询结果的元数据
  - 提供结果迭代接口
  - 处理流式结果

**结果集属性**:
```dart
class ResultsImpl implements Results {
  int? insertId;           // 最后插入的ID
  int? affectedRows;       // 影响的行数
  Stream<ResultRow>? stream; // 结果行流
  List<Field>? fields;     // 字段定义列表
}
```

## 使用示例

### 基本结果访问
```dart
var results = await conn.query('SELECT id, name, email FROM users LIMIT 5');

// 访问字段定义
for (var field in results.fields!) {
  print('字段: ${field.name}, 类型: ${field.type}, 长度: ${field.length}');
}

// 流式访问数据行
await for (var row in results) {
  // 按字段名访问
  print('ID: ${row['id']}');
  print('姓名: ${row['name']}');
  print('邮箱: ${row['email']}');
  
  // 按索引访问
  print('第一列: ${row.byIndex(0)}');
  
  // 转换为Map
  var rowMap = row.toMap();
  print('行数据: $rowMap');
  
  // 获取所有值
  var rowValues = row.toList();
  print('值列表: $rowValues');
}
```

### 字段类型检查
```dart
var results = await conn.query('DESCRIBE users');
await for (var row in results) {
  var fieldName = row['Field'];
  var fieldType = row['Type'];
  var isNullable = row['Null'] == 'YES';
  var defaultValue = row['Default'];
  
  print('字段 $fieldName:');
  print('  类型: $fieldType');
  print('  可空: $isNullable');
  print('  默认值: $defaultValue');
}
```

### 处理不同数据类型
```dart
var results = await conn.query('''
  SELECT 
    id,                    -- 整数
    name,                  -- 字符串
    price,                 -- 浮点数
    created_at,            -- 日期时间
    is_active,             -- 布尔值
    description,           -- 文本
    profile_image          -- 二进制数据
  FROM products
''');

await for (var row in results) {
  // 类型安全的访问
  var id = row['id'] as int;
  var name = row['name'] as String?;
  var price = row['price'] as double?;
  var createdAt = row['created_at'] as DateTime?;
  var isActive = row['is_active'] == 1;
  var description = row['description'] as String?;
  var profileImage = row['profile_image'] as Blob?;
  
  print('产品 $id: $name');
  if (price != null) print('价格: ¥$price');
  if (createdAt != null) print('创建时间: $createdAt');
  print('状态: ${isActive ? "激活" : "禁用"}');
}
```

### INSERT/UPDATE/DELETE结果处理
```dart
// INSERT查询
var insertResult = await conn.query(
  "INSERT INTO users (name, email) VALUES ('张三', 'zhang@example.com')"
);
print('新插入记录ID: ${insertResult.insertId}');
print('影响行数: ${insertResult.affectedRows}');

// UPDATE查询
var updateResult = await conn.query(
  "UPDATE users SET email = 'new@example.com' WHERE id = 1"
);
print('更新行数: ${updateResult.affectedRows}');

// DELETE查询
var deleteResult = await conn.query('DELETE FROM users WHERE age < 18');
print('删除行数: ${deleteResult.affectedRows}');
```

## 数据类型转换

### 自动类型转换
MySQL Plus库会自动将MySQL类型转换为相应的Dart类型：

| MySQL类型 | Dart类型 | 说明 |
|-----------|----------|------|
| INT, BIGINT | int | 整数类型 |
| FLOAT, DOUBLE | double | 浮点数类型 |
| DECIMAL | String | 精确小数（避免精度丢失） |
| VARCHAR, TEXT | String | 字符串类型 |
| DATE, DATETIME | DateTime | 日期时间类型 |
| TIME | Duration | 时间间隔类型 |
| BOOLEAN, TINYINT(1) | bool | 布尔类型 |
| BLOB, BINARY | Blob | 二进制数据 |
| NULL | null | 空值 |

### 手动类型转换
```dart
await for (var row in results) {
  // 安全的类型转换
  var id = int.tryParse(row['id']?.toString() ?? '0') ?? 0;
  var price = double.tryParse(row['price']?.toString() ?? '0.0') ?? 0.0;
  var isActive = ['1', 'true', 'yes'].contains(row['is_active']?.toString().toLowerCase());
  
  // 日期时间处理
  var createdAt = row['created_at'];
  if (createdAt is String) {
    createdAt = DateTime.tryParse(createdAt);
  }
}
```

## 性能优化

### 流式处理大结果集
```dart
// 避免将所有数据加载到内存
var results = await conn.query('SELECT * FROM large_table');
var count = 0;
var totalPrice = 0.0;

await for (var row in results) {
  count++;
  totalPrice += (row['price'] as double?) ?? 0.0;
  
  // 每处理1000行输出一次进度
  if (count % 1000 == 0) {
    print('已处理 $count 行，当前总价: $totalPrice');
  }
}

print('总计: $count 行，总价: $totalPrice');
```

### 选择性字段访问
```dart
// 只查询需要的字段
var results = await conn.query('SELECT id, name FROM users');
// 比 SELECT * 更高效

await for (var row in results) {
  // 只访问需要的字段
  print('${row['id']}: ${row['name']}');
}
```

## 错误处理

### NULL值处理
```dart
await for (var row in results) {
  // 安全的NULL值处理
  var name = row['name'] as String?;
  if (name != null && name.isNotEmpty) {
    print('用户名: $name');
  } else {
    print('用户名为空');
  }
  
  // 使用默认值
  var email = row['email'] as String? ?? 'no-email@example.com';
  var age = row['age'] as int? ?? 0;
}
```

### 类型转换异常
```dart
await for (var row in results) {
  try {
    var id = row['id'] as int;
    var price = row['price'] as double;
    
    // 处理数据...
  } catch (e) {
    print('数据类型转换错误: $e');
    print('行数据: ${row.toMap()}');
    continue; // 跳过此行继续处理
  }
}
```

## 最佳实践

### 1. 字段访问
```dart
// 推荐：使用字段名访问（更清晰）
var userName = row['name'];

// 避免：使用索引访问（容易出错）
var userName = row.byIndex(1); // 如果查询字段顺序改变会出错
```

### 2. 类型安全
```dart
// 推荐：显式类型转换
var id = row['id'] as int?;
var name = row['name'] as String?;

// 避免：假设类型
dynamic id = row['id']; // 失去类型安全
```

### 3. 内存管理
```dart
// 推荐：流式处理
await for (var row in results) {
  processRow(row);
}

// 避免：一次性加载所有数据
var allRows = <ResultRow>[];
await for (var row in results) {
  allRows.add(row); // 可能导致内存不足
}
```

### 4. 错误处理
```dart
// 推荐：适当的错误处理
try {
  await for (var row in results) {
    var data = processRow(row);
    // 处理数据...
  }
} catch (e) {
  logger.error('结果处理失败: $e');
  // 适当的错误恢复...
}
```

## 调试技巧

### 查看字段信息
```dart
// 打印字段定义信息
for (var field in results.fields!) {
  print('字段: ${field.name}');
  print('  数据库: ${field.db}');
  print('  表: ${field.table}');
  print('  类型: ${field.type}');
  print('  长度: ${field.length}');
  print('  标志: ${field.flags}');
  print('---');
}
```

### 查看原始数据
```dart
await for (var row in results) {
  print('原始行数据: ${row.toList()}');
  print('字段映射: ${row.toMap()}');
}
```