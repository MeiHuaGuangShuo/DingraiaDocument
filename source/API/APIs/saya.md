# Saya 模块

Saya模块是Dingraia的插件系统,用于实现模块化的功能扩展。

## 核心概念

### Channel

Channel是Saya的核心概念,代表一个功能模块。

#### 属性
- `name`: 模块名称
- `description`: 模块描述
- `author`: 模块作者
- `version`: 模块版本

#### 方法
- `use()`: 注册事件监听器
- `broadcast()`: 广播事件
- `current()`: 获取当前Channel实例

### Saya

Saya类是插件系统的主类。

#### 属性
- `channels`: 已加载的Channel列表

#### 方法
- `install_channel()`: 安装Channel
- `uninstall_channel()`: 卸载Channel
- `reload_channel()`: 重载Channel

## 内置功能

### 广播系统

广播系统用于实现事件的发布和订阅。

#### 事件类型
- `GroupMessage`: 群消息事件
- `PrivateMessage`: 私聊消息事件
- `GroupMemberIncrease`: 群成员增加事件
- `GroupMemberDecrease`: 群成员减少事件
- 其他自定义事件...

#### 使用方法
```python
from dingraia.saya import Channel
from dingraia.saya.builtins.broadcast import ListenEvent

channel = Channel.current()

@channel.use(
    ListenEvent=[GroupMessage]
)
async def message_handler(group: Group, message: MessageChain):
    # 处理群消息
    pass
```

### 上下文管理

提供了事件处理的上下文管理功能。

#### 上下文类型
- `MessageContext`: 消息上下文
- `EventContext`: 事件上下文
- `BroadcastContext`: 广播上下文

#### 使用方法
```python
from dingraia.saya import Channel
from dingraia.saya.context import MessageContext

channel = Channel.current()

@channel.use(MessageContext)
async def handler(context: MessageContext):
    # 使用上下文信息
    group = context.group
    message = context.message
    sender = context.sender
```

## 模块开发

### 基本结构

一个典型的Saya模块结构如下:

```
my_module/
  __init__.py      # 模块入口
  events.py        # 事件定义
  handlers.py      # 事件处理器
  models.py        # 数据模型
  utils.py         # 工具函数
```

### 模块示例

```python
from dingraia.saya import Channel
from dingraia.saya.builtins.broadcast import ListenEvent
from dingraia.message.chain import MessageChain

# 创建Channel
channel = Channel.current()
channel.name = "示例模块"
channel.description = "这是一个示例模块"
channel.author = "作者名"
channel.version = "1.0.0"

# 注册事件处理器
@channel.use(
    ListenEvent=[GroupMessage]
)
async def echo(group: Group, message: MessageChain):
    # 简单的复读功能
    await app.send_message(group, message)
```

## 最佳实践

1. 模块化设计
   - 每个功能独立成模块
   - 模块之间低耦合
   - 功能单一原则

2. 事件处理
   - 合理使用事件过滤
   - 异常处理完善
   - 避免阻塞操作

3. 上下文使用
   - 正确管理上下文
   - 避免上下文泄露
   - 及时释放资源

4. 性能优化
   - 减少不必要的事件监听
   - 优化事件处理逻辑
   - 合理使用缓存

## 注意事项

1. 模块加载
   - 模块需要在应用启动前加载
   - 避免循环依赖
   - 注意加载顺序

2. 事件监听
   - 不要过度使用事件监听
   - 合理设置事件优先级
   - 避免事件处理死循环

3. 资源管理
   - 及时清理不用的资源
   - 避免内存泄露
   - 正确关闭连接

4. 错误处理
   - 完善的异常处理
   - 合理的日志记录
   - 优雅的失败处理 