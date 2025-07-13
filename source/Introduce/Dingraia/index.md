# Dingraia

Dingraia 是一个简单易用的框架,用于快速开发基于 Python 的钉钉机器人应用。

## 主要特性

- 支持多种回调方式
  - HTTP 回调
  - Stream 回调
  - 支持同时运行多种回调方式
- 丰富的消息类型支持
  - 文本消息
  - Markdown消息
  - 卡片消息(ActionCard)
  - 信息流卡片(FeedCard)
  - 文件消息(普通文件、图片、音频、视频)
  - 互动卡片
  - AI流式卡片
- 完整的群组管理功能
  - 创建/解散群组
  - 群成员管理
  - 群设置管理
  - 消息管理(发送/撤回)
- 企业组织管理
  - 部门管理
  - 用户管理
  - 权限管理
- 智能缓存系统
  - 文件上传缓存
  - 用户信息缓存
  - 群组信息缓存
- 开发者友好
  - 完整的类型提示
  - 丰富的文档
  - 详细的错误提示
  - Debug模式支持

## 快速开始

1. 安装

```bash
pip install dingraia
```

2. 基本配置

```python
from dingraia.config import Config, Bot
from dingraia.DingTalk import Dingtalk

# 创建配置
config = Config(
    bot=Bot(
        appKey='YOUR_APP_KEY',
        appSecret='YOUR_APP_SECRET',
        robotCode='YOUR_ROBOT_CODE'  # 可选
    )
)

# 初始化应用
app = Dingtalk(config=config)
```

3. 发送消息

```python
from dingraia.message.chain import MessageChain
from dingraia.element import OpenConversationId

# 发送文本消息
await app.send_message(
    OpenConversationId("conversation_id"),
    MessageChain("Hello World!")
)
```

## 运行模式

Dingraia 支持多种运行模式:

1. 脚本模式
   - 直接在Python脚本中使用
   - 适合简单的自动化任务

2. 控制台模式
   - 使用 `python -m dingraia` 启动
   - 自动加载配置和模块
   - 适合开发和调试

3. HTTP服务模式
   - 启动HTTP服务器接收回调
   - 适合生产环境部署

4. Stream模式
   - 使用WebSocket保持长连接
   - 实时接收消息,低延迟
   - 支持流式消息

## 配置说明

Dingraia 使用分层的配置系统:

1. Bot配置
   - appKey: 应用的唯一标识
   - appSecret: 应用的密钥
   - robotCode: 机器人的标识码(可选)

2. 回调配置
   - HTTP回调配置
   - Stream回调配置
   - 可同时启用多种回调方式

3. 缓存配置
   - 是否启用缓存
   - 缓存过期时间
   - 缓存类型选择

4. 调试配置
   - 是否启用Debug模式
   - API错误处理方式
   - 日志级别设置

## 更多资源

- [完整文档](https://dingraia.readthedocs.io/)
- [GitHub仓库](https://github.com/MeiHuaGuangShuo/Dingraia)
- [问题反馈](https://github.com/MeiHuaGuangShuo/Dingraia/issues)