# 卡片模块

卡片模块提供了钉钉卡片消息的功能，包括AI卡片和私有数据构建器。

## 基础类

### BaseCard

卡片基类。

```python
class BaseCard:
    data: Optional[dict[str, str]] = None
    
    def __init__(self):
        pass
```

### MarkDown

Markdown字符串类。

```python
class MarkDown(str):
    pass
```

## 私有数据构建器

### PrivateDataBuilder

私有数据构建器，用于构建卡片的私有数据。

```python
class PrivateDataBuilder:
    def __init__(self, data: dict[str, dict]):
        """
        初始化私有数据构建器
        
        Args:
            data: 私有数据字典
        """
```

#### 方法

- `to_json()`: 转换为JSON格式
- `solve()`: 解析数据

#### 使用示例
```python
from dingraia.card import PrivateDataBuilder

# 创建私有数据
private_data = PrivateDataBuilder({
    "user1": {"name": "张三", "age": 25},
    "user2": {"name": "李四", "age": 30}
})

# 转换为JSON格式
json_data = private_data.to_json()
# 结果:
# {
#     "user1": {"cardParamMap": {"name": "张三", "age": 25}},
#     "user2": {"cardParamMap": {"name": "李四", "age": 30}}
# }
```

## AI卡片

### AICard

AI卡片类，用于发送AI流式卡片消息。

```python
class AICard(BaseCard):
    response: Optional[Union[SizedIterable[str], list]] = None
    """流式回复内容，必须是可以被迭代的"""
    
    content_type: Literal["auto", "full", "stream"] = None
    """流式回复内容的类型，默认为auto自动判断"""
    
    _texts: List[str] = []
    """流式回复内容的列表"""
    
    text: str = ""
    """最新的完整文本"""
    
    def __init__(self):
        super().__init__()
        self.content_type: str = "auto"
```

#### 方法

##### set_response

设置流式回复内容。

```python
def set_response(
    self,
    response: Union[SizedIterable[Union[str, MarkDown]], Generator, AsyncGenerator],
    content_type: Literal["auto", "full", "stream"] = "auto"
):
    """
    设置流式回复内容
    
    Args:
        response: 回复内容，可以是字符串、Generator、AsyncGenerator
        content_type: response输出内容的方式，默认为auto自动判断
            - auto: 自动判断
            - stream: 每次追加
            - full: 每次全量
    """
```

##### streaming_string

流式输出字符串。

```python
async def streaming_string(self, length_limit: int = 0, full_content: bool = True):
    """
    流式输出
    
    Args:
        length_limit: 最小输出长度，默认为0，即不限制
        full_content: 是否输出完整文本，默认为True
        
    Yields:
        str: 流式输出的字符串
    """
```

##### completed_string

获取完整的字符串。

```python
async def completed_string(self) -> str:
    """
    获取完整的字符串
    
    Returns:
        str: 完整的字符串
    """
```

##### withPostUrl

从POST请求获取流式回复。

```python
def withPostUrl(
    self, 
    post_url: str, 
    json: dict, 
    data_handler: Callable[[dict], Optional[str]], 
    headers: dict = None,
    timeout: Optional[float] = None
):
    """
    从POST请求获取流式回复
    
    Args:
        post_url: POST请求URL
        json: 请求体
        data_handler: 数据处理函数
        headers: 请求头
        timeout: 超时时间
    """
```

#### 属性

- `data`: 卡片数据，返回 `{"content": self.text}`

#### 使用示例

##### 基本使用

```python
from dingraia.card import AICard, MarkDown

# 创建AI卡片
card = AICard()

# 设置流式回复内容
async def generate_response():
    yield "你好"
    yield "，我是"
    yield "AI助手"

card.set_response(generate_response())

# 流式输出
async for text in card.streaming_string():
    print(text)
```

##### 使用字符串列表

```python
from dingraia.card import AICard

card = AICard()

# 设置字符串列表
card.set_response(["你好", "，我是", "AI助手"], content_type="stream")

# 获取完整字符串
full_text = await card.completed_string()
print(full_text)  # 输出: 你好，我是AI助手
```

##### 从POST请求获取回复

```python
from dingraia.card import AICard

card = AICard()

# 定义数据处理函数
def data_handler(data: dict) -> str:
    return data.get("content", "")

# 从POST请求获取流式回复
card.withPostUrl(
    post_url="https://api.example.com/chat",
    json={"message": "你好"},
    data_handler=data_handler,
    headers={"Authorization": "Bearer token"},
    timeout=30.0
)
```

##### 发送AI卡片消息

```python
from dingraia.DingTalk import Dingtalk
from dingraia.element import OpenConversationId
from dingraia.card import AICard

app = Dingtalk()
app.prepare()

# 创建AI卡片
card = AICard()

# 设置流式回复
async def generate_response():
    for i in range(10):
        yield f"消息 {i}\n"

card.set_response(generate_response())

# 发送AI卡片
target = OpenConversationId("cid123456")
template_id = "your_template_id"

await app.send_ai_card(
    target=target,
    cardTemplateId=template_id,
    card=card,
    update_limit=0.5,  # 更新频率限制（秒）
    key="content"  # 卡片数据在模板中的键名
)
```

## 内容类型

### auto

自动判断内容类型：
- 如果内容是累积的（每次包含之前的内容），则使用 `full` 模式
- 如果内容是增量的（每次只包含新增内容），则使用 `stream` 模式

### full

全量模式：每次更新都发送完整的内容。

适用于：
- AI模型返回完整响应
- 内容更新频率较低

### stream

流式模式：每次更新只发送新增的内容。

适用于：
- AI模型流式返回响应
- 打字机效果
- 实时更新

## 最佳实践

1. **选择合适的content_type**
   - 如果不确定，使用 `auto`
   - 如果AI模型返回完整响应，使用 `full`
   - 如果AI模型流式返回响应，使用 `stream`

2. **设置合理的更新频率**
   - 使用 `update_limit` 参数控制更新频率
   - 避免过于频繁的更新导致性能问题

3. **处理异常情况**
   - 捕获流式处理中的异常
   - 设置合理的超时时间
   - 提供降级方案

4. **优化用户体验**
   - 使用 `streaming_string` 的 `length_limit` 参数
   - 避免过于频繁的更新
   - 提供加载状态提示