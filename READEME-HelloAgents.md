# READEME

Welcome to your first markdown file in this repository!

You can use this file to introduce your project, write documentation, or share notes. Markdown helps format your text with headings, lists, code snippets, and more.

## Example Markdown

- Bullet list item
- **Bold text**
- [A link](https://guides.github.com/features/mastering-markdown/)

You can always edit this file as your project grows.

# 第一章-初识智能体
## 练习题解答

- 练习题1

- 练习题2

- 练习题3

- 练习题4

- 练习题5

- 练习题6

# 第二章-智能体发展史-个人思考和记录
## 符号主义
- 智能体智慧： 基于符号和逻辑的智能体的智慧来自于设计者自身的预先编码的知识库和逻辑推理规则，**智能的本质**，就是符号的计算与处理，
- 基于符号主义的应用：**专家系统**就是符号主义时代最重要最成功的系统应用成果
- 专家系统组成部分：专家系统通常由**知识库、推理机、用户界面**等几个核心部分构成

- 知识库和推理机功能：
  **知识库**（Knowledge Base）：右一系列IF-ELSE-THEN的产生式规则等形式进行知识表示的领域专家的知识和经验的存放系统，即为知识存储系统
  **推理机**：
  ### 基于规则的聊天机器人
- 核心痛点问题：缺乏真正理解上下文，系统脆弱性在于只是机械的执行规定的逻辑和规则，没有记忆功能，
## 心智社会
- 解决的问题：不再将心智视为一个金字塔式的层级结构，而是将其看作一个扁平化的、充满了互动与协作的“社会”，探索作为协作体的智能体
- 启发和意义：为多智能体的分布式智能体带来启发，**去中心化控制；涌现式计算；智能体的社会性**
## 基于强化学习的智能体
- 联结主义：三个核心思想分别是**知识的分布式表示、简单的处理单元、通过学习调整权重**，解决了感知和学习知识的问题
- 强化学习：联结主义主要解决了感知问题（例如，“这张图片里有什么？”），但智能体更核心的任务是进行决策（例如，“在这种情况下，我应该做什么？”）。强化学习（Reinforcement Learning, RL）正是  专注于解决序贯决策问题的学习范式。**试错中成长**，解决了任务决策的问题
- 强化学习框架的核心要素：智能体、环境、状态、行动、奖惩
- 预训练和微调范式图
- 预训练：**海量语料库数据集，自监督学习**，目标是学习通用知识，不是为某特定任务，最终**学习到了和数据集有关的丰富知识**
- 微调：**少量特定任务的标注数据**对模型进行微调
  <img width="1623" height="1053" alt="image" src="https://github.com/user-attachments/assets/5e2d60d1-9e0a-4b34-9144-56533f518713" />
- LLM的涌现能力：上下文学习和COT思维链推理的两个能力让LLM成为了兼具海量知识库和通用推理引擎双重角色的组件。
## 基于LLM为核心的智能体
-框架图
<img width="2067" height="873" alt="image" src="https://github.com/user-attachments/assets/7dad107c-ed69-4678-82ef-38de8ed8f1d7" />

# 第四章-智能体经典范式构建

当前代码实现的工具无法查询到最新的相关信息，因为LLM的训练数据集保留在2025年，工具集中也缺少2026年查询的相关工具，可以开发日期查询工具，优化prompt，让大模型可以查询到工具集中的日期查询部分工具，每次进行问题思考时，可以搜索到最新部分

## 添加新工具-日期查询工具
优化tools.py相关代码
 
 ```python
from datetime import datetime
now = datetime.now()
current_date = now.strftime("%Y年%m月%d日")

REACT_PROMPT = f"""
当前日期是：{current_date}。
你是一个具备推理和行动能力的AI助手……
```
如下部分是llm_client的核心代码部分的解释，可以作为其他程序的参考
```python
class HelloAgentsLLM:
    """
    为本书 "Hello Agents" 定制的LLM客户端。
    它用于调用任何兼容OpenAI接口的服务，并默认使用流式响应。
    """

    def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None):
        """
        初始化客户端。优先使用传入参数，如果未提供，则从环境变量加载。
        """
        self.model = model or os.getenv("LLM_MODEL_ID")
        apiKey = apiKey or os.getenv("LLM_API_KEY")
        baseUrl = baseUrl or os.getenv("LLM_BASE_URL")
        timeout = timeout or int(os.getenv("LLM_TIMEOUT", 60))

        if not all([self.model, apiKey, baseUrl]):
            raise ValueError("模型ID、API密钥和服务地址必须被提供或在.env文件中定义。")

        self.client = OpenAI(api_key=apiKey, base_url=baseUrl, timeout=timeout)

    def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str:
        """
        调用大语言模型进行思考，并返回其响应。
        """
        print(f"🧠 正在调用 {self.model} 模型...")
        try:
            # 调用大模型对话接口，获取流式响应对象
            response = self.client.chat.completions.create(
                model=self.model,  # 指定本次调用的大模型名称
                messages=messages,  # 传入历史对话与用户提问消息列表
                temperature=temperature,  # 设置生成文本随机度，数值越高创意性越强
                stream=True,  # 开启流式逐段返回响应模式
            )

            # 处理流式响应
            print("✅ 大语言模型响应成功:")
            collected_content = []  # 定义列表，存储分段返回的文本内容
            # 遍历流式迭代器，逐段接收模型返回数据
            for chunk in response:
                # 当前分片无有效回答内容则跳过处理
                if not chunk.choices:
                    continue
                # 提取当前分片文本，为空时赋值空字符串避免报错
                content = chunk.choices[0].delta.content or ""
                # 无换行实时打印文本，强制刷新输出缓冲区
                print(content, end="", flush=True)
                # 将分段文本存入列表，用于后续拼接完整内容
                collected_content.append(content)
            print()  # 在流式输出结束后换行
            # 拼接所有分段内容，返回完整回答文本
            return "".join(collected_content)

        except Exception as e:
            print(f"❌ 调用LLM API时发生错误: {e}")
            return None
```

