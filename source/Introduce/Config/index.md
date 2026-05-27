# 配置说明

Dingraia 使用分层的配置系统，主要通过 `Config` 类进行管理。

## 基础配置

### Bot 配置

机器人的基本配置，包含应用的密钥信息。

```python
from dingraia.config import Bot

bot = Bot(
    AppKey='YOUR_APP_KEY',          # 应用的唯一标识
    AppSecret='YOUR_APP_SECRET',    # 应用的密钥
    robotCode='YOUR_ROBOT_CODE',    # 机器人编码（可选）
    GroupWebhookAccessToken=None,   # 群组Webhook访问令牌（可选）
    GroupWebhookSecureKey=None      # 群组Webhook安全密钥（可选）
)
```

### CallBack 配置

HTTP回调的配置，用于接收钉钉的事件推送。

```python
from dingraia.config import CallBack

callback = CallBack(
    AesKey='YOUR_AES_KEY',     # 加密密钥
    Token='YOUR_TOKEN',        # 验证令牌
    appKey='YOUR_APP_KEY'      # 应用标识
)
```

### Stream 配置

Stream模式的配置，用于WebSocket长连接。

```python
from dingraia.config import Stream

stream = Stream(
    appKey='YOUR_APP_KEY',      # 应用标识
    appSecret='YOUR_APP_SECRET' # 应用密钥
)
```

## 完整配置示例

```python
from dingraia.config import Config, Bot, CallBack, Stream
from dingraia.DingTalk import Dingtalk

# 创建配置
config = Config(
    bot=Bot(
        AppKey='YOUR_APP_KEY',
        AppSecret='YOUR_APP_SECRET',
        robotCode='YOUR_ROBOT_CODE'
    ),
    event_callback=CallBack(
        AesKey='YOUR_AES_KEY',
        Token='YOUR_TOKEN',
        appKey='YOUR_APP_KEY'
    ),
    stream=[
        Stream(
            appKey='YOUR_APP_KEY',
            appSecret='YOUR_APP_SECRET'
        )
    ],
    autoBotConfig=True,           # 自动配置机器人
    useDatabase=True,             # 使用数据库缓存
    raiseForApiError=True,        # API错误时抛出异常
)

# 初始化应用
app = Dingtalk(config=config)
```

## 配置参数说明

### Config 类参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `bot` | `Bot` | `None` | 机器人配置 |
| `event_callback` | `CallBack` | `None` | HTTP回调配置 |
| `stream` | `List[Stream]` | `None` | Stream配置列表 |
| `autoBotConfig` | `bool` | `True` | 是否自动配置机器人 |
| `useDatabase` | `bool` | `True` | 是否使用数据库缓存 |
| `raiseForApiError` | `bool` | `True` | API错误时是否抛出异常 |
| `customStreamConnect` | `CustomStreamConnect` | `None` | 自定义Stream连接 |
| `webRequestHandlers` | `List[Middleware]` | `None` | 自定义Web请求处理器 |
| `language` | `str` | `None` | 语言设置 |

### Bot 类参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `AppKey` | `str` | 必填 | 应用的唯一标识 |
| `AppSecret` | `str` | 必填 | 应用的密钥 |
| `robotCode` | `str` | `None` | 机器人编码 |
| `GroupWebhookAccessToken` | `str` | `None` | 群组Webhook访问令牌 |
| `GroupWebhookSecureKey` | `str` | `None` | 群组Webhook安全密钥 |

### Stream 类参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `appKey` | `str` | 必填 | 应用标识 |
| `appSecret` | `str` | 必填 | 应用密钥 |

## 高级配置

### 自定义Stream连接

当需要连接到自定义的Stream服务时使用。

```python
from dingraia.config import CustomStreamConnect

custom_stream = CustomStreamConnect(
    StreamUrl='wss://your-stream-server.com',  # WebSocket地址
    SignHandler=None,                          # 签名处理器
    ExtraHeaders={'Authorization': 'Bearer token'}  # 额外请求头
)
```

### 数据缓存配置

```python
from dingraia.config import DataCacheTime

cache_config = DataCacheTime(
    dataCacheTime=3600,                    # 默认缓存时间（秒）
    userInfoCacheTime=1800,                # 用户信息缓存时间
    groupInfoCacheTime=1800,               # 群组信息缓存时间
    userUnionIdConventCacheTime=410281690  # UnionId转换缓存时间
)
```

### 发送失败配置

```python
from dingraia.config import FailedMessage
from dingraia.message.chain import MessageChain

failed_config = FailedMessage(
    onNSFWMessage=MessageChain("检测到不当内容，消息已屏蔽")  # NSFW消息提示
)
```

## 配置最佳实践

1. **安全性**
   - 不要将密钥直接写在代码中
   - 使用环境变量或配置文件管理敏感信息
   - 定期轮换密钥

2. **性能优化**
   - 启用数据库缓存减少API调用
   - 合理设置缓存时间
   - 根据需求选择回调方式

3. **错误处理**
   - 建议开启 `raiseForApiError` 以便调试
   - 实现适当的错误处理逻辑
   - 记录关键操作的日志