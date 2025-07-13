# AI API

Dingraia 提供了多个AI服务的集成支持，包括:

- OpenAI API
- DeepSeek API 
- SiliconFlow API
- Groq API
- LMStudio API
- Ollama API

## 基础类

### aiAPI

所有AI服务的基类，提供了基础的消息管理和对话功能。

```python
class aiAPI:
    def __init__(self, systemPrompt: str = "You are a helpful assistant."):
        """
        初始化AI API
        
        Args:
            systemPrompt: 系统提示词
        """
        
    def messages(self, user: Union[Member, str] = None) -> list:
        """
        获取对话历史
        
        Args:
            user: 用户对象或标识符
            
        Returns:
            list: 对话历史列表
        """
        
    def clearHistory(self, user: Union[Member, str] = None):
        """
        清除对话历史
        
        Args:
            user: 用户对象或标识符
        """
        
    def deleteMessage(self, count: int = None, user: Union[Member, str] = None):
        """
        删除最近的n条消息
        
        Args:
            count: 要删除的消息数量
            user: 用户对象或标识符
        """
        
    def setSystemPrompt(self, prompt: str, user: Union[Member, str] = None, 
                       clearHistory: bool = False, bindPrompt: bool = False):
        """
        设置系统提示词
        
        Args:
            prompt: 新的系统提示词
            user: 用户对象或标识符
            clearHistory: 是否清除历史记录
            bindPrompt: 是否将提示词绑定到用户
        """
```

### APIKeys

API密钥管理类，支持多个API密钥的负载均衡。

```python
class APIKeys:
    def __init__(self, *keys: str, method: Literal["average", "balance"] = "average"):
        """
        初始化API密钥管理器
        
        Args:
            *keys: API密钥列表
            method: 负载均衡方法
                - average: 平均分配
                - balance: 根据余额分配(暂未实现)
        """
        
    def getKey(self) -> str:
        """获取当前可用的API密钥"""
        
    def addValue(self, key: str, add: int = 1):
        """
        增加密钥使用次数
        
        Args:
            key: API密钥
            add: 增加的次数
        """
```

## OpenAI API

基于OpenAI API的实现，支持流式输出和思考标记。

```python
class OpenAI(aiAPI):
    def __init__(self, apiKey: Union[str, APIKeys], 
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 2048,
                 baseUrl: str = "https://api.openai.com/v1"):
        """
        初始化OpenAI API客户端
        
        Args:
            apiKey: API密钥或密钥管理器
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
            baseUrl: API基础URL
        """
        
    async def getAvailableModels(self) -> dict[str, str]:
        """获取可用的模型列表"""
        
    def generateAnswerFunction(self, question: str, model: str = "", 
                             user: Member = None, noThinkOutput: bool = False) -> AsyncGenerator[str, Any]:
        """
        生成回答
        
        Args:
            question: 问题文本
            model: 使用的模型
            user: 用户对象
            noThinkOutput: 是否输出思考过程
            
        Returns:
            AsyncGenerator: 流式输出的回答
        """
```

## DeepSeek API

DeepSeek API的实现，继承自OpenAI API类。

```python
class DeepSeek(OpenAI):
    def __init__(self, apiKey: Union[str, APIKeys], 
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 1024):
        """
        初始化DeepSeek API客户端
        
        Args:
            apiKey: API密钥或密钥管理器
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
        """
        
    async def getAccountBalance(self) -> tuple[float, float, float, str]:
        """
        获取账户余额
        
        Returns:
            tuple: (总余额, 赠送余额, 充值余额, 货币单位)
        """
```

## SiliconFlow API

SiliconFlow API的实现，支持多个模型。

```python
class SiliconFlow(OpenAI):
    def __init__(self, apiKey: Union[APIKeys, str],
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 2048):
        """
        初始化SiliconFlow API客户端
        
        Args:
            apiKey: API密钥或密钥管理器
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
        """
        
    async def getAccountBalance(self, apiKey: str = None) -> tuple[float, float, float]:
        """
        获取账户余额
        
        Args:
            apiKey: 指定的API密钥
            
        Returns:
            tuple: (总余额, 当前余额, 充值余额)
        """
```

### 支持的模型

SiliconFlow支持多个开源大语言模型，包括:

- DeepSeek系列
  - DeepSeek-R1
  - DeepSeek-V2.5/V3
  - DeepSeek-R1-Distill系列(70B/32B/14B/8B/7B/1.5B)
- Qwen系列
  - Qwen2.5系列(72B/32B/14B/7B)
  - Qwen2系列(7B/1.5B)
  - Qwen2.5-Coder系列(32B/7B)
- Yi系列
  - Yi-1.5系列(34B/9B/6B)
- Llama系列
  - Llama-3.3-70B-Instruct
  - Meta-Llama-3.1系列(405B/70B/8B)
- 其他模型
  - Gemma系列(27B/9B)
  - GLM系列
  - InternLM系列
  - Marco-o1等

## Groq API

Groq API的实现，支持流式输出。

```python
class Groq(aiAPI):
    def __init__(self, apiKey: str,
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 1000):
        """
        初始化Groq API客户端
        
        Args:
            apiKey: API密钥
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
        """
        
    async def getAvailableModels(self) -> dict[str, str]:
        """获取可用的模型列表"""
```

## LMStudio API

本地LMStudio API的实现。

```python
class LMStudio(aiAPI):
    def __init__(self, url: str = "http://localhost:1234/v1",
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 4096):
        """
        初始化LMStudio API客户端
        
        Args:
            url: API地址
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
        """
```

## Ollama API

本地Ollama API的实现。

```python
class Ollama(aiAPI):
    def __init__(self, url: str = "http://localhost:11434",
                 systemPrompt: str = "You are a helpful assistant.",
                 maxContextLength: int = 1000):
        """
        初始化Ollama API客户端
        
        Args:
            url: API地址
            systemPrompt: 系统提示词
            maxContextLength: 最大上下文长度
        """
```

### 支持的模型

Ollama支持多个开源模型，包括:

- DeepSeek-R1系列(1.5B/7B/8B/14B/32B/70B/671B)
- Llama-3.2等

## 使用示例

### 基本使用

```python
from dingraia.aiAPI import OpenAI

# 初始化API客户端
api = OpenAI(apiKey="your-api-key")

# 生成回答
async for chunk in api.generateAnswerFunction("你好"):
    print(chunk, end="")
```

### 多密钥负载均衡

```python
from dingraia.aiAPI import APIKeys, SiliconFlow

# 初始化密钥管理器
keys = APIKeys("key1", "key2", "key3", method="average")

# 使用多密钥的API客户端
api = SiliconFlow(apiKey=keys)
```

### 用户绑定提示词

```python
# 为特定用户设置专属提示词
api.setSystemPrompt(
    "你是一个专业的客服助手",
    user=member,
    bindPrompt=True
)
```

### 思考输出

AI回答中可以使用`<think>`标签来标记思考过程:

```
<think>
让我思考一下这个问题...
1. 首先...
2. 其次...
</think>
好的,我的回答是...
```

思考内容会以引用格式显示。 