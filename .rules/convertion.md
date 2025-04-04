# LLM 助手项目规范

本文档概述了 `astro-cactus` 博客项目中使用的关键规范和技术。在生成或修改代码时，请遵守这些准则。

## 1. 核心技术

*   **Framework:** Astro (`.astro` 文件)
*   **Language:** TypeScript (主要在 `.ts` 文件和 `.astro` 文件内的 `<script>` 标签中使用)
*   **Content:** Markdown (`.md`) 和 MDX (`.mdx`)
*   **Styling:** Tailwind CSS (utility-first)
*   **Package Manager:** pnpm

## 2. Formatting (格式化)

*   **主要格式化工具 (JS/TS/JSON):** Biome (`biome format`)。严格遵守 Biome 的格式化规则。
*   **次要格式化工具 (Astro/其他):** Prettier (`prettier`)。确保 Astro 文件 (`.astro`) 以及其他可能的非 JS/TS/JSON 文件使用 Prettier 进行格式化。
*   **Tailwind Class 排序:** Prettier (通过 `prettier-plugin-tailwindcss`) 应自动对模板内的 Tailwind utility classes 进行排序。确保生成的代码保持此排序。
*   **Import 排序:** Biome 处理 import 排序 (`biome check --formatter-enabled=false --write`)。确保 imports 按照 Biome 的规则进行组织。
*   **工作流程:** 使用项目脚本：
    *   `pnpm run format:code` (Biome + Prettier)
    *   `pnpm run format:imports` (Biome import 排序)
    *   `pnpm run format` (运行两者)

## 3. Linting (代码检查)

*   **Linter:** Biome (`biome lint`)。
*   **要求:** 所有代码必须通过 Biome 的 linting 检查，不得有错误或警告。
*   **工作流程:** 使用 `pnpm run lint` 检查问题。

## 4. Type Checking (类型检查)

*   **Checker:** Astro Check / TypeScript (`astro check`)。
*   **要求:** 所有 TypeScript 代码必须通过类型检查。避免使用 `any`，除非绝对必要且有充分理由。使用具体的 types 和 interfaces。仅在最后手段并附有明确解释的情况下使用 `@ts-expect-error` 或 `@ts-ignore`。
*   **工作流程:** 使用 `pnpm run check` 验证类型安全。

## 5. Styling (样式 - Tailwind CSS)

*   **方法论:** 主要在 HTML/Astro 模板中直接使用 Tailwind utility classes。
*   **自定义 CSS:** 避免创建单独的 CSS 文件或使用 `@apply`，除非对于复杂的组件或无法轻易通过 utilities 实现的基础样式是必需的。
*   **Typography:** 利用 `@tailwindcss/typography` 插件为通过 `prose` classes 渲染的 Markdown/MDX 内容设置样式。

## 6. Content (内容 - Markdown / MDX)

*   **Metadata:** 使用 YAML frontmatter 定义文章元数据 (例如 `title`, `publishDate`, `updatedDate`, `tags`)。
*   **自定义语法:** 注意由 Remark/Rehype 插件启用的潜在自定义 Markdown/MDX 功能 (例如，像 `remark-admonitions.ts` 中看到的 admonitions 等 directives)。遵循现有模式使用这些功能。

## 7. Components (组件)

*   **结构:** 使用 Astro components (`.astro`) 构建 UI 元素和页面结构。
*   **Props:** 在组件的 frontmatter script (`---`) 中使用 TypeScript interfaces 定义组件 props。

## 8. Icons (图标)

*   **实现:** 使用 `astro-icon` component/package 渲染 SVG 图标。遵循其使用模式。

## 9. API Routes / Endpoints (API 路由 / 端点)

*   **实现:** 使用 Astro 基于文件的路由定义 API endpoints (例如，`src/pages/` 目录中的 `.ts` 文件，如 `og-image/[...slug].png.ts`)。
*   **Data Fetching:** 遵循 Astro 在 API routes 和 `getStaticPaths` 中进行数据获取的模式。

## 10. 通用指南

*   **清晰性:** 编写清晰、可读、可维护的代码。
*   **一致性:** 遵循项目内现有的代码模式和规范。
*   **Dependencies:** 使用 `pnpm` 管理依赖项。
