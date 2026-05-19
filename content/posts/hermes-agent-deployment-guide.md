---
title: "Hermes Agent 部署与使用指南：开源AI编程代理的完整实战 - AI知识宇宙 - 用AI采集，用AI生成"
date: 2026-05-17
category: "AI 工具实战"
description: "从零部署 Hermes Agent 开源 AI 编程代理的完整指南，涵盖安装、配置、多平台接入、技能系统与高级用法"
read_time: "4分钟"
tags: [AI 工具，AI 工具实战]
---

AI编程助手
                2026-05-17 · 12分钟
            
            
## 一、一句话认识 Hermes Agent


Hermes Agent 是 Nous Research 开源的一款 **自主 AI 代理框架**。它和 GitHub Copilot、Cursor 这类"补全代码"的工具不同——Hermes 是一个拥有完整系统访问权限的 AI 助手，可以在终端、聊天软件、IDE 中运行，帮你是执行代码、管理文件、爬取数据、定时任务等等。

> 


它本质上是一个**拥有"手"的 AI**——不仅能和你对话，还能实际操作你的电脑。




和同类开源产品如 Claude Code（Anthropic）、OpenAI Codex 相比，Hermes 最大的特点是 **Provider 无关**：它不绑定任何特定模型，你可以随时切换 GPT-4o、Claude Sonnet 4、DeepSeek 甚至本地模型，工具链和技能体系保持不变。

## 二、核心特性一览









****



****



****



****



****



****



****




| 特性说明 |
|---|
| Provider 无关支持 20+ 模型提供商，随时切换，无需迁移配置 |
| 技能系统踩过的坑、学会的流程自动保存为可复用的技能 |
| 持久记忆跨会话记住你的偏好、项目信息和工作习惯 |
| 多平台网关Telegram、Discord、微信、飞书……同一代理无缝切换 |
| 子代理委派复杂任务分解给多个子代理并行执行 |
| 定时任务cron 风格的任务调度，自动执行并投递结果 |
| 插件生态MCP 服务器、自定义工具、webhook 触发器 |



## 三、环境要求




































| 环境最低要求推荐配置 |
|---|
| 操作系统Linux / macOS / Windows WSL2Ubuntu 22.04+ 或 macOS 14+ |
| Python3.10+3.12+ |
| 内存4GB8GB+ |
| 磁盘1GB10GB+（存会话记录和技能） |
| 网络能访问 API 提供商稳定的海外网络连接 |




Hermes 本身不需要 GPU，推理完全依赖 API 调用。

## 四、安装部署（3种方式）

### 方式一：一键安装脚本（推荐）

```curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

```



安装完成后验证：

```hermes doctor

```



如果看到所有依赖通过，即可开始使用。

### 方式二：pip 安装

```pip install hermes-agent
hermes setup

```


### 方式三：源码安装（开发/贡献用）

```git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
pip install -e .
hermes setup

```


### 安装后的目录结构

```~/.hermes/
├── config.yaml          # 主配置文件
├── .env                 # API 密钥和敏感信息
├── skills/              # 技能库
├── sessions/            # 会话记录
└── logs/                # 网关和错误日志

```


## 五、配置你的第一个 Agent

### 5.1 选择模型提供商


打开初始配置向导：

```hermes setup

```



它会引导你完成：
1. 选择 LLM 提供商（OpenRouter / Anthropic / OpenAI / DeepSeek 等）
2. 指定默认模型
3. 配置终端后端（本地 / SSH / Docker）
4. 可选：消息网关平台


**推荐首次使用**选择 **OpenRouter**——一个平台接入数十种模型，方便对比。

### 5.2 配置 API 密钥

```hermes config edit

```



或者在 ~/.hermes/.env 中添加：

```OPENROUTER_API_KEY=sk-or-xxxxxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxxxxx

```


### 5.3 验证配置

```hermes doctor --fix

```



这条命令会检查所有依赖、密钥有效性、工具可用性，并尝试自动修复发现的问题。

## 六、基础使用：与 Agent 对话

### 交互模式


直接运行 hermes 进入交互式对话：

```hermes

```



你会看到一个命令行界面，在这里可以像聊天一样给 Hermes 下达任务：

```🧑 用户 > 帮我创建一个 Python 脚本，读取当前目录下所有 CSV 文件，合并并输出为 Excel
🤖 Hermes > 好的，我创建一个 merge_csv.py 文件...

```


### 单次查询模式


适合脚本集成或快速任务：

```hermes chat -q "列出 /home/user/projects 目录下最大的 5 个文件"

```


### 关键功能验证


Hermes 的核心能力在于**工具调用**——不只是聊天，而是真正执行操作：

```# 搜索文件内容
hermes chat -q "找到所有包含 FIXME 的 Python 文件"

# 网页搜索和提取
hermes chat -q "搜索最新的 Claude Sonnet 4 评测，总结要点"

# 定时任务
hermes chat -q "每天早上 9 点检查我的服务器磁盘空间并通知我"

```


## 七、进阶功能详解

### 7.1 技能系统：让 Agent 越用越聪明


这是 Hermes 区别于其他 AI 代理最核心的特性。当你在某个任务上踩了坑、找到了最佳实践，可以将其保存为**技能**——下次遇到类似问题，Hermes 会自动加载相关技能。


**保存一个技能：**


在一次成功的部署流程之后：

```🧑 用户 > 把刚才 Nginx 部署的过程保存为技能
🤖 Hermes > 🧠 技能 "nginx-deploy" 已创建。
下次你再提到 Nginx 部署时，我会自动加载它。

```



**手动创建技能：**


技能本质是 Markdown 文件，位于 ~/.hermes/skills/，内容包含 YAML 头信息和详细的步骤说明。也可以直接在会话中通过 /skill create 创建。


**浏览和搜索技能：**

```hermes skills list          # 列出所有技能
hermes skills search deploy # 搜索部署相关技能

```


> 


💡 **实用技巧**：每周运行一次 hermes curator run --dry-run，可以预览 curator（技能管家）如何帮你整理和合并冗余技能。



### 7.2 持久记忆


Hermes 能跨会话记住你的信息：

```hermes memory status  # 查看记忆状态

```



支持多种记忆后端：
- **内置 SQLite**（默认，零配置）
- **Mem0**（更智能的记忆管理）
- **Honcho**（分布式记忆）


会话中直接说"记住……"即可保存信息，下次对话会自动加载。

### 7.3 子代理委派


复杂任务可以拆解给多个子代理并发执行，它们彼此隔离、互不干扰：

```🧑 用户 > 帮我做一个项目：后端用 FastAPI 写用户 API，前端用 React 写管理面板
🤖 Hermes > 我将同时开启两个子代理并行工作：
  └─ Agent A: 后端 FastAPI 用户模块
  └─ Agent B: 前端 React 管理面板

```



完成后两个子代理的结果会自动汇总给你。

### 7.4 定时任务

```hermes cron create "0 9 * * *" \
  --prompt "每天收集 AI 领域最新新闻并保存" \
  --deliver telegram

```



支持 cron 表达式和自然语言（"every 2h"、"30m"），任务结果可以投递到 Telegram、Discord、邮箱等多个平台。

### 7.5 消息网关


Hermes 不只活在终端里。配置网关后，你可以用微信、Telegram、Discord 等平台与它交互：

```hermes gateway setup  # 交互式配置
hermes gateway start  # 启动网关

```



目前支持 15+ 平台，包括微信（Weixin）、飞书、Telegram、Discord、Slack、WhatsApp、Signal、电子邮件等。每个平台的工具权限可以独立配置——你在微信上只能问问题，在 Telegram 上可以执行文件操作，在终端里拥有全部权限。

## 八、真实场景实战

### 场景一：自动化内容采集

> 


需求：每天从 Product Hunt 和 Hacker News 抓取 AI 工具信息，汇总成日报



```hermes cron create "0 8 * * *" \
  --prompt "抓取 Product Hunt 今日热门 AI 产品和 Hacker News 上 AI 相关讨论，生成中英文双语的日报摘要并保存"

```



Hermes 会自动运行爬虫、整理数据、生成日报，并通过配置的渠道推送到你手上。

### 场景二：代码库维护

> 


需求：分析项目中的技术债务



```hermes chat -q "检查 /path/to/project 代码库：
1. 统计所有 Python 文件的行数和技术栈
2. 找出没有类型注解的函数
3. 找到 TODO/FIXME 标记
4. 检查测试覆盖率
5. 生成一份技术债务报告"

```


### 场景三：代码审查


在 git diff 之后，让 Hermes 帮你审查变更：

```git diff --cached | hermes chat -q "审查以下代码变更，关注：安全性问题、性能隐患、代码风格"

```


### 场景四：服务器监控

```hermes cron create "*/30 * * * *" \
  --prompt "检查服务器状态：
  - CPU 和内存使用率
  - 磁盘空间
  - 最近 100 条日志中的错误
  如果磁盘使用率 > 85% 则发送告警"

```


## 九、与同类工具对比












****






****






****






****






****






****






****






****






****







| 维度Hermes AgentClaude CodeOpenAI Codex CLICursor |
|---|
| 开源✅ 完全开源❌ 闭源❌ 闭源❌ 闭源 |
| Provider 无关✅ 20+ 提供商❌ 仅 Anthropic❌ 仅 OpenAI❌ 仅内置模型 |
| 技能系统✅ 自学习❌ 无❌ 无❌ 无 |
| 跨会话记忆✅ 持久记忆⚠️ 有限⚠️ 有限⚠️ 项目配置 |
| 多平台网关✅ 15+ 平台❌ 仅 CLI❌ 仅 CLI❌ 仅 IDE |
| 定时任务✅ 内置❌ 无❌ 无❌ 无 |
| 子代理并行✅ 原生支持⚠️ 实验性❌❌ |
| 上手难度★★★☆☆★★☆☆☆★★☆☆☆★☆☆☆☆ |
| 企业可扩展性★★★★★★★★☆☆★★★☆☆★★★☆☆ |




**Hermes 的核心优势**可以概括为三个词：**开放、自学习、全平台**。

## 十、最佳实践与注意事项

### ✅ 推荐做法


- **从小任务开始**：先让 Hermes 做文件操作、代码审查这类明确的任务，再上生产级复杂场景

- **利用技能系统**：每次踩坑都是技能素材，养成保存技能的习惯

- **配置文件备份**：config.yaml 和 .env 定期备份，重新部署只需这两个文件

- **使用 profile 隔离环境**：工作和个人场景用不同的 profile

- **善用 --yolo 参数**：在可信任的 CI 环境中用 --yolo 跳过二次确认




### ❌ 避免做法


- **不要给 Hermes 生产环境 root 权限**，除非你充分理解风险

- **不要在公共平台（如公司 Slack）上讨论敏感信息**——所有对话会记录在会话文件中

- **避免过于模糊的指令**，越具体的需求结果越好

- **不要在一个会话中让 Hermes 同时处理多个无关任务**，用子代理或分会话处理




### ⚠️ 常见问题






























| 问题解决方法 |
|---|
| hermes: command not found检查 venv 是否激活：source /path/to/venv/bin/activate |
| 工具不可用hermes tools list 检查启用状态，然后 /reset |
| 模型调用错误hermes doctor --fix 检查密钥和配置 |
| 网关无法启动检查 ~/.hermes/logs/gateway.log 查看错误详情 |
| 消息发不出去检查平台 token 是否过期，用 /platforms 查看连接状态 |



## 十一、总结


Hermes Agent 不是另一个"帮你写代码的 AI"，它是一个**完整的 AI 自动化基础设施**。它的价值不在于某个单一功能有多强，而在于：


- **Provider 无关**让你永远不锁定在任何模型上

- **技能系统**让 Agent 随着使用越来越了解你

- **多平台网关**让你在任何地方都能调用它的能力

- **开源和插件生态**意味着你可以深度定制它





如果你已经用惯了 ChatGPT 或 Claude 的 Web 界面，Hermes 会让你体验到 AI 与系统的深度融合是什么感觉——它不仅是聊天窗口里的答案机器，更是能在你的服务器、代码库、数据管道中真正干活的数字同事。

### 快速上手指南

```# 1. 安装
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 2. 配置
hermes setup

# 3. 开始使用
hermes

# 4. （可选）启动消息网关
hermes gateway setup
hermes gateway start

```




*本文基于 Hermes Agent v0.13.0 编写。开源项目迭代迅速，建议定期查看 [官方文档](https://hermes-agent.nousresearch.com/docs) 和 [GitHub 仓库](https://github.com/NousResearch/hermes-agent) 获取最新信息。*
            
            
            
                

🛒 京东 挑好物逛京东(Skechers杯具)
                

京东大促进行中，超值好物等你来选
                [去京东看看 →](https://u.jd.com/H1BFc81)
                

* 京东联盟推广链接，你不需要多花钱，我们获得小额佣金