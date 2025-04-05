---
title: "配置 Aider 连接 LLM：模型选择与编辑模式详解"
publishDate: 2024-07-27 # 请将此日期修改为实际发布日期
tags: ["aider", "llm", "ai编程", "配置", "教程"]
description: "本文介绍如何配置 Aider 工具连接不同的 LLM 服务，包括模型选择、编辑模式以及提供详细的配置示例。"
draft: false
---

本文内容基于 [Linux.do 论坛的讨论](https://linux.do/t/topic/490383)，并对 Aider 的配置和使用进行补充说明。站内已有关于 [aider-chat 的共建文档](https://linux.do/t/topic/416641)，本文旨在提供更具体的配置指导。

## 连接 LLM

Aider 内置了对许多主流 LLM 提供商和模型的支持，开箱即用。然而，考虑到许多用户会使用各种中转服务，了解其连接机制很重要。

Aider 通过 **模型 ID 的前缀** 来判断使用哪个渠道以及对应的 API 格式。例如，配置 `openai/claude-3.7-sonnet-think` 意味着：

1.  使用 `openai` 前缀，表示遵循 OpenAI 的 API 格式进行请求。
2.  请求的模型 ID 是 `claude-3.7-sonnet-think`。

因此，推荐的最佳实践是使用像 [New API](https://github.com/Calcium-Ion/new-api) 这样的工具，将你的各种 LLM 渠道统一转换为 OpenAI 兼容的接口。这样，你只需要一份 Aider 配置即可适配所有模型。

## 模型选择

Aider 允许为不同的任务指定不同的模型，主要分为三类：

1.  **主模型 (model):** 执行核心编码任务，如 `/code`、`/ask`、`/help` 等指令都使用此模型。
2.  **编辑器模型 (editor-model):** 在 `/architect` 模式下，规划步骤使用主模型，而具体的代码编辑则由此模型完成。**建议:** 选择一个指令遵循能力强、幻觉较低的模型（通常非纯推理优化模型表现更好）。
3.  **弱模型 (weak-model):** 用于生成 Git commit messages（通过 `/commit` 指令）。通常选择一个价格便宜、性能尚可的小模型即可。

## 编辑模式

Aider 提供两种主要的编辑模式：

*   **diff 模式:** 使用类似 `git diff` 的格式来描述文件更改。这种模式通常更节省 token。
*   **whole 模式:** 重新输出整个需要修改的文件内容。

**选择建议:** 大部分情况下 `diff` 模式效率更高。但某些模型可能在处理 `diff` 格式时容易出错或产生幻觉。如果遇到问题，可以尝试切换到 `whole` 模式。更多详细说明请参考 [Aider 官方文档](https://aider.chat/docs/more/edit-formats.html)。

## 配置示例

以下是一个配置示例，你可以将其保存在用户主目录下的 `.aider.conf.yml` 文件中。

```yaml title=".aider.conf.yml"
# .aider.conf.yml (通常放置在家目录 ~/.aider.conf.yml)

# --- 连接设置 ---
openai-api-base: https://your-new-api-endpoint.com/v1  # 替换为你的 New API 或兼容 OpenAI 的中转地址
# openai-api-key: ${OPENAI_API_KEY} # 可以直接引用环境变量，或在 .env 文件中设置

# --- 模型选择 ---
model: openai/claude-3.7-sonnet-think # 主力编码模型
editor-model: openai/claude-3.5-sonnet # architect 模式下的编辑模型
weak-model: openai/deepseek-chat      # 生成 commit message 的模型

# --- 编辑器与格式 ---
editor: nvim  # /edit 命令调用的编辑器 (可选: vi, code, etc.)
edit-format: diff # /code 模式使用的编辑格式
# editor-edit-format: editor-diff # /architect 模式下的编辑格式 (可选, 默认可能与 edit-format 一致或有特定优化)

# --- 其他设置 ---
architect: true # 默认启用 architect 模式 (可以设为 false)
cache-prompts: true # 启用提示缓存以节省 token (对某些官方 API 可能更有效)
read:  # 启动时默认读取的规则文件或目录 (相对于项目根目录)
  - .rules/
```

同时，在家目录创建 `.env` 文件来存放 API 密钥：

```bash title=".env"
# .env (通常放置在家目录 ~/.env)
# Aider 会自动加载这些环境变量

OPENAI_API_KEY="sk-your-openai-or-newapi-key"
# OPENROUTER_API_KEY="sk-or-v1-your-openrouter-key" # 如果使用 OpenRouter
# ANTHROPIC_API_KEY="sk-ant-..." # 如果直接使用 Anthropic
```

## 使用说明

完成配置后，直接在你的项目工程目录下运行 `aider` 命令即可启动。如果想直接编辑特定文件，可以使用 `aider /path/to/your/file.ext`。
