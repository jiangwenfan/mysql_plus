mysql_plus
======

一个为Dart编程语言开发的MySQL驱动程序。支持Flutter和服务器端应用。

这个库旨在为MySQL提供易于使用的接口。`mysql_plus`最初是从`mysql1`分叉而来

使用方法
--------

连接到数据库

```dart
var settings = new ConnectionSettings(
  host: 'localhost', 
  port: 3306,
  user: 'bob',
  password: 'wibble',
  db: 'mydb'
);
var conn = await MySqlConnection.connect(settings);
```

执行带参数的查询：

```dart
var userId = 1;
var results = await conn.query('select name, email from users where id = ?', [userId]);
```

使用查询结果：

```dart
for (var row in results) {
  print('姓名: ${row[0]}, 邮箱: ${row[1]}');
};
```

插入数据

```dart
var result = await conn.query('insert into users (name, email, age) values (?, ?, ?)', ['Bob', 'bob@bob.com', 25]);
```

插入查询的结果将为空，但如果表中有自动递增列，则会有一个id：

```dart
print("新用户的id: ${result.insertId}");
```

使用多组参数执行查询：

```dart
var results = await query.queryMulti(
    'insert into users (name, email, age) values (?, ?, ?)',
    [['Bob', 'bob@bob.com', 25],
    ['Bill', 'bill@bill.com', 26],
    ['Joe', 'joe@joe.com', 37]]);
```

更新数据：

```dart
await conn.query(
    'update users set age=? where name=?',
    [26, 'Bob']);
```

Flutter Web
-----------

这个包会打开到数据库的套接字连接。Web平台不支持套接字，因此这个包无法在Flutter Web上工作。