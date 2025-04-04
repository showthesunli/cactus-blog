# 博客撰写流程与规范 (Astro Cactus 项目)

本文档基于 `.rules/convertion.md` 和项目代码结构，为 `astro-cactus` 项目撰写博客文章提供指导。

## 1. 文件格式

*   **选择:** 使用 Markdown (`.md`) 或 MDX (`.mdx`)。
*   **何时使用 `.mdx`:** 当你需要在文章中嵌入自定义 Astro/React/Vue 组件或使用 JSX 语法时。
*   **存放位置:** 通常在 `src/content/post/` 目录下。

## 2. Frontmatter (元数据)

*   **格式:** 每篇文章开头必须包含 YAML frontmatter 块 (`--- ... ---`)。
*   **必需字段:**
    *   `title`: (string) 文章标题。
    *   `publishDate`: (Date, e.g., `YYYY-MM-DD`) 文章发布日期。
*   **常用字段:**
    *   `updatedDate`: (Date, e.g., `YYYY-MM-DD`) 文章更新日期 (可选)。
    *   `tags`: (string[]) 文章标签数组 (例如 `["Astro", "指南"]`)。标签会自动处理为小写并去重。
    *   `description`: (string) 文章简短描述，用于 SEO 和预览。
    *   `ogImage`: (string) 自定义社交媒体预览图 URL (可选)。若省略，系统可能会自动生成。
*   **自动添加字段 (无需手动添加):**
    *   `readingTime`: (string) 文章阅读时长，由 `remark-reading-time` 插件自动计算。
*   **示例:**
    ```yaml
    ---
    title: "如何撰写 Astro Cactus 博客"
    publishDate: 2024-04-04
    tags: ["Astro", "规范", "教程"]
    description: "遵循项目规范撰写博客文章的指南。"
    ---
    ```

## 3. 内容撰写

*   **语法:** 使用标准的 Markdown 语法。
*   **样式:**
    *   主要内容（段落、标题、列表等）的样式由 `@tailwindcss/typography` 通过 `prose` 类自动处理。**避免**在 Markdown 内容中直接添加 Tailwind 类来设置基础文本样式。
*   **自定义语法/指令:**
    *   **Admonitions (提示块):** 项目使用 `remark-admonitions.ts`。你可以使用以下指令：
        ```markdown
        :::note[可选标题]
        这是一个笔记。
        :::

        :::tip
        这是一个提示。
        :::

        :::important
        这是一个重要说明。
        :::

        :::caution
        这是一个需要注意的地方。
        :::

        :::warning
        这是一个警告。
        :::
        ```
    *   **其他:** 留意项目中可能存在的其他 Remark/Rehype 插件引入的自定义语法。参考现有文章是最佳实践。
*   **图标 (MDX):** 在 `.mdx` 文件中使用图标时，遵循 `astro-icon` 组件的使用方式。
*   **组件 (MDX):** 在 `.mdx` 文件中嵌入自定义组件时，像在 `.astro` 文件中一样导入和使用。

## 4. 代码块

*   **语法:** 使用标准的 Markdown 代码块 (``` ```)。
*   **高亮:** `astro-expressive-code` 插件负责语法高亮和样式。

## 5. 链接与图片

*   **语法:** 使用标准的 Markdown 语法。
*   **外部链接:** 自动添加 `target="_blank"` 和 `rel="noopener noreferrer"`。
*   **图片:** 可能会被 `rehype-unwrap-images` 处理，避免被 `<p>` 标签包裹。

## 6. 代码检查与格式化

*   **推荐:** 在提交前运行 `pnpm run format` 和 `pnpm run lint`。
*   **目的:** 确保代码风格统一，特别是 `.mdx` 文件中可能包含的 JS/TS 代码块或组件。

## 7. 一致性

*   **关键:** 浏览 `src/content/post/` 下的现有文章。
*   **学习:** 参考它们的 frontmatter 结构、内容组织方式和自定义语法的使用。
*   **目标:** 保持新文章与项目整体风格和规范的一致性。
