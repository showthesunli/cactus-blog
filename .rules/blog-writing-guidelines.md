# 博客撰写流程与规范 (Astro Cactus 项目)

本文档基于 `.rules/convertion.md` 和项目代码结构，为 `astro-cactus` 项目撰写博客文章提供指导。

## 1. 文件格式

*   **1. 选择:** 使用 Markdown (`.md`) 或 MDX (`.mdx`)。
*   **2. 何时使用 `.mdx`:** 当你需要在文章中嵌入自定义 Astro/React/Vue 组件或使用 JSX 语法时。
*   **3. 存放位置:** 通常在 `src/content/post/` 目录下。
*   **4. 文件名 (URL Slug) 规范 (SEO 建议):**
    *   **重要性:** 文件名将直接影响生成的博客文章 URL (slug)，这对 SEO 至关重要。一个好的 URL slug 应该是简洁、描述性强且易于理解的。
    *   **使用描述性关键词:** 文件名应包含与文章内容相关的核心关键词。
    *   **保持简洁:** 避免过长或包含不必要词语（如 "a", "the", "in" 等，除非它们对理解至关重要）的文件名。
    *   **小写字母:** 统一使用小写英文字母或标准汉语拼音。
    *   **连字符分隔:** 使用连字符 (`-`) 分隔单词或拼音音节。**绝对不要**使用空格、下划线 (`_`) 或其他特殊字符。
    *   **语言选择:**
        *   推荐使用有意义的英文单词组合。
        *   如果目标读者主要是中文用户，也可以使用清晰、标准的汉语拼音。
        *   保持项目内风格统一。
    *   **避免日期:** 通常不在文件名中包含日期，`publishDate` 和 `updatedDate` frontmatter 字段负责处理日期信息。
    *   **示例:**
        *   **推荐:**
            *   `astro-optimization-tips.md` (英文)
            *   `ruhe-zhuanxie-astro-boke.md` (拼音)
            *   `crawl4ai-python-script.md`
        *   **不推荐:**
            *   `Blog Post Draft v2.md` (包含空格和大写字母)
            *   `文章1.md` (非描述性)
            *   `how_to_write_Astro_Blog_0405.md` (包含下划线、大写字母和日期)
            *   `guanyuwomen.md` (拼音不清晰，应为 `guan-yu-wo-men.md` 或 `about-us.md`)

## 2. Frontmatter (元数据)

*   **格式:** 每篇文章开头必须包含 YAML frontmatter 块 (`--- ... ---`)。
*   **必需字段:**
    *   `title`: (string) 文章标题。**SEO 关键:** 这是页面的 `<title>` 标签和社交媒体分享时的主要标题，应包含核心关键词并具有吸引力。
    *   `publishDate`: (Date, e.g., `YYYY-MM-DD`) 文章发布日期。
        *   *注意:* 日期格式可以更灵活，例如 `"DD MMMM YYYY"` (如 `"27 January 2023"`) 或完整的 ISO 字符串 (`"YYYY-MM-DDTHH:mm:ssZ"`)，但建议在项目中保持一致。
*   **常用字段:**
    *   `updatedDate`: (Date, e.g., `YYYY-MM-DD` or `"DD MMMM YYYY"`) 文章更新日期 (可选)。搜索引擎可能会关注内容的更新时间。
    *   `tags`: (string[]) 文章标签数组 (例如 `["Astro", "指南"]`)。标签会自动处理为小写并去重。有助于站内内容组织和发现，间接影响用户体验和 SEO。
    *   `description`: (string) 文章简短描述。**SEO 关键:** 这通常用作搜索引擎结果页面 (SERP) 上的 meta description，应准确概括文章内容并包含关键词，吸引用户点击。
    *   `draft`: (boolean) 如果设置为 `true`，则该文章通常只在开发环境中可见 (可选)。
    *   `coverImage`: (object) 用于文章顶部的封面/英雄图片 (可选)。包含以下属性：
        *   `src`: (string) 图片路径，通常是相对于当前 Markdown 文件的相对路径 (例如 `./cover.png`)。
        *   `alt`: (string) 图片的替代文本。
    *   `ogImage`: (string) 自定义社交媒体预览图 URL (可选)。**若省略此字段，系统将根据文章标题和日期（优先使用 `updatedDate`，其次是 `publishDate`）自动生成一个预览图。**
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
        *   **自定义标题:** 你可以在类型后面添加方括号来自定义标题：
        ```markdown
        :::note[我的自定义标题]
        这是一个带有自定义标题的笔记。
        :::
        ```
        ```
    *   **其他:** 留意项目中可能存在的其他 Remark/Rehype 插件引入的自定义语法。参考现有文章是最佳实践。
*   **图标 (MDX):** 在 `.mdx` 文件中使用图标时，遵循 `astro-icon` 组件的使用方式。
*   **组件 (MDX):** 在 `.mdx` 文件中嵌入自定义组件时，像在 `.astro` 文件中一样导入和使用。
*   **脚注/参考文献:** 使用标准的 Markdown 脚注语法：
    ```markdown
    这是一个需要引用的文本[^1]。

    [^1]: 这是脚注的内容。
    ```
    脚注内容通常会自动显示在文章末尾。
*   **键盘快捷键:** 使用 `<kbd>` HTML 标签来表示键盘按键：
    ```markdown
    按 <kbd>Ctrl</kbd> + <kbd>C</kbd> 复制文本。
    ```

## 4. 代码块

*   **语法:** 使用标准的 Markdown 代码块 (``` ```)。
*   **高亮:** `astro-expressive-code` 插件负责语法高亮和样式。
*   **增强功能:** `astro-expressive-code` 支持更多功能，例如添加代码块标题 (`js title="filename.js"`) 或高亮特定行 (`{1, 3-5}`)。详情请查阅其文档或参考示例文章。

## 5. 链接与图片

*   **语法:** 使用标准的 Markdown 语法。
*   **外部链接:** 自动添加 `target="_blank"` 和 `rel="noopener noreferrer"`。
*   **图片:** 可能会被 `rehype-unwrap-images` 处理，避免被 `<p>` 标签包裹。
*   **图片存放:**
    *   **通用内容图片:** 推荐将文章中使用的图片文件存放在项目根目录下的 `public/` 文件夹内（例如 `public/images/`）。在 Markdown 中引用这些图片时，请使用**根相对路径** (以 `/` 开头)，例如：
    ```markdown
    ![图片描述](/images/your-image-name.png)
    ```
    *   **封面图片 (`coverImage.src`):** 对于 `coverImage` 中的 `src` 字段，通常使用**相对于当前 Markdown 文件的相对路径**，例如：
    ```yaml
    coverImage:
      src: "./cover.png" # 图片与 markdown 文件在同一目录
      alt: "封面图片描述"
    ```

## 6. 代码检查与格式化

*   **推荐:** 在提交前运行 `pnpm run format` 和 `pnpm run lint`。
*   **目的:** 确保代码风格统一，特别是 `.mdx` 文件中可能包含的 JS/TS 代码块或组件。

## 7. 一致性

*   **关键:** 浏览 `src/content/post/` 下的现有文章。
*   **学习:** 参考它们的 frontmatter 结构、内容组织方式和自定义语法的使用。
*   **目标:** 保持新文章与项目整体风格和规范的一致性。

## 8. Notes (笔记)

*   **用途:** 除了常规的博客文章 (`src/content/post/`)，项目还支持"笔记" (`src/content/note/`)。笔记通常用于发布更短、更简洁的内容或想法。
*   **特点:** 笔记通常不包含复杂的结构（如多级标题），但遵循相同的 Frontmatter 和 Markdown 基础规范。
*   **示例:** 参考 `src/content/note/welcome.md`。
