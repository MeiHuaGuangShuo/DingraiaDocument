# 事件模块

事件模块提供了钉钉机器人开发中使用的各种事件类。

## 基础事件类

### BasicEvent

所有事件的基类。

```python
class BasicEvent:
    type = "BasicEvent"
    trace_id: str = None
    
    def __init__(self, raw_mes: str = "", dec_mes: Union[str, dict] = ""):
        """
        初始化基础事件
        
        Args:
            raw_mes: 原始消息
            dec_mes: 解码后的消息
        """
```

#### 属性
- `type`: 事件类型
- `trace_id`: 追踪ID
- `raw_mes`: 原始消息
- `dec_mes`: 解码后的消息

## 框架事件

### LoadComplete

框架加载完成事件。

```python
class LoadComplete(BasicEvent):
    """框架加载完成事件"""
```

#### 使用示例
```python
from dingraia.saya import Channel
from dingraia.event.event import LoadComplete

channel = Channel.current()

@channel.use(ListenerSchema(listening_events=[LoadComplete]))
async def on_load_complete(app: Dingtalk):
    print("框架加载完成")
```

### RadioComplete

广播完成事件。

```python
class RadioComplete(BasicEvent):
    trace_id: str = None
```

### NoMessageSend

无消息发送事件。

```python
class NoMessageSend(BasicEvent):
    trace_id: str = None
```

### MessageSend

消息发送事件。

```python
class MessageSend(BasicEvent):
    trace_id: str = None
```

### CheckUrl

URL检查事件。

```python
class CheckUrl(BasicEvent):
    type = 'CheckUrl'
```

### CheckIn

签到事件。

```python
class CheckIn(BasicEvent):
    member: Member
    bot: Bot
```

## 群组事件

### ChatQuit

群成员退群事件。

```python
class ChatQuit(BasicEvent):
    """群成员退群事件"""
    operator: int
    """操作者的StaffID"""
    
    openConversationId: OpenConversationId
    """对话ID"""
    
    corpId: str
    """企业ID"""
    
    time: int
    """时间戳*1000"""
    
    chatId: str
    """聊天ID"""
    
    operatorUnionId: str
    """操作者的UnionID"""
```

### ChatKick

群成员被踢事件。

```python
class ChatKick(BasicEvent):
    """群成员被踢事件"""
    operator: int
    """操作者的StaffID"""
    
    userIds: List[int]
    """被踢成员的StaffID列表"""
    
    openConversationId: OpenConversationId
    """对话ID"""
    
    corpId: str
    """企业ID"""
    
    time: int
    """时间戳*1000"""
    
    chatId: str
    """聊天ID"""
    
    operatorUnionId: str
    """操作者的UnionID"""
```

### GroupNameChange

群名称变更事件。

```python
class GroupNameChange(BasicEvent):
    """群名称变更事件"""
    operator: int
    """操作者的StaffID"""
    
    operatorUnionId: str
    """操作者的UnionID"""
    
    openConversationId: OpenConversationId
    """对话ID"""
    
    title: str
    """变更后的群标题"""
    
    time: int
    """时间戳*1000"""
    
    chatId: str
    """聊天ID"""
    
    corpId: str
    """企业ID"""
```

### GroupDisband

群解散事件。

```python
class GroupDisband(BasicEvent):
    """群解散事件"""
    operator: int
    """操作者的StaffID"""
    
    operatorUnionId: str
    """操作者的UnionID"""
    
    openConversationId: OpenConversationId
    """对话ID"""
    
    title: str
    """群标题"""
    
    time: int
    """时间戳*1000"""
    
    chatId: str
    """聊天ID"""
    
    corpId: str
    """企业ID"""
```

## 日程事件

### CalendarEventChange

日程事件变更事件。

```python
class CalendarEventChange(BasicEvent):
    """日程事件变更事件"""
    eventId: str
    """事件ID"""
    
    calendarEventUpdateTime: int
    """日程事件更新时间（毫秒）"""
    
    calendarEventId: str
    """日程事件唯一标识"""
    
    calendarId: str
    """日程ID"""
    
    unionIdList: List[str]
    """涉及成员的UnionID列表"""
    
    changeType: str
    """变更类型"""
    
    legacyCalendarEventId: str
    """旧版日程事件唯一标识"""
    
    operator: dict
    """操作者权限组信息"""
    
    EventType: str = "calendar_event_change"
    """事件类型"""
    
    corpId: str
    """企业ID"""
```

## 圈子事件

### CircleUserAction

圈子成员操作事件。

```python
class CircleUserAction(BasicEvent):
    """圈子成员操作事件"""
    EventType: str = "circle_user_action"
    """事件类型"""
    
    allUserOnline: bool
    """是否所有用户在线"""
    
    eventId: str
    """事件ID"""
    
    circleCorpId: str
    """圈子企业ID"""
    
    optName: str
    """操作名称"""
    
    circleUserId: str
    """圈子内用户ID"""
    
    belongCorpId: str
    """所属企业ID"""
    
    optTime: int
    """操作时间戳"""
    
    circleOrgName: str
    """圈子名称"""
```

## 消息事件

### BasicMessage

消息事件基类。

```python
class BasicMessage:
    type = "BasicMessage"
    message: MessageChain
    sender: Member
```

### GroupMessage

群消息事件。

```python
class GroupMessage(BasicMessage):
    type = "GroupMessage"
    sender: Member
    group: Group
    bot: Bot
```

#### 使用示例
```python
from dingraia.saya import Channel
from dingraia.event.message import GroupMessage
from dingraia.message.chain import MessageChain

channel = Channel.current()

@channel.use(ListenerSchema(listening_events=[GroupMessage]))
async def on_group_message(group: Group, message: MessageChain, sender: Member):
    print(f"收到来自 {sender.name} 的消息: {message}")
    # 回复消息
    await app.send_message(group, MessageChain("收到你的消息"))
```

### AiAssistantMessage

AI助理消息事件。

```python
class AiAssistantMessage(BasicMessage):
    type = "AiAssistantMessage"
    sender: Member
    msgType: str
    group: Group
    webhook: Webhook
    openConversationId: OpenConversationId
    corpId: str
    conversationToken: str
    data: dict
```

#### 使用示例
```python
from dingraia.saya import Channel
from dingraia.event.message import AiAssistantMessage

channel = Channel.current()

@channel.use(ListenerSchema(listening_events=[AiAssistantMessage]))
async def on_ai_assistant_message(sender: Member, message: MessageChain):
    print(f"收到AI助理消息: {message}")
```

## 事件监听

### 使用Saya模块监听事件

```python
from dingraia.saya import Channel
from dingraia.saya.builtins.broadcast import ListenEvent
from dingraia.event.event import LoadComplete, ChatQuit, ChatKick, GroupNameChange
from dingraia.event.message import GroupMessage

channel = Channel.current()

# 监听框架加载完成事件
@channel.use(ListenerSchema(listening_events=[LoadComplete]))
async def on_load_complete(app: Dingtalk):
    print("框架加载完成")

# 监听群消息事件
@channel.use(ListenerSchema(listening_events=[GroupMessage]))
async def on_group_message(group: Group, message: MessageChain):
    print(f"收到群消息: {message}")

# 监听群成员退群事件
@channel.use(ListenerSchema(listening_events=[ChatQuit]))
async def on_chat_quit(event: ChatQuit):
    print(f"成员退群: {event.operator}")

# 监听群成员被踢事件
@channel.use(ListenerSchema(listening_events=[ChatKick]))
async def on_chat_kick(event: ChatKick):
    print(f"成员被踢: {event.userIds}")

# 监听群名称变更事件
@channel.use(ListenerSchema(listening_events=[GroupNameChange]))
async def on_group_name_change(event: GroupNameChange):
    print(f"群名称变更: {event.title}")
```

### 使用装饰器监听事件

```python
from dingraia.lazy import *

# 监听群消息
@channel.use(ListenerSchema(listening_events=[GroupMessage]))
async def handle_group_message(app: Dingtalk, group: Group, message: MessageChain, sender: Member):
    # 处理群消息
    pass

# 监听多个事件
@channel.use(ListenerSchema(listening_events=[ChatQuit, ChatKick]))
async def handle_member_leave(event):
    # 处理成员离开事件
    pass
```

## 最佳实践

1. **事件处理**
   - 及时处理事件，避免阻塞
   - 合理使用事件过滤
   - 完善的异常处理

2. **性能优化**
   - 避免在事件处理中执行耗时操作
   - 合理使用异步处理
   - 避免过度监听事件

3. **错误处理**
   - 记录关键事件的日志
   - 优雅处理异常情况
   - 避免事件处理死循环