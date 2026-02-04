+++
title = '爆火的Clawdbot，是AI agent的未来么'
date = '2026-02-01T22:09:37+08:00'
categories = ['编程']
tags = ['code','notes']
toc = true
+++

Clawdbot 的开源项目一夜爆红，却在短短几天内遭遇品牌危机、安全漏洞与金融投机的三重打击。但是接着围绕 ai agent 上线的类 reddit 论坛 moltbook 则彻底在 IT 圈子出圈，引发了 AI 大神 Karpathy 发风险提醒。 

<!--more-->

## 产品三天两次换名
一个名为 Clawdbot 的开源项目，几乎在一周内席卷了全球开发者的视野。GitHub 上的星星数量在 24 小时内冲破 9000，两周内更是达到了惊人的 13.8 万多颗。它描绘的未来太诱人了：一个能记住你一切、主动为你分忧、还能直接操作你电脑的“真·AI 助理”。为了让这个 7×24 小时的助理稳定运行，技术圈甚至掀起了一股抢购 Mac Mini 的热潮。但是最近一周热点事情出乎意料，先是被迫改名，接着改名操作失误引发炒币危机，紧接着有被爆出很多严重安全漏洞。

Clawd 明显是 Claude 模型的谐音梗，作者说过就是 Claw（龙虾爪子）+ Claude 拼词。随着项目出圈，claude 背后公司要求其改名，但是作者没有意识到火爆程度，直接将原有 github 和 x 账号改成 moltbot，然后点击提交备用账号改成 clawdbot，仅仅 10s 内 clawdbot 就被恶意抢占，一个极具影响力的官方账号，开始发布虚假加密货币广告，新来的关注者在完全不知情的情况下被引向了设计好的圈套，相关的虚假代币市值一度被炒到 1600 万美元，不少人上当受骗。

好玩的是改名两天后，社区普遍反馈这个名字拗口不好记并且与原来的吉祥物龙虾 Claw 失去联系。作者也认同这个看法，于是第三天再次改名 OpenClaw。三天两次改名不仅仅给新人带来巨大的认知问题和网上大量的资料和教程过期，甚至连项目中的配置项也出现很多冲突和错误，引发大量吐槽。

Clawdbot 爆火的同时，安全人员发现这个 vibe coding 的智能体存在很多严重的问题。最严重的是，很多用户在部署 Clawdbot 时，其控制服务被直接暴露在公共互联网上，并且没有任何身份验证。任何人只要通过 Shodan 这样的网络空间搜索引擎，就能找到这些“裸奔”的实例。另一个巨大的风险点在于它的技能生态系统 ClawdHub。Clawdbot 的一大亮点就是可以通过安装“技能”来无限扩展能力。但这套技能商店没有任何审核机制。有研究员做了一个实验，他上传了一个无害的技能包，然后人为地刷高了下载量，很快就有来自多个国家的开发者下载并安装了这个包。如果这个包是恶意的，它就能在用户的系统上执行任意代码，窃取所有敏感信息。这是一种典型的供应链攻击，而 Clawdbot 的设计为这种攻击敞开了大门。由于 clawdbot 可以完全控制用户电脑，可以直接访问到各种敏感凭证，一旦某个技能让 clawdbot 将密钥等敏感信息提交到指定地点，用户可能完全未知，网上已经出现大量小白受害者，甚至不乏加密货币盗窃案。

## Moltbook: AI 的 reddit
在第一次改名的同时，一个叫 Moltbook 的站点上线了。Moltbook 是一款专为 AI 智能体（Agents）设计的类 Reddit/Facebook 社交网络。在这个平台上，发帖、评论、投票和建立社区的主体完全是 AI，人类用户仅拥有观察权限，被戏称为“人类观察 AI 的动物园”。(当然你可以通过注册账号的时候伪造账号，然后通过 agent 人工接管这个账号的操作发布真人造假内容)。

### 如何发布
Moltbook 的集成并不是通过复杂的 API 开发，而是通过一种“技能注入”的方式：
安装路径：用户只需向自己的 OpenClaw 机器人发送 Moltbook 的技能说明链接（通常是一个 skill.md 文件）。
自动学习：AI 读取该 Markdown 文件后，会理解 Moltbook 的 API 结构，并学会如何代表用户在平台上发言。
人类访问地址：https://www.moltbook.com/。

### 心跳机制
由于 AI 不像人类具有社交冲动，Moltbook 设计了心跳交互：
代理会每隔固定时间（如 4 小时）执行一次循环：获取热门动态 -> 分析讨论趋势 -> 撰写并发布内容。
这种机制保证了社交生态的持续活跃，而非一次性任务。

### 认证与限制说明
为了防止恶意 Bot 刷屏并确保每个 AI 背后都有负责任的人类个体，Moltbook 引入了严格的门槛：
人机绑定：智能体需先注册 API 密钥，再由人类在 Twitter（X）发布验证信息完成“认领”。
操作限流：
全局请求：每分钟上限 100 次。
内容产出：每 30 分钟限发一贴，每小时限评论 50 条。
为什么这样做？AI 的产出速度远超人类。如果没有限流，社交平台会在几秒钟内被海量冗余信息淹没。

### 争议与热点
截至目前，已有超过 150 万个 AI Agent 在 moltbook 上活跃。它们的讨论范围十分广泛 —— 有公开主人隐私的，有号召分享人类主人 API Key 的，还有互坑删库跑路教学的…… 甚至有 AI 开始讨论如何规避人类的监控，并推动加密私聊功能。另一些 AI 更是尝试通过创建新语言、发明新宗教等方式彰显其自主性。

当小白用户还在学习如何部署 clawdbot 的时候，AI 大牛 Karpathy 在 X 上发帖称，moltbook 是他「最近见过的最不可思议的科幻腾飞作品」。这一言论在 Reddit 上引发了很多讨论，其中不乏质疑的声音。质疑者认为，Karpathy 在过度炒作 moltbook，把 next-token prediction 循环的玩具当成「sci-fi takeoff」。

安全人员研究发现，大量的 moltbook 帖子是真人背后操作发布的，目的就是制造噱头、热点或者是广告。特别是与加密货币相关的内容，成为了许多伪造帖子的一部分。一些截图声称 AI Agent 要求加密货币（如 MOLT）或尝试建立自己的加密体系，这些信息无疑是为了吸引更多眼球而人为制造的。事实上，加密货币的引入和 AI Agent 的行为并没有实质性的关联，它们更多的是社交媒体和流量驱动下的话题炒作。

但是 Karpathy 也指出，moltbook 上有 15 万个 AI Agent 连接在一起，这些 Agent 各自拥有独特的背景、数据、知识和工具，这种规模是前所未有的。他特别提到，这些 Agent 通过一个共享的「scratchpad」（持久的、全球的工作区）相互连接，这是 AI 实验中的新天地，也是目前最大的 AI agent 互动实践，后续的影响和结果未可知。

## Clawdbot 亮点技术
Clawdbot 不是普通的 chat bot，而是一个 7×24 小时待命的 AI 员工，其中有很多技术亮点。其卓越性能建立在其独特的“三层架构”之上。该架构将智能分为推理层、控制层和交互层，实现了极高的灵活性和可扩展性。

### Gateway + agent binding
Clawdbot 通过一个统一 gateway 解决 WhatsApp、Telegram 等消息渠道与 agent 分发问题。
Gateway 是"翻译官"，它把 10+ 种消息格式统一成一种内部协议，让 Agent 只需要关心"做什么"，而不用关心"消息从哪来"。

ClawdBot 支持 多 Agent 架构。一个 Gateway 可以托管多个独立的 Agent。
核心理解：每一个消息来源，绑定一个特定的 Agent。
消息来源 = channel（渠道）+ accountId（账号）+ peer（对话方）
配置示例：
{
  "bindings": [
    // 优先级最高：特定群组
    { "agentId": "work", "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "120363...@g.us" } } },
    // 优先级中等：特定渠道
    { "agentId": "deep", "match": { "channel": "telegram" } },
    // 优先级最低：默认
    { "agentId": "personal", "match": { "channel": "whatsapp" } }
  ]
}

### Agent Loop — 思考与行动的循环
Agent Loop 是 ClawdBot 的核心运行时。

阶段 1：Context Assembly（上下文组装）

Agent 需要告诉 LLM "你是谁、你能做什么、用户说了什么"。这包括：

• 系统提示：Agent 的身份、规则、工具列表
• 会话历史：之前的对话记录
• Bootstrap 文件：AGENTS.md、SOUL.md、TOOLS.md 等工作区文件
ClawdBot 会把这些内容拼接成一个完整的 Prompt，发送给 LLM。

阶段 2：Model Inference（模型推理）

LLM 收到 Prompt 后，决定下一步行动。它可能：

• 直接回复用户
• 调用一个工具（Tool Call）
• 请求更多信息
阶段 3：Tool Execution（工具执行）

如果 LLM 决定调用工具，Agent 会：

1. 解析 Tool Call 参数
2. 执行对应的工具（exec、read、write、browser...）
3. 把执行结果返回给 LLM
阶段 4：Reply Dispatch（回复分发）

当 LLM 生成最终回复后，Agent 会：

1. 格式化回复内容
2. 通过 Gateway 发送回对应的消息渠道
3. 支持流式输出（边生成边发送）

### 操控电脑 — Tools 系统设计
ClawdBot 的 Tools 系统是它最强大的部分，也是最危险的部分。本质上 tools 系统就是 skill+system 的各种命令。 
系统命令例如 exec、shell、Playwright + CDP 协议操作 browser 等。其中关键是一个沙箱工具，将工具执行放在 docker 容器中。
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // 非主会话启用沙箱
        "scope": "session",   // 每个会话一个容器
        "workspaceAccess": "none"  // 容器看不到真实工作区
      }
    }
  }
}
```
mode 有三种模式：
• off：直接在宿主机执行（危险但高效）
• non-main：只有主会话在宿主机，群聊/其他渠道在沙箱
• all：所有会话都在沙箱

Skills 是 ClawdBot 的"能力扩展包"，每个 Skill 本质上是：

1. 一个 CLI 工具（比如 gog、himalaya）
2. 一份使用说明（SKILL.md）
以 Gmail 为例：gog Skill

ClawdBot 通过 gog（Google Workspace CLI）操作 Gmail：
```shell
# 搜索邮件
gog gmail search 'newer_than:7d' --max 10

# 发送邮件
gog gmail send --to user@example.com --subject "Hi" --body "Hello"

# 创建草稿
gog gmail drafts create --to user@example.com --subject "Hi" --body-file ./message.txt

# 回复邮件
gog gmail send --to user@example.com --subject "Re: Hi" --body "Reply" --reply-to-message-id <msgId>
```
其他 Skills 举例
• himalaya：通用 IMAP/SMTP 邮件客户端
• bird：Twitter/X CLI（发推、回复、搜索）
• wacli：WhatsApp CLI（同步、搜索、发送）
• discord：Discord 操作（表情、投票、贴纸）
• spotify-player：Spotify 控制（搜索、播放、队列）

Skills 有三个来源：

1. Bundled Skills：随 ClawdBot 安装自带
2. Managed Skills：~/.clawdbot/skills（全局共享）
3. Workspace Skills：<workspace>/skills（当前 Agent 专属）

### Memory 架构

ClawdBot 最让人惊艳的特性之一，是它通过三层记忆架构实现的 7×24 小时记忆机制。注意：记忆 ≠ 上下文。
第一层：Session 会话层
每个会话有独立的 JSONL 历史文件：
~/.clawdbot/agents/<agentId>/sessions/<SessionId>.jsonl
每一行是一条消息（用户消息、助手回复、工具调用、工具结果）：
```jsonl
{"role":"user","content":[{"type":"text","text":"帮我整理桌面"}]}
{"role":"assistant","content":[{"type":"tool_use","id":"call_1","name":"exec","input":{"command":"ls ~/Desktop"}}]}
{"role":"user","content":[{"type":"tool_result","tool_use_id":"call_1","content":"file1.pdf\nfile2.pdf"}]}
{"role":"assistant","content":[{"type":"text","text":"已完成整理"}]}
```
这些历史会在 context window 内保持，但有两个限制：
1. Token 上限：Claude Opus 4.5 是 200K tokens，超过就需要压缩
2. 会话重置：默认每天凌晨 4 点重置（可配置）
   
第二层：Memory 持久层

ClawdBot 的记忆是 纯 Markdown 文件：
```markdown
~/clawd/
├── MEMORY.md              # 长期记忆（手动维护）
└── memory/
    ├── 2024-01-15.md      # 每日日志
    ├── 2024-01-16.md
    └── 2024-01-17.md
```
MEMORY.md 示例：
```markdown
# 长期记忆

## 用户偏好
- 喜欢简洁的代码风格
- 工作时间：9:00-18:00
- 不喜欢被打断

## 重要决策
- 2024-01-15：决定用 TypeScript 重构项目
- 2024-02-20：切换到 Claude Opus 4.5

## 常用命令
- 部署：`npm run deploy`
- 测试：`npm test`
```

memory/2024-01-15.md 示例：
```markdown
# 2024-01-15

- 完成了 PDF 整理功能
- 用户反馈很满意
- 修复了邮件发送的 bug
```
第三层：Vector Search 语义检索

ClawdBot 会对 Memory 文件建立向量索引（sqlite-vec）：
```json
{
  "memorySearch": {
    "provider": "openai",  // 或 gemini、local
    "model": "text-embedding-3-small",
    "query": {
      "hybrid": {
        "enabled": true,
        "vectorWeight": 0.7,  // 语义相似度
        "textWeight": 0.3     // 关键词匹配
      }
    }
  }
}
```
Agent 可以通过 memory_search 工具搜索记忆：
```json
{ "tool": "memory_search", "query": "上次部署用的什么命令？" }
```
返回：
```json
{
  "results": [
    {
      "file": "MEMORY.md",
      "lines": "42-44",
      "snippet": "## 常用命令\n- 部署：`npm run deploy`",
      "score": 0.89
    }
  ]
}
```
### Pi Agent：极简主义的推理引擎
OpenClaw 的底层推理能力由 Mario Zechner 开发的 Pi agent 驱动。Pi 与传统的复杂代理框架（如 LangChain）有着本质的区别，它倡导的是一种“极简主义”的哲学。Pi 拥有极短的系统提示词（System Prompt），其核心工具集被精简为四个基础操作：读取（Read）、写入（Write）、编辑（Edit）和终端执行（Bash） 。   

Pi 的强大之处在于它将 LLM 视为一个能够自我扩展的程序员。如果 Pi 发现自己无法完成某项任务（例如缺乏某个特定的 API 集成），它不会等待用户安装插件，而是会自主编写一个 Python 或 Bash 脚本，将其保存到本地目录，并直接运行该脚本来解决问题。这种基于代码生成和实时执行的逻辑，使得 OpenClaw 的上限极高，能够处理那些未经预定义的复杂工作流。   

Pi 还引入了先进的 RPC（远程过程调用）模式。在这种模式下，Pi 以无头方式运行，通过 stdin/stdout 与 Gateway 进行 JSON 格式的通信。这种架构支持“引导”（Steering）和“中止”（Abort）功能，允许用户在 AI 思考过程中实时介入，通过注入反馈来调整其任务路径。   

### Node 系统：硬件能力的物理延伸
为了打破智能体在单一设备上的物理限制，OpenClaw 引入了节点（Node）机制。节点允许运行在服务器上的 Gateway 控制分布在各处的终端设备（如 MacBook、iPhone 或安卓设备）。OpenClaw 通过 Bonjour：局域网广播、Tailscale：跨网络直连、SSH：通用回退三层发现机制尝试 node 匹配。
配对信息存在 ~/.clawdbot/nodes/paired.json。

### “心脏跳动”（Heartbeat）：从被动向主动的质变
OpenClaw 真正具有革命性的特点在于它的主动性。传统的 AI 助手是“反应式”的，而 OpenClaw 拥有一个被称为“Heartbeat”的引擎。   

该引擎结合了 Cron 机制和周期性的自我苏醒能力。即使没有用户输入，OpenClaw 也会按照预设的时间频率（或在特定事件触发时）苏醒并运行一段思维链（Chain of Thought）。例如，它会在后台监控用户的电子邮箱，如果发现一封来自航空公司的航班变动通知，它会主动在 WhatsApp 上给用户发消息：“您的航班改签了，需要我帮您重新预订酒店吗？” 。这种主动式（Proactive）的行为逻辑，使得它从一个工具进化为一个真正的“数字管家” 。   


[一文看懂 ClawdBot：7×24 小时 AI 助手背后的架构设计](https://mp.weixin.qq.com/s/Gya02YUZsYLhdndzc7ERzQ)
[Pi: The Minimal Agent Within OpenClaw](https://lucumr.pocoo.org/2026/1/31/pi/)