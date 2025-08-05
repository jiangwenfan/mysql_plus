# Auth 认证模块文档

## 模块概述
认证模块负责处理与MySQL服务器的连接认证过程，包括握手协议、SSL连接、字符集设置等功能。

## 文件结构
```
auth/
├── auth_handler.dart          # 认证处理器
├── character_set.dart         # 字符集配置
├── handshake_handler.dart     # 握手协议处理器
└── ssl_handler.dart          # SSL连接处理器
```

## 核心组件

### HandshakeHandler (handshake_handler.dart)
- **功能**: 处理MySQL连接的初始握手协议
- **主要职责**:
  - 解析服务器的握手包
  - 构建客户端认证响应
  - 处理连接能力协商
  - 初始化连接状态

**关键特性**:
- 支持多种认证插件
- 处理服务器版本兼容性
- 管理连接标志和能力

### AuthHandler (auth_handler.dart)
- **功能**: 具体的认证逻辑处理
- **主要职责**:
  - 实现各种认证方法
  - 密码加密和验证
  - 处理认证响应

**支持的认证方式**:
- mysql_native_password
- caching_sha2_password
- 其他MySQL支持的认证插件

### SSLHandler (ssl_handler.dart)
- **功能**: 处理SSL/TLS加密连接
- **主要职责**:
  - 建立SSL连接
  - 证书验证
  - 加密通信管理

### CharacterSet (character_set.dart)
- **功能**: 字符集配置和管理
- **主要职责**:
  - 定义支持的字符集
  - 字符编码转换
  - 字符集协商

## 认证流程说明

### 1. 初始连接
```
客户端 → 服务器: TCP连接
服务器 → 客户端: 握手包(版本信息、能力、认证插件等)
```

### 2. 认证协商
```
客户端 → 服务器: 客户端认证包(用户名、密码hash、能力标志)
服务器 → 客户端: 认证结果(OK包或错误包)
```

### 3. SSL连接(可选)
```
客户端 → 服务器: SSL请求
服务器 → 客户端: SSL响应
建立SSL隧道...
继续认证流程
```

## 开发指南

### 添加新的认证方法
1. 在`AuthHandler`中添加新的认证插件支持
2. 实现对应的密码加密算法
3. 更新握手处理器以支持新的认证插件
4. 添加相应的测试用例

### 处理认证错误
- 检查用户名和密码是否正确
- 验证服务器是否支持所选的认证方法
- 确认客户端能力标志设置正确

### 调试认证问题
- 启用详细日志记录
- 检查握手包的内容
- 验证认证插件的兼容性
- 检查SSL证书配置

## 使用示例

### 基本连接设置
```dart
var settings = ConnectionSettings(
  host: 'localhost',
  port: 3306,
  user: 'username',
  password: 'password',
  db: 'database_name',
  useSSL: true,
  characterSet: CharacterSet.utf8mb4,
);
```

### 处理认证错误
```dart
try {
  var conn = await MySqlConnection.connect(settings);
} catch (e) {
  if (e is MySqlException) {
    // 处理认证相关错误
    print('认证失败: ${e.message}');
  }
}
```

## 注意事项
- 确保密码加密方式与服务器配置匹配
- SSL连接需要正确的证书配置
- 不同MySQL版本可能支持不同的认证插件
- 字符集设置影响数据的正确传输和显示