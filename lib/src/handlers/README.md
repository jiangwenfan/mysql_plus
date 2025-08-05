# Handlers 处理器模块文档

## 模块概述
处理器模块实现了MySQL协议的各种命令处理器，每个处理器负责处理特定类型的MySQL协议消息和命令。

## 文件结构
```
handlers/
├── handler.dart              # 处理器基类
├── debug_handler.dart        # 调试处理器
├── ok_packet.dart           # OK包处理
├── parameter_packet.dart    # 参数包处理
├── ping_handler.dart        # 心跳处理器
├── quit_handler.dart        # 退出处理器
└── use_db_handler.dart      # 切换数据库处理器
```

## 核心架构

### Handler基类 (handler.dart)
- **功能**: 所有处理器的基类，定义了处理器的标准接口
- **主要组件**:
  - `HandlerResponse`: 处理器响应类，包含处理结果和下一个处理器
  - `Handler`: 处理器抽象基类

**处理器模式**:
```dart
abstract class Handler {
  Buffer createRequest();                    // 创建请求包
  HandlerResponse processResponse(Buffer response); // 处理响应包
}
```

### 专用处理器

#### PingHandler (ping_handler.dart)
- **功能**: 处理数据库连接心跳检测
- **用途**: 
  - 保持连接活跃
  - 检测连接是否正常
  - 服务器状态检查

```dart
// 发送心跳命令
Buffer createRequest() {
  var buffer = Buffer(1);
  buffer.writeByte(COM_PING);
  return buffer;
}
```

#### QuitHandler (quit_handler.dart)
- **功能**: 处理数据库连接关闭
- **职责**:
  - 发送退出命令到服务器
  - 清理连接资源
  - 优雅关闭连接

```dart
// 发送退出命令
Buffer createRequest() {
  var buffer = Buffer(1);
  buffer.writeByte(COM_QUIT);
  return buffer;
}
```

#### UseDbHandler (use_db_handler.dart)
- **功能**: 切换当前使用的数据库
- **职责**:
  - 发送USE DATABASE命令
  - 处理数据库切换结果
  - 更新连接状态

#### DebugHandler (debug_handler.dart)
- **功能**: 调试和诊断工具
- **用途**:
  - 协议包内容调试
  - 连接状态诊断
  - 开发阶段问题排查

## 数据包处理

### OkPacket (ok_packet.dart)
- **功能**: 解析MySQL的OK响应包
- **包含信息**:
  - 受影响的行数
  - 最后插入的ID
  - 服务器状态标志
  - 警告数量

### ParameterPacket (parameter_packet.dart)
- **功能**: 处理预处理语句的参数包
- **职责**:
  - 参数元数据解析
  - 参数类型处理
  - 参数值编码

## 处理器工作流程

### 1. 请求创建
```
Handler.createRequest() → Buffer(包含MySQL命令的二进制数据)
```

### 2. 请求发送
```
Buffer → Socket → MySQL服务器
```

### 3. 响应处理
```
MySQL服务器 → Socket → Buffer → Handler.processResponse()
```

### 4. 结果返回
```
HandlerResponse(finished, nextHandler, result)
```

## 开发指南

### 创建新的处理器
1. 继承`Handler`基类
2. 实现`createRequest()`方法创建请求包
3. 实现`processResponse()`方法处理响应
4. 在相应的地方注册和使用新处理器

```dart
class MyCustomHandler extends Handler {
  @override
  Buffer createRequest() {
    var buffer = Buffer(1);
    buffer.writeByte(MY_COMMAND);
    // 添加命令参数...
    return buffer;
  }

  @override
  HandlerResponse processResponse(Buffer response) {
    // 解析响应包...
    return HandlerResponse(finished: true, result: result);
  }
}
```

### 处理器链模式
- 某些复杂操作需要多个处理器协作
- 通过`HandlerResponse.nextHandler`指定下一个处理器
- 支持动态处理器切换

### 错误处理
- 检查响应包的错误标志
- 抛出适当的异常类型
- 提供详细的错误信息

## 调试技巧

### 使用DebugHandler
```dart
// 启用调试模式
var debugHandler = DebugHandler();
// 查看协议包内容
```

### 日志记录
- 每个处理器都支持日志记录
- 使用Logger类记录详细的处理过程
- 调试时启用详细日志级别

### 常见问题排查
1. **命令执行失败**: 检查`createRequest()`生成的包格式
2. **响应解析错误**: 验证`processResponse()`的解析逻辑
3. **处理器链中断**: 确保`nextHandler`正确设置
4. **内存泄漏**: 检查处理器是否正确释放资源

## 扩展建议
- 添加新的MySQL命令支持
- 实现自定义协议扩展
- 优化性能敏感的处理器
- 增强错误处理和恢复机制

## 性能考虑
- 避免在处理器中执行重IO操作
- 合理使用缓冲区大小
- 减少不必要的内存分配
- 优化热路径代码