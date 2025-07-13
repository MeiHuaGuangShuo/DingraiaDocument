# Model 模块

Model模块提供了钉钉机器人开发中常用的数据模型类。

## 基础类

### Group

群组类,用于表示钉钉群组。

#### 属性
- `id`: 群组ID
- `name`: 群组名称
- `openConversationId`: 群组会话ID
- `webhook`: 群组Webhook对象
- `member`: 当前操作成员对象
- `traceId`: 消息追踪ID

#### 方法
- `__str__()`: 返回群组ID
- `__int__()`: 返回群组ID的整数形式

### Member

成员类,用于表示钉钉用户。

#### 属性
- `id`: 用户ID
- `name`: 用户名称
- `staffId`: 员工ID
- `unionId`: 用户唯一标识
- `mobile`: 手机号码
- `avatar`: 头像URL
- `group`: 所属群组对象
- `traceId`: 消息追踪ID

#### 方法
- `__str__()`: 返回用户ID
- `__int__()`: 返回用户ID的整数形式

### OpenConversationId

群组会话ID类。

#### 属性
- `openConversationId`: 会话ID
- `name`: 会话名称
- `group_id`: 群组ID
- `traceId`: 消息追踪ID

#### 方法
- `__str__()`: 返回会话ID
- `__int__()`: 返回会话ID的整数形式

### Webhook

Webhook类,用于处理钉钉的Webhook回调。

#### 属性
- `url`: Webhook URL
- `expired_time`: 过期时间
- `_type`: Webhook类型

#### 方法
- `__str__()`: 返回Webhook URL

### TraceId

消息追踪ID类。

#### 属性
- `id`: 追踪ID

#### 方法
- `__str__()`: 返回追踪ID

### Bot

机器人类。

#### 属性
- `id`: 机器人ID
- `name`: 机器人名称
- `origin_id`: 原始ID
- `trace_id`: 追踪ID

#### 方法
- `__str__()`: 返回机器人ID

## 使用示例

```python
from dingraia.model import Group, Member, OpenConversationId

# 创建群组对象
group = Group()
group.id = "123456"
group.name = "测试群"

# 创建成员对象
member = Member()
member.id = "user123"
member.name = "张三"
member.staffId = "staff123"

# 设置关联关系
member.group = group
group.member = member

# 创建会话ID对象
conv_id = OpenConversationId("cid123456")
conv_id.name = "测试会话"
```

## 注意事项

1. 模型对象通常由框架自动创建和填充,一般不需要手动创建
2. 模型对象的属性可能不会全部填充,使用前需要判断
3. 模型对象之间可以建立关联关系,但要注意避免循环引用
4. 某些属性(如traceId)在异步操作中特别重要,需要正确传递 