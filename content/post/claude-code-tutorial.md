+++
title = 'Claude Code 教程'
date = '2026-01-03T15:33:01+08:00'
categories = ['编程','cluade code']
tags = ['code','notes']
toc = true
+++

12 月 27 号 Andrej Karpathy 在 X 上表达了 AI 对软件开发的模式重构带来的焦虑:
> I've never felt this much behind as a programmer. The profession is being dramatically refactored as the bits contributed by the programmer are increasingly sparse and between. I have a sense that I could be 10X more powerful if I just properly string together what has become available over the last ~year and a failure to claim the boost feels decidedly like skill issue. There's a new programmable layer of abstraction to master (in addition to the usual layers below) involving agents, subagents, their prompts, contexts, memory, modes, permissions, tools, plugins, skills, hooks, MCP, LSP, slash commands, workflows, IDE integrations, and a need to build an all-encompassing mental model for strengths and pitfalls of fundamentally stochastic, fallible, unintelligible and changing entities suddenly intermingled with what used to be good old fashioned engineering. Clearly some powerful alien tool was handed around except it comes with no manual and everyone has to figure out how to hold it and operate it, while the resulting magnitude 9 earthquake is rocking the profession. Roll up your sleeves to not fall behind.

<!--more-->

第一条评论是 claude code 创始人 Boris Cherny 的认同：
> I feel this way most weeks tbh. Sometimes I start approaching a problem manually, and have to remind myself “claude can probably do this”. Recently we were debugging a memory leak in Claude Code, and I started approaching it the old fashioned way: connecting a profiler, using the app, pausing the profiler, manually looking through heap allocations. My coworker was looking at the same issue, and just asked Claude to make a heap dump, then read the dump to look for retained objects that probably shouldn’t be there; Claude 1-shotted it and put up a PR. The same thing happens most weeks.

> In a way, newer coworkers and even new grads that don’t make all sorts of assumptions about what the model can and can’t do — legacy memories formed when using old models — are able to use the model most effectively. It takes significant mental work to re-adjust to what the model can do every month or two, as models continue to become better and better at coding and engineering.

> The last month was my first month as an engineer that I didn’t open an IDE at all. Opus 4.5 wrote around 200 PRs, every single line. Software engineering is radically changing, and the hardest part even for early adopters and practitioners like us is to continue to re-adjust our expectations. And this is *still* just the beginning.

可以说，软件开发模式在高手中已经完全被重塑，新的模式在未来一两年内会普及到所有程序员。软件开发从手动挡迈向了自动挡。如何掌握 Andrej Karpathy 提到的那些概念和工具是已经不是效率问题，而是关于就业的问题。

## 安装与配置
1. CC 安装非常简单。参考官方文档即可。2.0 之后支持 homebrew 实现 native 安装了，当然 npm 安装是也继续支持。并且 cc 大量依赖 nodejs，所以建议开发电脑安装 nodejs 和 python，以便后面有些 mcp server 需要用到 node 或者 python。
2. 目前最好的编程模型就是 claude opus, 但是国内注册付费非常困难，且 claude 官方对于中国用户的封禁极其严格。如果不差钱，可以找一些中转站比如 openrouter，目前也支持 claude code 了。或者咸鱼买一些账号体验一下。 
3. 如果想要长期使用并且项目价值不高，可以买一个智谱的 GLM 4.7 模型。这个是目前开源最好的编程模型，在国外的口碑也相当炸裂。[点这个连接可以享受 10% 折扣](https://www.bigmodel.cn/glm-coding?ic=WQ0OVRYE2I)
4. CC 配置 GLM 模型有官方文档 [在 Claude Code 中使用 GLM ](https://docs.bigmodel.cn/cn/coding-plan/tool/claude)
5. 也可以配置 deepseek 模型，网上教程也比较多。 

## 使用
1. 先阅读下面 Orange AI 的入门文档，理解文件夹、
2. 实际上手体验一下，最好是可以配置 opus 体验一下真实的能力，用 glm 的话可能复杂场景会让你感到失望，影响后面的信心。 
3. 然后看 CC 作者关于的使用分享[宝玉翻译版](https://x.com/dotey/status/2007217136176148737)
4. 最后看[A Guide to Claude Code 2.0](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)
5. 多用，先不用考虑移动手机控制、subagent 这些，因为 vibe coding 最大的难点还是在于自己对于代码验收的瓶颈。对于新手来说，如果快速验收 AI 生成的代码或者如何给 AI 搭建一个可验证的测试环境才是瓶颈，其他工具和技巧等熟练了去网上检索或者问 AI 即可。

参考文档
[Andrej Karpathy 对 AI 编程的焦虑](https://x.com/karpathy/status/2004607146781278521)
[A Guide to Claude Code 2.0](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)
[cc 作者 Boris Cherny 如何使用 claude code](https://x.com/bcherny/status/2007179832300581177)
[How I code with agents, without being 'technical'](https://x.com/bentossell/status/2006352820140749073)
[Orange AI 写的关于 cc 的小白安装文档](https://x.com/oran_ge/status/2005419365450252425)