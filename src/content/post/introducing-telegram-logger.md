---
title: "介绍 Telegram Logger：一个强大的 Telegram 消息记录与管理工具"
publishDate: 2025-04-06
tags: ["Python", "Telegram", "Open Source", "Tools", "Message Logging"]
description: "详细介绍 telegram-logger 项目，一个用于记录、管理和自动处理 Telegram 消息的 Python 工具，包括其主要功能、安装部署方法和配置选项。"
---

你是否曾经想过要可靠地备份重要的 Telegram 聊天记录？或者需要一个自动化工具来监控、转发甚至加密存储来自特定用户或群组的消息和媒体文件？如果你有这些需求，那么 [showthesunli/telegram-logger](https://github.com/showthesunli/telegram-logger) 这个开源项目可能正是你所寻找的解决方案。

`telegram-logger` 是一个使用 Python 编写的工具，基于强大的 [Telethon](https://github.com/LonamiWebs/Telethon) 库构建，旨在提供一个全面、可配置的 Telegram 消息记录和管理系统。

## 主要功能亮点

该项目提供了丰富的功能，使其不仅仅是一个简单的消息备份工具：

*   **📝 全面记录:** 记录新消息，以及消息的编辑和删除操作，让你不会错过任何重要信息。
*   **🔄 智能转发:** 可以根据配置，自动将来自指定用户、频道（皮套）或群组的消息转发到指定的日志频道。
*   **🖼️ 媒体处理:** 支持自动下载消息中的媒体文件（图片、视频、文档等），并可以选择使用密码进行加密存储，保护你的隐私数据。
*   **🗑️ 自动清理:** 内建基于时间的清理机制，可以自动删除过期的消息记录和媒体文件，防止存储空间无限增长。
*   **🔒 安全存储:** 数据库文件和下载的媒体文件都可以进行加密，增加数据安全性。
*   **⚙️ 高度可配置:** 通过环境变量 (`.env` 文件) 可以轻松配置各种行为，如忽略特定聊天、设置消息持久化时间、调整转发规则等。

## 快速开始：使用 Docker Compose 部署 (推荐)

部署 `telegram-logger` 最便捷的方式是使用 Docker Compose。

1.  **获取代码:**
    ```bash
    git clone https://github.com/showthesunli/telegram-logger.git
    cd telegram-logger
    ```
2.  **准备环境:**
    *   复制配置文件模板：`cp .env.example .env`
    *   编辑 `.env` 文件，填入你的 Telegram `API_ID`、`API_HASH` (可从 [my.telegram.org](https://my.telegram.org/apps) 获取) 和 `LOG_CHAT_ID` (你的日志频道的 ID)。
    *   **重要:** 确保 `SESSION_NAME` 指向 `db/` 目录，例如 `db/user`，以便会话文件保存在 Docker 卷中。
    *   根据需要配置其他选项，如 `FORWARD_USER_IDS`, `FORWARD_GROUP_IDS`, `FILE_PASSWORD` 等。
    *   创建必要的本地目录：
        ```bash
        mkdir -p files/{db,media,log}
        ```
3.  **首次启动 (交互式登录):**
    首次运行需要登录你的 Telegram 账号。**不要**直接使用 `docker compose up`。
    ```bash
    # 拉取镜像
    docker compose pull
    # (可选) 清理旧会话文件，如 rm ./files/db/user.session
    # 运行交互式登录
    docker compose run --rm telegram-logger
    ```
    按照终端提示输入你的手机号、验证码和可能的两步验证密码。成功后，你应该能在 `./files/db/` 目录下看到 `.session` 文件。
4.  **正常启动:**
    登录成功并生成会话文件后，使用以下命令启动服务：
    ```bash
    # 后台启动
    docker compose up -d
    # 或前台启动查看日志
    docker compose up
    ```

## 本地安装与运行

如果你倾向于本地运行：

1.  确保你的环境满足要求 (Python 3.13+, uv)。
2.  克隆仓库并进入目录。
3.  安装依赖：`uv pip sync`。
4.  配置 `.env` 文件 (同 Docker 步骤 2)。
5.  运行程序：`python main.py`，首次运行同样需要交互式登录。

## 总结

`telegram-logger` 提供了一个强大而灵活的方式来管理你的 Telegram 消息。无论是个人备份、社群管理还是信息监控，它都能提供可靠的支持。其对媒体文件的处理和加密功能更是增加了实用性和安全性。

如果你对这个项目感兴趣，不妨访问它的 GitHub 仓库了解更多详情，或者直接动手部署体验一下吧！

**项目地址:** [https://github.com/showthesunli/telegram-logger](https://github.com/showthesunli/telegram-logger)
