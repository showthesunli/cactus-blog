# Project Conventions for LLM Assistants

This document outlines the key conventions and technologies used in the `astro-cactus` blog project. Please adhere to these guidelines when generating or modifying code.

## 1. Core Technologies

*   **Framework:** Astro (`.astro` files)
*   **Language:** TypeScript (primarily in `.ts` files and within `<script>` tags in `.astro` files)
*   **Content:** Markdown (`.md`) and MDX (`.mdx`)
*   **Styling:** Tailwind CSS (utility-first)
*   **Package Manager:** pnpm

## 2. Formatting

*   **Primary Formatter (JS/TS/JSON):** Biome (`biome format`). Adhere strictly to Biome's formatting rules.
*   **Secondary Formatter (Astro/Other):** Prettier (`prettier`). Ensure Astro files (`.astro`) and potentially other non-JS/TS/JSON files are formatted using Prettier.
*   **Tailwind Class Sorting:** Prettier (via `prettier-plugin-tailwindcss`) should automatically sort Tailwind utility classes within templates. Ensure generated code maintains this sorting.
*   **Import Sorting:** Biome handles import sorting (`biome check --formatter-enabled=false --write`). Ensure imports are organized according to Biome's rules.
*   **Workflow:** Use the project scripts:
    *   `pnpm run format:code` (Biome + Prettier)
    *   `pnpm run format:imports` (Biome import sorting)
    *   `pnpm run format` (Runs both)

## 3. Linting

*   **Linter:** Biome (`biome lint`).
*   **Requirement:** All code must pass Biome's linting checks without errors or warnings.
*   **Workflow:** Use `pnpm run lint` to check for issues.

## 4. Type Checking

*   **Checker:** Astro Check / TypeScript (`astro check`).
*   **Requirement:** All TypeScript code must pass type checking. Avoid using `any` unless absolutely necessary and justified. Use specific types and interfaces. Use `@ts-expect-error` or `@ts-ignore` only as a last resort with a clear explanation.
*   **Workflow:** Use `pnpm run check` to verify type safety.

## 5. Styling (Tailwind CSS)

*   **Methodology:** Primarily use Tailwind utility classes directly within the HTML/Astro templates.
*   **Custom CSS:** Avoid creating separate CSS files or using `@apply` unless essential for complex components or base styles not easily achievable with utilities.
*   **Typography:** Leverage the `@tailwindcss/typography` plugin for styling Markdown/MDX content rendered via `prose` classes.

## 6. Content (Markdown / MDX)

*   **Metadata:** Use YAML frontmatter for post metadata (e.g., `title`, `publishDate`, `updatedDate`, `tags`).
*   **Custom Syntax:** Be aware of potential custom Markdown/MDX features enabled by Remark/Rehype plugins (e.g., directives like admonitions, as seen in `remark-admonitions.ts`). Follow existing patterns for using these features.

## 7. Components

*   **Structure:** Use Astro components (`.astro`) for UI elements and page structure.
*   **Props:** Define component props using TypeScript interfaces within the component's frontmatter script (`---`).

## 8. Icons

*   **Implementation:** Use the `astro-icon` component/package for rendering SVG icons. Follow its usage patterns.

## 9. API Routes / Endpoints

*   **Implementation:** Use Astro's file-based routing for API endpoints (e.g., `.ts` files in the `src/pages/` directory, like `og-image/[...slug].png.ts`).
*   **Data Fetching:** Follow Astro patterns for data fetching within API routes and `getStaticPaths`.

## 10. General Guidelines

*   **Clarity:** Write clear, readable, and maintainable code.
*   **Consistency:** Follow existing code patterns and conventions within the project.
*   **Dependencies:** Use `pnpm` to manage dependencies.
