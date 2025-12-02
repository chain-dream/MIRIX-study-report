# MIRIX-study-report
MIRIX 基本介绍
MIRIX 是为付诸于基于大型语言模型（LLM, Large Language Model）代理系统而设计的“记忆系统”框架，其核心目标包括：
使代理能够跨会话、长期保留用户／系统交互、环境事件、程序操作等信息。 
 支持多模态输入（不仅文本，还包括屏幕截图、图像、语音等）而不仅限于传统文本。 
 通过结构化记忆组件（而非“平板”文本堆砌）提升检索效果、响应相关性与存储效率。  

架构概要
MIRIX 的识别／处理机制可以从两个维度来看：记忆组件结构 + 检索／响应流程。

关于记忆：
Core Memory	存储关键的用户／代理“事实”：如用户姓名、喜好、代理人格语气等。  
Episodic Memory	存储时间戳化的“事件／交互”记录，比如“用户上次做了什么操作”“某次对话结果” 等。  
Semantic Memory	存储抽象知识、命名实体、概念关系 —— 类似“知识库”但与用户／代理互动关联更强。  
Procedural Memory	存储流程／任务操作的步骤、脚本或者“如何做”的结构化信息。  
Resource Memory	资源型记忆：文档、图片、音频、链接这些“外部资料”或媒介形式的信息。  
Knowledge Vault	用于保存敏感／精确的事实信息，如账号、凭证、联系方式等，需要严格控制访问。
响应流程：
1.	输入处理：用户（或系统）输入可能是文本、图像、屏幕截图、语音等。系统首先进行解析／分类。  
2.	主题推断：系统（由 Meta Memory Manager）推断用户当前的“主题”或“意图”——即从用户输入中抽取一个检索议题。  
3.	多模块检索：基于该主题，系统从上述六类记忆模块中检索相关条目。检索机制包括例如 embedding 匹配、BM25全文检索、字符串匹配等。  
4.	标签／源注释：系统将检索出的条目按其来源模块打标签，以便在生成响应时模型能够知道这些信息“来自何处”。  
5.	构建提示／生成响应：将检索结果注入到提示（prompt）中，与基础语言模型一起生成对用户输入的响应／输出。
6.	记忆更新：如果用户输入包含新的信息（如新的偏好、新事件、截图内容等），系统会将新信息路由到合适的记忆模块并进行存储／更新。


这种流程让系统不仅“回答当前问题”，还同时“识别”用户上下文、更新记忆、并在后续交互中使用这些记忆，从而提升连贯性和个性化。
关于相应的使用体验：


<img width="468" height="131" alt="image" src="https://github.com/user-attachments/assets/071bdf5d-13d4-4dae-9652-fb6ec134af8b" />



如果截屏时长超过2小时（如果屏幕内容包含视频则更短）会导致相应流程无法反应，或者响应时间过长。
例如，开启MIRIX长达2小时进行网页浏览，最后让
mirix总结浏览内容，这时候模型就会完全无响应，
但是如果时间在30min-60min之间模型响应会非常迅速并且捕捉的细节量也较大。


 

恰当使用实例：在使用时间恰当的时候即使是92页的内容也能完成总结。
 不当使用实例：
 <img width="468" height="212" alt="image" src="https://github.com/user-attachments/assets/ad43acab-9313-4f66-bd99-cfb9bae881d8" />

如果时长超过2小时，那么模型就只能做到单一重复的任务，例如记录下用户在浏览什么网页，但是细节会大量丢失。



总结：模型的记忆能力很出色，但是响应能力不是很强，记忆可以保存大量的信息，及时关键细节丢失。


 <img width="468" height="213" alt="image" src="https://github.com/user-attachments/assets/09554ab4-7b44-4e69-913c-fcd787959ca8" />
关于模型的理解能力：模型的理解能力欠佳，他有时不能很好的理解用户下达的命令，或者命令不够精确就无法执行。 




 
<img width="468" height="85" alt="image" src="https://github.com/user-attachments/assets/b8ba4f6b-2da4-4414-8eb3-5dd6e8aba0fc" />

上文中提到的内容模型不会进行联想。


关键信息：
模型对于关键信息的拓展还是非常强力的，例如我使用模型浏览了很长时间的某主题内容但是模型可以把其中某些关键信息提取出来并扩展。（只能几个当中出现1-3个丢失较大） 

<img width="468" height="162" alt="image" src="https://github.com/user-attachments/assets/68faa393-891c-4f3a-b0f7-ef473db27078"/>

模型“识别”的关键机制

从“识别”角度来看，MIRIX 执行以下关键操作，使其能够“理解”／“记住”／“调用”信息：
•	主题识别（Topic Inference）：将用户输入转化成一个或多个“检索议题”，这是识别流程的第一步。系统需要理解用户在问什么／想做什么，然后把它映射到适合的记忆模块。
•	多模态解析：系统支持非文本输入（如屏幕截图、图像、音频），因此需要先将这些转换为可识别的表征（例如图像转文本、提取关键视觉元素）后才进行记忆存储／检索。  
•	嵌入 + 检索索引机制：系统在后台可能对记忆条目做向量化（embeddings）或者全文索引，以便快速匹配用户意图与历史记忆。比如使用 embedding 匹配或 BM25。  
•	结构化记忆存储：因为不同模块有不同的字段（如事件类别、时间戳、资源类型等），系统可以更精准地存储和检索，而非把所有内容模糊地当作“文本片段”存放。
•	上下文注入与提示工程：当生成回应时，系统会把检索到的相关记忆“注入”到 prompt 中，帮助基础语言模型（如 Google Gemini、OpenAI GPT 等）基于更丰富的上下文来生成回复。
•	隐私与本地存储：MIRIX 特别强调用户记忆数据的隐私控制与本地存储选项，这意味着系统在“识别”阶段还需要判断哪些数据属于敏感／需特别保护。



https://github.com/Mirix-AI/MIRIX?utm_source=chatgpt.com

https://github.com/DanielViberg/ActivityLogger/blob/main/start.sh




关于ChatGPT 的API:


 MIRIX 本地运行与 API 配置完整踩坑日志

本文总结了 2025/11/16—11/17 期间本地运行 MIRIX 过程中的所有错误、原因、以及最终解决方案，并附上 MIRIX 如何调用 OpenAI/Gemini API 的正确方法，以便后续维护或提交项目记录。

1. 开发环境概要
•	macOS（Apple Silicon）
•	Python 3.11（通过 brew 安装）
•	venv 虚拟环境
•	MIRIX 本地源代码版本（GitHub 克隆）
•	使用 .env 导入 API key

2. 遇到的主要错误与原因

错误 1：ModuleNotFoundError: No module named ‘demjson3’

原因： 初始系统 Python（/usr/bin/python3）依赖旧版 pip 环境，未安装 MIRIX 所需依赖。

解决：
•	使用 brew 安装 Python 3.11
•	在项目目录创建虚拟环境：
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

错误 2：不要直接使用系统 Python，否则出现 urllib3 NotOpenSSLWarning
urllib3 v2 only supports OpenSSL 1.1.1+, currently compiled with LibreSSL 2.8.3
原因： macOS 自带 Python 使用 LibreSSL，不被 MIRIX/urllib3 支持。

解决： 统一使用 venv + brew python。

错误 3：RuntimeError: OPENAI_API_KEY 未设置

原因：

.env 文件存在，但 key 未被加载入当前 shell。

正确加载方式（适用于 zsh）：
set -a
source .env
set +a
检查：
echo $OPENAI_API_KEY

错误 4：Incorrect API key provided (401)

原因： .env 里填写的 API key 格式错误或已经被撤销。

在 OpenAI 控制台重新创建 key，并重新写入 .env：
OPENAI_API_KEY=sk-proj-xxxx...
重新加载 .env 后，错误消失。

错误 5：SQLite schema 版本错误
no such column: users.status
SQLite schema is invalid...
出现这一大段提示：
•	SQLite 不支持 schema migration，MIRIX 要求删除旧数据库。

解决：
rm ~/.mirix/sqlite.db
重启后数据库自动重新初始化。

错误 6：GEMINI_API_KEY not found（但这里不是致命错误）
Info: GEMINI_API_KEY not found. Gemini features will be available after API key is provided.
Initializing model: gemini-2.5-flash-lite
MIRIX 即使指定主模型为 Gemini，但内部 memory agent / chat agent 默认仍会尝试 OpenAI。

错误 7（最终）：429 insufficient_quota


错误内容：
You exceeded your current quota, please check your plan and billing details.
原因：

OpenAI 项目没有 API 使用额度：
•	可能是没有充值
•	可能是试用额度已用完
•	ChatGPT Plus 不会自动赠送 API 额度

解决方法：

选择一种即可：

方法 A：给当前项目充值

在 platform.openai.com → Billing 里充值几美元即可。

方法 B：换一个有额度的 key

把 .env 改为：
OPENAI_API_KEY=另外一个账号的key
方法 C：改用 Google Gemini（完全免费额度）

设置：
GEMINI_API_KEY=xxx
并需要修改 MIRIX 的配置文件，让它不用 OpenAI 的 memory/chat agent。

3. MIRIX 如何调用 OpenAI / Gemini（机制解析）

MIRIX 内部结构是 多代理系统，并不是只调用你选择的模型，而是：
•	meta_memory_agent
•	chat_agent
•	reflection_agent
•	embedding_agent

这些 agent 默认 全部基于 OpenAI。

即使主模型写的是 Gemini，只要 memory/chat agent 没改，MIRIX 仍然会调用 OpenAI → 导致看到的 429 重试风暴。

4. 正确设置 API key 的方式（最稳定做法）

.env
 推荐格式
OPENAI_API_KEY=sk-proj-xxxxxx
GEMINI_API_KEY=你的GeminiKey（可选）
加载方式（zsh 专用）
set -a
source .env
set +a

5. MIRIX 调用 OpenAI 的实际流程（核心总结）

MIRIX 内部调用流程：
1.	读取环境变量 → OPENAI_API_KEY / GEMINI_API_KEY
2.	加载 agent 配置（默认 provider=openai）
3.	chat_agent → 调用 gpt-4o / gpt-4o-mini
4.	meta_memory_agent → 也调用 OpenAI（处理记忆）
5.	embedding_agent → 调用 text-embedding-3-small
6.	server 尝试序列化与管理 memory 动作

因此：
只要任意一个 agent 失败（如没额度），整个 MIRIX 都会抛错误。

6. MIRIX 调用 OpenAI 的教程（简明）

第一步：写入 
.env
OPENAI_API_KEY=sk-proj-xxxxxx
第二步：加载 key
set -a
source .env
set +a
第三步：启动 MIRIX 测试脚本
python3 openai_test.py
如果看到：
Initializing model...
并且没有 401 或 429，即成功。

7. 今日踩坑完整总结


错误	类型	原因	是否解决
没有 demjson3	Python import	pip 环境错误	✔️
urllib3 LibreSSL	SSL	系统 Python 不支持	✔️
OPENAI_API_KEY 未设置	RuntimeError	.env 未加载	✔️
Incorrect API key provided (401)	OpenAIAuth	key 无效	✔️
SQLite schema 错误	DB schema	数据库版本不兼容	✔️
GEMINI_API_KEY not found	信息	可忽略	⚠️（提示）
429 insufficient_quota	API error	账号没有 API 额度	❗最终问题
 

MIRIX 本地运行过程的核心问题在于环境隔离、依赖兼容性，以及 OpenAI API key 的权限与配额配置。通过统一 Python 版本、正确使用 .env 加载密钥，并确保 OpenAI 项目具有有效额度，可以成功启动 MIRIX 的所有代理系统。


配置文件：
import os
from dotenv import load_dotenv 
load_dotenv()
from mirix import Mirix

def main():
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise RuntimeError("环境变量 OPENAI_API_KEY 未设置，请检查 .env 文件")

    agent = Mirix(
        api_key=api_key,
        model_provider="openai",
        model="gpt-4o"   # 可改成 gpt-4o-mini, gpt-4.1 等等
    )

    agent.add("I am doing a Microsoft data analysis internship.")
    agent.add("I am testing MIRIX with OpenAI models.")

    question = "What am I doing in this internship?"
    answer = agent.chat(question)

    print("Question:", question)
    print("Answer:", answer)

if __name__ == "__main__":
    main()


