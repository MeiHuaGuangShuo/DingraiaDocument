# Dingraia APIs

## 注意事项

除非特殊说明，否则默认为已经为 dingraia 配置了 `config` 参数的完整版本，否则**只可以**发送 `WebHook` 消息

对于已经配置好 `main.py` 并可以正常运行的情况，可以使用 `python -m dingraia` 启用控制台模式，框架将自动加载app配置并载入大部分所需模块，无需再次导入

## `__init__`

初始化 `Dingtalk` 类。

### 参数

- `config`: `dingraia.config.Config` 配置类实例，用于初始化 `Dingtalk` 类。

### 示例

```python
from dingraia.config import Config, Bot, CallBack, Stream
from dingraia.DingTalk import Dingtalk

app = Dingtalk(config=Config(
    bot=Bot('AppKey',
            'AppSecret',
            'CorpId'),
    # 机器人配置，必填
    event_callback=CallBack("AesKey",
                            "Token",
                            "AppKey"),
    # 事件回调配置，可选
    stream=[
        Stream('AppKey',
               'AppSecret')
    ],
    # Stream配置，可选
    customStreamConnect=None,
    # 自定义Stream连接器，可选
    autoBotConfig=True,
    # 是否自动配置机器人，如果不想在使用Stream时自动配置机器人，可设置为False，可选
    useDatabase=True,
    # 是否使用数据库，可选。（建议使用，数据库操作本身占用时间极小，远远小于网络连接）
    dataCacheTime=3600,
    # 数据缓存时间，可选。（建议适当调高，避免频繁访问，减少网络请求次数）
    webRequestHandlers=None,
    # Web请求处理器，可选。
    raise_for_api_error=True,
    # 是否在API调用失败时抛出API错误，可选。（建议开启）
))
```

## `send_message`

向指定用户发送消息。

### 额外需求

- 发送到 OpenConversationId、发送文件、模板消息
  - 需要配置 `Dingtalk.config.bot`
  - API 消耗：`1`
- 发送到群聊、私聊、Webhook
  - 缓存过期
    - 需要配置 `Dingtalk.config.bot`
    - API 消耗：`1`
- 在 `MessageChain` 中包含 `File` 对象
  - 需要配置 `Dingtalk.config.bot`
  - API 消耗：`1`×`File`数量

### 参数

- `target`: 要发送消息的目标，可以是 `Group`, `Member`, `OpenConversationId`, `Webhook`, `str`
- `*msg`: 要发送的对象，可以是 `MessageChain`, `File`, `str`, `list`
  - 特别注意：如果 `msg` 是 `list`，则会被逐个执行并返回 `list` 结果

### 返回值

- `dingraia.element.Response`: 发送消息的响应

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.message.chain import MessageChain

app = Dingtalk()
app.prepare()
res = await app.send_message(OpenConversationId('cid1145149191810'), MessageChain('Hello, world!'))
# 一定要配置config参数，否则只可以发送WebHook消息
# 在缓存启用时，可以通过带 id 的 Group 实例发送消息，若过期则自动转为 API 发送
# res = <Response True>
```

## `sendMessage`

对 `send_message` 的同步函数包装，使用方式同 `send_message`。

## `recall_message`

撤回消息。

### 额外需求

需要配置 `Dingtalk.config.bot`

API 消耗：`1`

### 参数

- `message`: 撤回的消息对象，可以是 `Message`, `List[Message]`
- `openConversationId`: 消息所在对话，可以是 `OpenConversationId`, `Group`, `str`
- `processQueryKeys`: 消息的唯一标识，可以是 `str`, `List[str]`
- `robotCode`: 机器人标识，可以是 `str`, 若配置了 `config` 则可不填
- `inThreadTime`: 等待x秒后撤回，可以是 `int`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.message.chain import MessageChain


app = Dingtalk()
app.prepare()
send = await app.send_message(OpenConversationId('cid1145149191810'), MessageChain('Hello, world!'))
res = await app.recall_message(send)
```

## `send_ding`

发送 Ding 消息

### 注意事项

本接口 **尚未测试**

### 额外需求

- **钉钉专业版专属功能**
- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `targets`: 要发送到的人员列表，可以是 `Member`, `List[Member]`
- `content`: 要发送的消息内容，可以是 `str`
- `remindType`: 提醒类型，可以是 `int`
  - `1`: 应用内提醒
  - `2`: 短信提醒
  - `3`: 电话提醒


### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import Member


app = Dingtalk()
app.prepare()
user = Member()
user.staffId = '1145149191810'
res = await app.send_ding([user], 'Hello, world!', 1)
```

## `send_card`

发送卡片消息

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`2`

### 参数
- `target`: 要发送到的目标，可以是 `OpenConversationId`, `Group`, `Member`
- `cardTemplateId`: 卡片模板 ID
- `cardData`: 卡片数据，字典类型（本接口默认将数据包裹在 `cardData`->`cardParamMap` 中）
- `privateData`: 私有数据，可以是 `dict` 或者 `PrivateDataBuilder` 实例（使用PrivateDataBuilder可以更方便地设置私有数据）
- `supportForward`: 是否支持折叠，默认 `False`
- `callbackType`: 回调类型，可以是 `auto`, `Stream`, `HTTP`
- `outTrackId`: 外部追踪 ID，默认随机生成


### 返回值

- `dingraia.element.CardResponse`: 包含原始卡片的公共数据、私有数据和追踪 ID

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.send_card(
    OpenConversationId('cid1145149191810'), 
    'template_id', 
    {'title': 'Hello, world!'}
)
```

## `send_markdown_card`

发送 Markdown 卡片消息

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`2`

### 参数

- `target`: 要发送到的目标，可以是 `OpenConversationId`, `Group`, `Member`
- `markdown`: Markdown 内容，必须是 `Markdown` 实例
- `logo`: 卡片 logo，可以是 `File` 实例或者 `@` 开头的图片 URL
- `outTrackId`: 外部追踪 ID，默认随机生成
- `supportForward`: 是否支持折叠，默认 `False`

### 返回值

- `dingraia.element.CardResponse`: 包含原始卡片的公共数据、私有数据和追踪 ID

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.message.element import Markdown


app = Dingtalk()
app.prepare()
res = await app.send_markdown_card(
    OpenConversationId('cid1145149191810'), 
    Markdown('## Hello, world!', title='标题'), 
    '@mediaId'  # 也可以使用 File 实例
)
```

## `update_card`

更新卡片

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数
            outTrackId: Union[str, CardResponse],
            cardParamData: Union[dict, Markdown],
            privateData: Union[dict, PrivateDataBuilder] = None,


- `outTrackId`: 卡片的外部追踪 ID，可以是 `str` 或者 `CardResponse` 实例
- `cardParamData`: 卡片数据，可以是 `dict` 或者 `Markdown` 实例
- `privateData`: 私有数据，可以是 `dict` 或者 `PrivateDataBuilder` 实例（使用PrivateDataBuilder可以更方便地设置私有数据）

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.message.element import Markdown


app = Dingtalk()
app.prepare()
res = await app.send_markdown_card(
    'out_track_id', 
    Markdown('## Hello, world!', title='标题'), 
    {'key': 'value'}
)
await app.update_card(
    res, 
    Markdown('## Updated', title='标题'), 
)
```

## `sned_ai_card`

发送 AI 卡片消息

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`3+`(根据 AI 卡片更新的频率判定)

### 参数

- `target`: 要发送到的目标，可以是 `OpenConversationId`, `Group`, `Member`
- `cardTemplateId`: 卡片模板 ID
- `card`: 卡片数据，`AICard` 实例
- `update_limit`: 卡片更新频率限制，单位为秒，默认 `0` 表示不限制
- `key`: 卡片数据在卡片模板中的键名，默认 `content`
- `outTrackId`: 外部追踪 ID，默认随机生成

### 返回值

- `dingraia.element.CardResponse`: 包含原始卡片的公共数据、私有数据和追踪 ID

### 示例

由于 AI 卡片配置较为复杂，请参考 `example_module.AICard.py` 中的示例代码

如果你的 AI API接口支持 POST 方式的流式输出，可以使用示例代码中的方法进行快速流式输出

## `create_group`

创建群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `name`: 群聊名称
- `templateId`: 群聊模板 ID
- `ownerUserId`: 群主 ID
- `icon`: 群聊头像，可以是 `File` 实例或者 `@` 开头的图片 URL
- `userIds`: 群成员 ID 列表，可以是 `Member`, `List[Member]`, `str`, `List[str]`
- `subAdminIds`: 群管理员 ID 列表，可以是 `Member`, `List[Member]`, `str`, `List[str]`
- `showHistory`: 是否展示群历史消息，默认 `False`
- `validation`: 是否开启群验证，默认 `True`
- `searchable`: 是否允许群搜索，默认 `False`
- `UUID`: 唯一标识符 ID，默认随机生成
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import Member


app = Dingtalk()
app.prepare()
user1 = Member()
user1.staffId = '1145149191810'
user2 = Member()
user2.staffId = '1145149191811'
res = await app.create_group(
    '测试群聊', 
    'template_id', 
    'owner_id', 
    '@mediaId', 
    [user1, user2]
)
```

## `get_group`

获取群聊信息，包含群成员列表

### 额外需求

- 群聊必须是场景群或者启用酷应用的群聊
- 需要配置 `Dingtalk.config.bot`
- API 消耗：`2`

### 参数

- `OpenConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.get_group(OpenConversationId('cid1145149191810'))
```

# 未完成...



