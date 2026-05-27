# 消息元素模块

消息元素模块提供了钉钉消息中使用的各种元素类。

## 基础文件类

### File

文件基类，用于处理文件类型的元素。

```python
class File:
    def __init__(self, file: Union[Path, BinaryIO, bytes, str] = None, fileName: str = None):
        """
        初始化文件对象
        
        Args:
            file: 文件路径、二进制流或字节数据
            fileName: 文件名
        """
```

#### 属性
- `file`: 文件内容
- `size`: 文件大小
- `fileType`: 文件类型
- `mediaId`: 媒体ID（上传后获得）
- `downloadCode`: 下载码
- `fileName`: 文件名

#### 使用示例
```python
from dingraia.message.element import File

# 从文件路径创建
file = File("path/to/file.txt")

# 从二进制流创建
with open("file.txt", "rb") as f:
    file = File(f)
```

### Image

图片类，继承自File。

```python
class Image(File):
    def __init__(self, file: Union[Path, BinaryIO, bytes, str] = None, fileName: str = None):
        """
        初始化图片对象
        
        Args:
            file: 图片文件路径、二进制流或字节数据
            fileName: 文件名
        """
```

#### 属性
- `fileType`: 固定为 'image'

#### 使用示例
```python
from dingraia.message.element import Image

# 从文件路径创建
image = Image("path/to/image.png")

# 从网络URL创建
image = Image("https://example.com/image.png")
```

### Audio

音频类，继承自File。

```python
class Audio(File):
    def __init__(self, file: Union[Path, BinaryIO, bytes, str] = None, fileName: str = None):
        """
        初始化音频对象
        
        Args:
            file: 音频文件路径、二进制流或字节数据
            fileName: 文件名
        """
```

#### 属性
- `fileType`: 固定为 'voice'
- `duration`: 音频时长（秒）

#### 使用示例
```python
from dingraia.message.element import Audio

# 从文件路径创建
audio = Audio("path/to/audio.mp3")
```

### Video

视频类，继承自File。

```python
class Video(File):
    def __init__(
        self, 
        file: Union[Path, BinaryIO, bytes, str] = None, 
        coverPicture: Union[Image, str] = None,
        fileName: str = None
    ):
        """
        初始化视频对象
        
        Args:
            file: 视频文件路径、二进制流或字节数据
            coverPicture: 视频封面图片
            fileName: 文件名
        """
```

#### 属性
- `fileType`: 固定为 'video'
- `videoType`: 视频类型，默认 'mp4'
- `duration`: 视频时长（秒）
- `picMediaId`: 封面图片媒体ID
- `height`: 视频高度
- `width`: 视频宽度

#### 使用示例
```python
from dingraia.message.element import Video, Image

# 从文件路径创建
video = Video("path/to/video.mp4")

# 带封面图创建
cover = Image("path/to/cover.png")
video = Video("path/to/video.mp4", coverPicture=cover)
```

## 消息元素类

### Link

链接消息元素。

```python
class Link(BaseElement):
    def __init__(self, url: str, title: str = "[Link]", text: str = "", pic_url: str = ""):
        """
        初始化链接元素
        
        Args:
            url: 链接地址
            title: 链接标题
            text: 链接描述
            pic_url: 链接图片
        """
```

#### 使用示例
```python
from dingraia.message.element import Link

link = Link(
    url="https://example.com",
    title="示例网站",
    text="这是一个示例链接",
    pic_url="https://example.com/image.png"
)
```

### Markdown

Markdown消息元素。

```python
class Markdown(BaseElement):
    def __init__(self, text: str, title: str = "[Markdown]"):
        """
        初始化Markdown元素
        
        Args:
            text: Markdown文本内容
            title: 标题
        """
```

#### 使用示例
```python
from dingraia.message.element import Markdown

markdown = Markdown(
    text="## 标题\n\n这是一个**Markdown**消息",
    title="消息标题"
)
```

### ActionCard

ActionCard消息元素，支持单按钮和多按钮。

```python
class ActionCard(BaseElement):
    def __init__(
        self, 
        text: str, 
        buttons: List[List[str]] = None, 
        title: str = "[ActionCard]", 
        orientation: int = 0,
        button: list[list[str]] = None
    ):
        """
        初始化ActionCard元素
        
        Args:
            text: Markdown文本内容
            buttons: 按钮列表，格式：[[标题, URL], ...]
            title: 标题
            orientation: 排列方向，0为竖向，1为横向
        """
```

#### 按钮限制
- 竖向排列：最多5个按钮
- 横向排列：最多2个按钮

#### 使用示例
```python
from dingraia.message.element import ActionCard

# 单按钮
card = ActionCard(
    text="## 消息内容\n\n这是消息正文",
    buttons=[["查看详情", "https://example.com"]],
    title="消息标题"
)

# 多按钮（竖向）
card = ActionCard(
    text="## 消息内容\n\n请选择操作",
    buttons=[
        ["同意", "https://example.com/approve"],
        ["拒绝", "https://example.com/reject"]
    ],
    title="审批消息",
    orientation=0
)

# 多按钮（横向，最多2个）
card = ActionCard(
    text="## 消息内容",
    buttons=[
        ["同意", "https://example.com/approve"],
        ["拒绝", "https://example.com/reject"]
    ],
    title="审批消息",
    orientation=1
)
```

### ActionCardButton

ActionCard按钮辅助类。

```python
class ActionCardButton(list):
    def __init__(self, text: str, url: str):
        """
        初始化按钮
        
        Args:
            text: 按钮文本
            url: 点击链接
        """
```

#### 使用示例
```python
from dingraia.message.element import ActionCard, ActionCardButton

buttons = [
    ActionCardButton("同意", "https://example.com/approve"),
    ActionCardButton("拒绝", "https://example.com/reject")
]
card = ActionCard(text="消息内容", buttons=buttons)
```

### FeedCard

FeedCard消息元素，用于展示多条图文信息。

```python
class FeedCard(BaseElement):
    def __init__(self, links: Iterable[Iterable[str]]):
        """
        初始化FeedCard元素
        
        Args:
            links: 链接列表，格式：[[标题, URL, 图片URL], ...]
        """
```

#### 使用示例
```python
from dingraia.message.element import FeedCard

feed = FeedCard(links=[
    ["文章标题1", "https://example.com/1", "https://example.com/img1.png"],
    ["文章标题2", "https://example.com/2", "https://example.com/img2.png"]
])
```

### FeedCardNode

FeedCard节点辅助类。

```python
class FeedCardNode:
    def __init__(self, title: str, messageUrl: str, picUrl: str):
        """
        初始化FeedCard节点
        
        Args:
            title: 标题
            messageUrl: 消息链接
            picUrl: 图片链接
        """
```

#### 使用示例
```python
from dingraia.message.element import FeedCard, FeedCardNode

nodes = [
    FeedCardNode("标题1", "https://example.com/1", "https://example.com/img1.png"),
    FeedCardNode("标题2", "https://example.com/2", "https://example.com/img2.png")
]
feed = FeedCard(links=nodes)
```

### At

@提及元素。

```python
class At:
    def __init__(self, target: Union[tuple, Member], display: str = ""):
        """
        初始化@元素
        
        Args:
            target: @的目标，可以是Member对象或(staffId, name)元组
            display: 显示文本
        """
```

#### 使用示例
```python
from dingraia.message.element import At
from dingraia.model import Member

# @指定用户
member = Member()
member.staffId = "staff123"
at = At(member)

# @所有人
at = At("all")

# 使用元组
at = At(("staff123", "张三"))
```

## 消息链

### MessageChain

消息链类，用于组合多个消息元素。

```python
class MessageChain:
    def __init__(self, *elements):
        """
        初始化消息链
        
        Args:
            *elements: 消息元素列表
        """
```

#### 使用示例
```python
from dingraia.message.chain import MessageChain
from dingraia.message.element import At, Markdown, Image

# 纯文本消息
chain = MessageChain("Hello, World!")

# 包含@的消息
member = Member()
member.staffId = "staff123"
chain = MessageChain(At(member), " 你好！")

# 包含Markdown的消息
chain = MessageChain(Markdown("## 标题\n\n内容"))

# 包含图片的消息
chain = MessageChain(Image("path/to/image.png"))

# 组合多个元素
chain = MessageChain(
    At(member),
    " 请查看",
    Markdown("[链接](https://example.com)")
)
```

## 最佳实践

1. **文件处理**
   - 上传前检查文件大小限制
   - 使用对应的文件类型（Image、Audio、Video）
   - 优先使用文件路径而非二进制流

2. **消息组合**
   - 合理使用消息链组合多种元素
   - 注意消息长度限制
   - 避免过度使用@功能

3. **卡片设计**
   - ActionCard按钮数量不宜过多
   - FeedCard条目建议3-5个
   - 保持消息内容简洁明了