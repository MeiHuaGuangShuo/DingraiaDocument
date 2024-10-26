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
- API 消耗：`2` / `0`

### 参数

- `OpenConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `access_token`: 企业的 access_token，默认 `None`
- `using_cache`: 是否使用缓存，默认 `None` 表示自动判断，若缓存不存在会自动使用API获取

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

## `get_depts`

获取部门列表

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `deptId`: 部门 ID，默认 `1` 表示获取根部门列表
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk


app = Dingtalk()
app.prepare()
res = await app.get_depts()
```

## `get_dept_users`

获取部门成员列表

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `deptId`: 部门 ID，默认 `1` 表示获取根部门成员列表
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `list` 所有部门成员的 StaffId(`str`) 列表

### 示例

```python
from dingraia.DingTalk import Dingtalk


app = Dingtalk()
app.prepare()
res = await app.get_dept_users()
```

## `get_user`

获取用户信息

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1` / `0`

### 参数

- `userId`: 用户 ID，可以是 `Member`, `str`
- `language`: 语言，默认 `zh_CN`
- `access_token`: 企业的 access_token，默认 `None`
- `using_cache`: 是否使用缓存，默认 `None` 表示自动判断，若缓存不存在会自动使用API获取
  - 当使用缓存时，将会在 `dict` 中添加 `dingraia_cache` 键，包含以下内容
    - `id`: 用户 ID （本框架内）
    - `name`: 用户名
    - `staffId`: 组织内唯一标识
    - `unionId`: 钉钉用户唯一标识
    - `timeStamp`: 时间戳
    - `strfTimeStamp`: 格式化的时间

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
res = await app.get_user(user)
```

## `unionId2staffId`

根据 unionId 获取 staffId

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `unionId`: 钉钉用户唯一标识
- `using_cache`: 是否使用缓存，默认 `None` 表示自动判断，若缓存不存在会自动使用API获取

### 返回值

- `str` 组织内唯一标识
  - 若传入了 `Member` 实例，则会同时更新 `Member` 实例的 `staffId` 属性
  

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import Member


app = Dingtalk()
app.prepare()
user = Member()
user.unionId = 'unionId1145149191810'
res = await app.unionId2staffId(user)
```

## `remove_user`

移除用户

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `userStaffId`: 组织内唯一标识，可以是 `Member`, `str`
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk


app = Dingtalk()
app.prepare()
res = await app.remove_user('staffId1145149191810')
```

## `create_user`

创建用户

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数
- `name`: 用户名
- `mobilePhone`: 手机号
- `deptIds`: 部门 ID 列表，可以是 `str` 或者 `List[str]`
- `userId`: 用户 ID，默认随机生成
- `hidePhone`: 是否隐藏手机号
- `jobNumber`: 工号
- `positionName`: 职位
- `personalEmail`: 个人邮箱
- `organizeEmail`: 组织邮箱
- `organizeEmailType`: 组织邮箱类型，可以是 `base` 或者 `profession`
- `workPlace`: 办公地点
- `remark`: 备注
- `extension`: 扩展属性
- `isSenior`: 是否开启高管模式
- `managerUserId`: 上级用户 ID
- `loginEmail`: 登录邮箱


### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk


app = Dingtalk()
app.prepare()
res = await app.create_user(
    '测试用户', 
    '13800138000', 
    ['deptId1145149191810', 'deptId1145149191811'], 
    jobNumber='jobNumber1145149191810', 
    positionName='positionName1145149191810', 
    personalEmail='personalEmail1145149191810', 
    organizeEmail='organizeEmail1145149191810', 
    organizeEmailType='profession', 
    workPlace='workPlace1145149191810', 
    remark='remark1145149191810', 
    extension='extension1145149191810', 
    isSenior=True, 
    managerUserId='managerUserId1145149191810', 
    loginEmail='loginEmail1145149191810'
)
```

## `update_user`

更新用户

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`


### 参数

- `userId`: 用户 ID，可以是 `Member`, `str`
- `name`: 用户名
- `hide_mobile`: 是否隐藏手机号
- `telephone`: 手机号
- `job_number`: 工号
- `manager_userid`: 上级用户 ID
- `title`: 职位
- `email`: 个人邮箱
- `org_email`: 组织邮箱
- `work_place`: 办公地点
- `remark`: 备注
- `dept_id_list`: 部门 ID 列表，可以是 `str` 或者 `List[str]`
- `dept_title_list`: 部门名称列表，可以是 `dict` 或者 `List[dict]`
- `dept_order_list`: 部门排序列表，可以是 `dict` 或者 `List[dict]`
- `extension`: 扩展属性
- `senior_mode`: 是否开启高管模式
- `hired_date`: 入职时间
- `language`: 语言，默认 `zh_CN`
- `force_update_fields`: 强制更新的字段列表，默认 `None` 表示不强制更新

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
res = await app.update_user(
    user, 
    name='测试用户', 
    job_number='jobNumber1145149191810', 
    manager_userid='managerUserId1145149191810', 
    title='positionName1145149191810', 
    email='personalEmail1145149191810', 
    org_email='organizeEmail1145149191810', 
    work_place='workPlace1145149191810', 
    remark='remark1145149191810', 
    dept_id_list=['deptId1145149191810', 'deptId1145149191811'], 
    dept_title_list=[{'dept_id': 'deptId1145149191810', 'dept_title': '测试部门1'}, {'dept_id': 'deptId1145149191811', 'dept_title': '测试部门2'}], 
    dept_order_list=[{'dept_id': 'deptId1145149191810', 'dept_order': 1}, {'dept_id': 'deptId1145149191811', 'dept_order': 2}], 
    extension='extension1145149191810', 
    senior_mode=True, 
    hired_date='2021-01-01', 
    force_update_fields=['name', 'job_number', 'title', 'email', 'org_email', 'work_place', 'dept_id_list', 'dept_title_list', 'dept_order_list', 'extension','senior_mode', 'hired_date']
)
```

## `mirror_group`

创建群聊的镜像

### 注意事项

- 该 API 仅支持创建场景群聊的镜像，不支持创建普通群聊的镜像
- 通过此方法创建的群聊，会在群聊名字后方加上 ` - 转生` 字样
- 此方法会随机挑选临时群主，防止群主创建群过多无法成功创建群，随后通过API更改回原群主
- 此方法无视青少年模式

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`6+`

### 参数

- `OpenConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.mirror_group(OpenConversationId('cid1145149191810'))
```

## `update_group`

更新群聊

### 注意事项

- 该 API 仅支持更新场景群聊，不支持更新普通群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `title`: 群聊名称
- `owner_user_id`: 群主 ID
- `icon`: 群头像，可以是 `File` 实例，也可以是 mediaId
- `mention_all_authority`: 是否允许 @所有人
- `show_history_type`: 是否展示历史消息
- `validation_type`: 是否开启入群验证
- `searchable`: 是否可被搜索
- `chat_banned_type`: 是否全员禁言
- `management_type`: 管理类型
  - `0`: 所有人可管理
  - `1`: 仅群主可管理
- `only_admin_can_ding`: 是否仅群主能使用 DING 消息
  - `0`: 所有人
  - `1`: 仅群主
- `all_members_can_create_mcs_conf`: 群会议权限
  - `0`: 群主和管理员
  - `1`: 所有人
- `all_members_can_create_calendar`: 是否所有成员均可创建日程
- `group_email_disabled`: 是否禁用群邮件
- `only_admin_can_set_msg_top`: 是否仅群主能设置消息置顶
- `add_friend_forbidden`: 是否禁止加好友
- `group_live_switch`: 是否允许群直播
- `members_to_admin_chat`: 成员是否允许和群管理员聊天
- `plugin_customize_verify`: 自定义插件是否需要群主或管理员审批
- `access_token`: 企业的 access_token，默认 `None`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.update_group(
    OpenConversationId('cid1145149191810'), 
    title='测试群聊', 
    owner_user_id='ownerUserId1145149191810', 
    icon='mediaId1145149191810', 
    mention_all_authority=True, 
    show_history_type=1, 
    validation_type=0, 
    searchable=True, 
    chat_banned_type=0, 
    management_type=0, 
    only_admin_can_ding=0, 
    all_members_can_create_mcs_conf=0, 
    all_members_can_create_calendar=True, 
    group_email_disabled=False, 
    only_admin_can_set_msg_top=True, 
    add_friend_forbidden=False, 
    group_live_switch=True, 
    members_to_admin_chat=True, 
    plugin_customize_verify=False
)
```


## `disband_group`

解散群聊

### 注意事项

- 该 API 仅支持解散场景群聊，不支持解散普通群聊
- 此方法非官方方法，其原理为移出所有群成员，群聊仍然存在

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.disband_group(OpenConversationId('cid1145149191810'))
```

## `change_group_title` (`update_group` 的分支)

修改群聊名称

### 注意事项

- 该 API 仅支持更新场景群聊，不支持更新普通群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `title`: 群聊名称

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.change_group_title(OpenConversationId('cid1145149191810'), '测试群聊')
```

## `change_group_owner` (`update_group` 的分支)

修改群主

### 注意事项

- 该 API 仅支持更新场景群聊，不支持更新普通群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `owner_user_id`: 群主 ID

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.change_group_owner(OpenConversationId('cid1145149191810'), 'ownerUserId1145149191810')
```

## `mute_all` (`update_group` 的分支)

全员禁言

### 注意事项

- 该 API 仅支持更新场景群聊，不支持更新普通群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.mute_all(OpenConversationId('cid1145149191810'))
```


## `unmute_all` (`update_group` 的分支)

取消全员禁言

### 注意事项

- 该 API 仅支持更新场景群聊，不支持更新普通群聊

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()
res = await app.unmute_all(OpenConversationId('cid1145149191810'))
```

## `kick_member`

踢出群成员

### 注意事项

- 该 API 仅支持踢出场景群聊的成员，不支持踢出普通群聊的成员

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `memberStaffIds`: 用户 ID 列表，可以是 `Member`, `str` , `List[Member]` 或者 `List[str]`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId, Member


app = Dingtalk()
app.prepare()
res = await app.kick_member(
    OpenConversationId('cid1145149191810'), 
    [Member('staffId1145149191810'), Member('staffId1145149191811')]
)
```

## `add_member`

邀请群成员

### 注意事项

- 该 API 仅支持邀请场景群聊的成员，不支持邀请普通群聊的成员

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `memberStaffIds`: 用户 ID 列表，可以是 `Member`, `str` , `List[Member]` 或者 `List[str]`


### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId, Member


app = Dingtalk()
app.prepare()
res = await app.add_member(
    OpenConversationId('cid1145149191810'), 
    [Member('staffId1145149191810'), Member('staffId1145149191811')]
)
```

## `set_admin`

设置管理员

### 注意事项

- 该 API 仅支持设置场景群聊的管理员，不支持设置普通群聊的管理员

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `memberStaffIds`: 用户 ID 列表，可以是 `Member`, `str` , `List[Member]` 或者 `List[str]`
- `set_admin`: 设置管理员，`True` 为设置，`False` 为取消设置


### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId, Member


app = Dingtalk()
app.prepare()
res = await app.set_admin(
    OpenConversationId('cid1145149191810'), 
    [Member('staffId1145149191810'), Member('staffId1145149191811')]
)
```

## `mute_member`

禁言群成员

### 注意事项

- 该 API 仅支持禁言场景群聊的成员，不支持禁言普通群聊的成员

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `memberStaffIds`: 用户 ID 列表，可以是 `Member`, `str` , `List[Member]` 或者 `List[str]`
- `muteTime`: 禁言时长，单位为秒，`0` 为取消禁言
  - 似乎没有最大时间限制，你甚至可以禁言他10年

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId, Member


app = Dingtalk()
app.prepare()
res = await app.mute_member(
    OpenConversationId('cid1145149191810'), 
    [Member('staffId1145149191810'), Member('staffId1145149191811')],
    60*60*24*365 # 禁言一年
)
```

## `unmute_member` (`mute_member` 的分支)

取消禁言群成员

### 注意事项

- 该 API 仅支持取消禁言场景群聊的成员，不支持取消禁言普通群聊的成员

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `openConversationId`: 群聊 ID，可以是 `OpenConversationId`, `Group`, `str`
- `memberStaffIds`: 用户 ID 列表，可以是 `Member`, `str` , `List[Member]` 或者 `List[str]`

### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId, Member


app = Dingtalk()
app.prepare()
res = await app.unmute_member(
    OpenConversationId('cid1145149191810'), 
    [Member('staffId1145149191810'), Member('staffId1145149191811')]
)
```

## `set_off_duty_prompt`

设置 Stream 机器人离线提醒

### 注意事项

- 此接口只支持 Stream 机器人，不支持 HTTP 机器人

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数
            text: str = "人家今天下班了呢~请晚些再来找我吧",
            title: str = "钉钉Stream机器人",
            logo: Union[File, str] = "@lALPDfJ6V_FPDmvNAfTNAfQ",
            robotCode: str = None,
            access_token: str = None

- `text`: 离线提醒内容
- `title`: 离线提醒标题
- `logo`: 离线提醒头像，可以是 `File` 对象或者 `@` 开头的 mediaID
- `robotCode`: 机器人编码，如果不指定，则使用默认机器人
- `access_token`: 机器人 access_token，如果不指定，则使用默认机器人


### 返回值

- `dict` 钉钉原始返回（可能以后会改为更适合的返回类型）

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import Image


app = Dingtalk()
app.prepare()
res = await app.set_off_duty_prompt(
    text="人家今天下班了呢~请晚些再来找我吧",
    title="钉钉Stream机器人",
    logo=Image("path/to/your/logo.png")
)
```

## `upload_file`

上传文件

### 注意事项

- 上传完成的文件获取到的 mediaId 在全钉钉可用

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1` / `0`

### 参数

- `file`: 文件路径，可以是 `str` 或者 `File` 对象
  - 不建议直接使用 `File` 对象，建议使用对应对象
  - `Image` 对象大小必须是 `20MiB` 以下
  - `Voice` 对象大小必须是 `2MiB` 以下
  - `Video` 对象大小必须是 `20MiB` 以下
  - `File` 对象大小必须是 `20MiB` 以下

### 返回值

- 输入的为 `File` 对象，则更新其 `mediaId` 属性，并返回
- 输入的为 `str` 对象，则返回已经更新 `mediaId` 属性的 `File` 对象

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import File, Image


app = Dingtalk()
app.prepare()
image = await app.upload_file(Image("path/to/your/image.png"))
```

## `download_file`

下载文件

### 注意事项

- 需要指定下载的路径和文件名

### 额外需求

- 需要配置 `Dingtalk.config.bot`
- API 消耗：`1`

### 参数

- `downloadCode`: 下载码，可以是 `str`  或者 `File` 对象
- `path`: 下载路径，可以是 `str` 或者 `Path` 对象

### 返回值

- `bool` 下载成功与否

### 示例

忘记怎么写了，这个功能我很少用，因为不能自动判断文件类型，所以很麻烦

## `run_coroutine`

在同步函数中运行异步函数

### 参数

- `coroutine`: 异步函数

### 返回值

- `Any` 异步函数的返回值

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId


app = Dingtalk()
app.prepare()

res = app.run_coroutine(app.semd_message(OpenConversationId('cid1145149191810'), 'hello'))
```

## `with_access_token`

在特定上下文中使用特定 access_token

### 注意事项

- 该 API 仅支持在特定上下文中使用特定 access_token，不支持全局使用
- 似乎在控制台模式下存在无法切换回原有 access_token 的问题

### 参数

- `access_token`: `AccessToken` 对象

### 返回值

- `ContextManager` 特定上下文管理器

### 示例

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.access_token import AccessToken


app = Dingtalk()
app.prepare()

with app.with_access_token(AccessToken('your_access_token')):
    res = app.run_coroutine(app.semd_message(OpenConversationId('cid1145149191810'), 'hello'))
```

## `create_task`

创建异步任务

### 注意事项

该函数只能在框架还未开始进入消息等待时调用，否则无效

### 参数

- `coroutine`: 异步函数
- `name`: 任务名称，可选
- `show_info`: 是否显示任务信息，可选
- `not_cancel_at_the_exit`: 是否在退出时不取消任务，可选

### 返回值

- `tuple`: `Task`, `name`

### 示例

```python
from dingraia.lazy import *


@channel.use(ListenerSchema(listening_events=[LoadComplete]))
async def init(app: Dingtalk):
    app.create_task(check_power_fee(), "Check Power Fee")
```

## `stop`

停止框架

### 返回值

- `None`

## `coroutine_watcher`

监控异步函数的运行状态并在意外结束时自动重启

### 参数

- `function`: 异步函数
- `*args`: 异步函数参数
- `**kwargs`: 异步函数关键字参数


### 返回值

- `None`

## `get_info`

从缓存中读取对象的信息

### 参数

- `target`: 目标对象，可以是 `Member`, `Group`, `OpenConversationId`, `str`, `int`

### 返回值

- `tuple`: `dict` 目标对象信息, `int` 缓存时间戳

## `update_object`

从缓存中更新对象信息

### 参数

- `target`: 目标对象，可以是 `Member`, `Group`, `OpenConversationId` 或其他

### 返回值

- `target` （已更新）


# 未完成...



