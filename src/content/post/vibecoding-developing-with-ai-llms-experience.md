---
title: "AI 编程实践：LLM 与 Memory Bank 提升项目可控性 (Vibecoding 经验)"
publishDate: 2024-05-16 # 请改为实际开始写作或计划发布的日期
tags: ["AI Programming", "LLM", "Memory Bank", "Project Controllability", "Aider", "Development Workflow", "Vibecoding"]
description: "记录使用 AI 编程方法和 LLM 辅助开发 Vibecoding 项目的过程、选择与心得体会。" # 初步描述
draft: true # 标记为草稿
---

# Vibecoding 开发过程记录

开发了一个 tg 的 [user bot](https://github.com/showthesunli/telegram-logger)，仍然是全程aider，稍微总结下，发现还是很多可以说的。

## LLM选择

我一开始嫌弃gemini 2.5 pro是思考模型，首字太慢了，等着急人，所以前半程全程ds v3，这个真是性价比之王，便宜好用，速度又快。
后来发现不行，gemini 的大上下文太香了，用ds总得考虑你提出的任务的规模，不然爆上下文再重新规划prompt，真是太浪费时间。

## 工具选择

仍然是aider，这货越用越顺手，不想换了。并且这次我完全弃用了 aider 的 /architect 模式，为嘛？下面就说。

## vibecoding 面对的问题

我觉得用ai编码，如果不是那种用完就丢的脚本，那么最核心的目标就是项目的可控性。
可控性体现在两方面，一个是你对项目的控制，另一个是llm对项目的控制。
由于llm上下文容量、注意力和幻觉的限制，llm并不可靠。人脑同样也有这问题，并且似乎人脑更不可靠。
那么解决方式是什么？文档！

## 开发模式

我读了 cline 的 [memory bank](https://docs.cline.bot/improving-your-prompting-skills/cline-memory-bank)之后，
发现这就是一个简单又高效的控制项目的方式。

简单来说它就是一个自文档系统，每次llm行动之前和之后，都使用文档化的形式压缩项目信息提供用于llm的行动步骤。
这真是一个简单又好用的模式，它有几大好处。

- 文档化的形式自动的描述了项目的目标和当前的状态，可以使得llm直接了解项目而不需要用源码塞满它的上下文
- 文档自成prompt，用于指导LLM下一步行动
- 文档自动拆分任务，减小当前任务的规模，保证LLM的注意力，尽可能少的产生幻觉
- 文档对人友好，特别当你指导llm一步步迭代文档的时候，你会对项目变得很有信心

这些好处完美的解决 vibecoding 所面对的问题。

但是cline和roo code的实现方式我不喜欢，我讨厌它呼呼啦啦的发送请求，呼呼啦啦的生成一堆代码，这种代码让人心虚。

所以我用 aider 实践了一遍这个开发思路。这个过程相当有趣，下面就说说咋做的。

## aider 中使用memory bank开发过程

首先我建了一个RFC 001的 RFC 文档，并把这个文档加入 editable 中,然后使用 /ask 模式聊需求。
每聊一次后都使用 /code 指令让 llm 修改文档，然后基于这份RFC文档反复迭代，直到自己感觉差不多。
LLM真的是会帮你发现很多设计上的不足。

当 RFC 文档完备之后，我把这份 RFC 文档放入 read-only ，然后重新建了一份 IMP 的实现文档，将它加入 editable 中。
然后开始用 /ask 和 /code 模式重复上述过程，到最后可以清晰的得到一份实现步骤。这份文档每个步骤前都有实现状态 []

最后将项目本身的 rules 文件和 rfc 文档放入read-only ，将 IMP 文档放入 editable 中，让 llm 开始一步步的实现。
每实现一个步骤就把 IMP 文档的对应步骤☑️。到最后我发现我使用的 prompt 几乎全部都是实现第几步，写的都有点烦

如果你感兴趣，可以看看我项目中的这两份文档，[RFC-003-用户机器人设计.md](https://github.com/showthesunli/telegram-logger/blob/main/RFC/003-%E7%94%A8%E6%88%B7%E6%9C%BA%E5%99%A8%E4%BA%BA%E8%AE%BE%E8%AE%A1.md) 和 [IMP-001-RFC003-UserBot.md](https://github.com/showthesunli/telegram-logger/blob/main/IMP/001-RFC003-UserBot.md) 。同时，它们的 git 提交记录也很清晰的
展示了迭代的过程，并且 aider 自动生成 commit message 真是让人舒服。

在这个过程中，还有一个最大的惊喜就是，通过 IMP 文档和 aider 自身的 repomap，它可以非常精确的找到要修改的代码。
我的体感是，它比 rag 之类的奇技淫巧准确多了。通常的情况是，你让 llm 实现某一步骤，然后看他自言自语说一堆，
然后最后它来了一句，我需要编辑某某文件，这时候 aider 就会提示让你将这个文件加入 editable，然后它重新分析，很有趣。

这种开发方式很顺畅，但是 debug 要了命。基本上是将报错信息丢给 chat 让他自己改。问题在于随着规模增加，这种模式无异于在
堆屎山，到最后大概率是修好了a又坏了b。

以后再做的话，我想应该在 RFC 里面更加清晰的写明使用场景，然后在 IMP 文档里面让 LLM 根据实使用场景先写单元测试,再写具体
的实现步骤。

## 最后

我觉得 ai 编程对我来说，最大的优势是，它促使我真正开始做东西了。以前有很多想法，你想做，你知道你能做，但开始做之前
总是会觉烦，总是很难迈出第一步。有 ai 之后，动手开始似乎不再那么痛苦。
