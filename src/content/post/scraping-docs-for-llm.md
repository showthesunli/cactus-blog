---
title: "一个爬取文档站点生成 LLM 知识库语料的 Python 脚本"
publishDate: 2025-04-05
updatedDate: "2025-02-12" # 根据更新内容添加，并确保是字符串
tags: ["Python", "Web Scraping", "LLM", "Knowledge Base", "Script", "crawl4ai", "教程"]
description: "介绍并分享一个使用 crawl4ai 库编写的 Python 脚本，用于爬取文档型网站，生成 Markdown 文件，方便构建 LLM 知识库语料。"
---

## 背景

平常开发的时候经常要阅读第三方包文档。通常是粗读整个文档，对各个模块有概念了以后就上手写，写的时候再去翻文档找调用细节。

这里面其实有两个点可以用 AI 加速：

1.  让它介绍开发包的核心功能和用法。
2.  让它基于第三方包直接实现某个具体的功能函数。

而且大部分情况下，我们并不需要用到第三方包的全部功能，AI 可以很好地帮你过滤掉不相关的信息。

那么问题就来了，即便是联网的 LLM，也经常基于互联网上过时或低质量的资料胡说八道，导致生成的代码或解释不准确。

所以，最好的方式可能是使用非联网模型，并基于第三方包的**官方文档**进行问答，这样能确保信息的准确性和时效性。

之前看到 @kangfenmao 大佬的 `cherry` 知识库工具很好用，可惜它主要依赖单个 URL 或站点的 `sitemap.xml` 来抓取内容。对于那些没有提供 `sitemap` 的文档站点，就比较难处理了。

因此，我写了一个简单的 Python 脚本，专门用于抓取这类文档型站点，并将内容保存为 Markdown 文件，方便后续用作 LLM 的知识库语料。

## 脚本

脚本的基本原理是：

1.  选择一个种子 URL (通常是文档首页或索引页) 作为起点。
2.  爬取种子 URL 页面，提取所有指向**相同域名**的内部链接。
3.  并发地爬取这些内部链接指向的页面。
4.  将每个页面的主要内容提取并转换为 Markdown 格式。
5.  根据原始 URL 的路径结构，将 Markdown 内容保存到本地对应的目录和文件中。

这个脚本使用了 `crawl4ai` 库，可以处理需要 JavaScript 渲染（客户端渲染）的站点。

### 主要功能与改进

*   **并发爬取:** 使用 `asyncio` 和信号量 (`Semaphore`) 控制并发爬取任务数量 (`--max-concurrent`)。
*   **并发写入:** 使用 `aiofiles` 库进行异步文件写入，提高效率。
*   **日志记录:** 将详细的运行日志记录到 `crawler.log` 文件中，方便排查问题。
*   **参数化:** 支持通过命令行参数配置种子 URL、输出目录、最大并发数、重试次数和最小内容字数阈值。
*   **帮助信息:** 提供 `-h` 或 `--help` 参数显示使用说明。

## 注意事项与局限性

虽然进行了一些改进，但脚本仍有一些需要注意的地方：

*   **并发控制:** 虽然加入了并发限制，但默认值 (`--max-concurrent=10`) 可能仍需根据目标网站的承受能力和你的网络环境进行调整。请**负责任地使用**，避免给目标服务器带来过大压力。
:::caution[请勿滥用]
过度频繁或高并发地爬取可能会对目标网站造成负担，甚至可能导致你的 IP 被封禁。请根据实际情况调整并发数，并尊重网站的 `robots.txt` (虽然本脚本目前未检查)。
:::
*   **错误处理:** 脚本包含基本的错误处理和重试机制，但可能无法覆盖所有异常情况。
*   **内容提取:** 依赖 `crawl4ai` 的 Markdown 提取能力，对于结构特别复杂的页面，提取效果可能不完美。
*   **测试有限:** 脚本主要在几个文档站点上进行了测试。在我的测试中，结合 `cherry` 知识库工具（使用 `r1` 问答模型和 `bge-m3` 嵌入模型），效果尚可。

## 使用说明

1.  **安装依赖:** 脚本基于 [crawl4ai](https://crawl4ai.com/mkdocs/) 库，使用前请确保已安装该库及其依赖：
    ```bash
    pip install crawl4ai aiofiles
    # 可能还需要安装 playwright 的浏览器驱动
    playwright install --with-deps
    ```
    :::tip[关于 crawl4ai]
    `crawl4ai` 是一个功能强大的库，专门用于爬取网页并提取内容，特别适合 AI 应用场景。推荐有兴趣的同学深入研究其[官方文档](https://crawl4ai.com/mkdocs/)。
    :::
2.  **获取脚本:** 将下面的 Python 代码保存到一个文件（例如 `scrape_docs.py`）。
3.  **运行脚本:**
    *   打开终端或命令行界面。
    *   使用 `python` 命令运行脚本，并提供必要的参数。
    *   **必需参数:**
        *   `url`: 种子 URL (例如，文档首页 `https://docs.example.com/`)。
        *   `output`: 输出目录路径。**重要:** 目录名**必须**以 `_docs` 结尾 (例如 `/path/to/your/output_docs`)。
    *   **可选参数:**
        *   `--max-concurrent`: 最大并发爬取任务数 (默认: 10)。
        *   `--retry-count`: 初始链接提取失败时的重试次数 (默认: 3)。
        *   `--min-word-count`: 提取页面 Markdown 内容的最小字数阈值 (默认: 100)。低于此字数的页面将被跳过。
    *   **查看帮助:**
        ```bash
        python scrape_docs.py -h
        ```
    *   **示例命令:**
        ```bash
        python scrape_docs.py https://docs.example.com/ /path/to/output_docs --max-concurrent 5 --min-word-count 50
        ```

```python
import asyncio
import os
import argparse
import logging
from typing import List, Optional, Dict, Any
from urllib.parse import urlparse, urljoin # 修正：添加 urljoin
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
import re
import traceback
from pathlib import Path
import aiofiles

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("crawler.log", encoding='utf-8')
    ]
)
logger = logging.getLogger(__name__)


async def extract_links(base_url: str, retry_count: int) -> List[str]:
    """使用Crawl4AI提取链接，并确保是同域链接

    Args:
        base_url: 要提取链接的基础URL
        retry_count: 重试次数

    Returns:
        提取到的同域链接列表
    """
    base_domain = urlparse(base_url).netloc
    processed_links = set()

    for attempt in range(retry_count):
        try:
            async with AsyncWebCrawler() as crawler:
                logger.info(f"Attempt {attempt + 1}: Extracting links from {base_url}")
                result = await crawler.arun(url=base_url)
                if not result or not result.success:
                    logger.warning(f"Failed to extract links from {base_url} on attempt {attempt + 1}")
                    continue # 继续下一次尝试

                # 提取内部链接
                links = result.links.get("internal", [])
                extracted_count = 0
                for link_data in links:
                    href = link_data["href"] if isinstance(link_data, dict) else link_data
                    if not href: continue

                    # 构造绝对 URL
                    absolute_url = urljoin(base_url, href)
                    parsed_absolute_url = urlparse(absolute_url)

                    # 检查是否是同域链接，且不是锚点链接，且未处理过
                    if parsed_absolute_url.netloc == base_domain and \
                       '#' not in href and \
                       absolute_url not in processed_links:
                        processed_links.add(absolute_url)
                        extracted_count += 1

                logger.info(f"Extracted {extracted_count} new same-domain links from {base_url} on attempt {attempt + 1}")
                # 即使某次尝试成功，也可能需要多次尝试来获取动态加载的链接，这里简化为一次成功即返回
                return list(processed_links)

        except Exception as e:
            logger.error(f"Attempt {attempt + 1} failed for {base_url}: {str(e)}")
            if attempt == retry_count - 1:
                logger.error(f"All attempts failed for {base_url}")
                return []
            await asyncio.sleep(2 ** attempt) # 指数退避

    logger.warning(f"Could not extract links from {base_url} after {retry_count} attempts.")
    return []


def sanitize_filename(filename):
    """清理文件名中的非法字符，并处理可能的空文件名"""
    sanitized = re.sub(r'[\\/*?:"<>|]', "", filename)
    # 如果清理后为空（例如 URL 是 /），则使用 index
    return sanitized if sanitized else "index"


async def save_markdown(mdContent, fullUrl, output_dir):
    """将结果保存为Markdown文件，处理路径和文件名

    Args:
        mdContent: Markdown内容
        fullUrl: 完整的URL
        output_dir: 输出目录路径
    """
    logger.info(f"🔄 Processing: {fullUrl}")

    if not mdContent or not mdContent.strip():
        logger.warning(f"❌ Empty content for {fullUrl}, skipping save.")
        return

    try:
        # 解析URL
        parsed_url = urlparse(fullUrl)
        url_path = parsed_url.path.strip("/")

        # 构建子目录路径
        subdir = Path(output_dir)
        path_parts = [part for part in url_path.split("/") if part] # 过滤空部分

        # 如果路径为空 (根路径), 直接放在输出目录下
        if not path_parts:
             filename_base = "index"
        # 如果路径最后一部分像文件名 (有.)，则前面的作为目录
        elif '.' in path_parts[-1]:
            filename_base = sanitize_filename(path_parts[-1].split('.')[0]) # 取点之前的部分
            path_parts = path_parts[:-1]
        # 否则，最后一部分也作为目录，文件名为 index
        else:
            filename_base = "index"

        # 创建子目录
        for part in path_parts:
            subdir = subdir / sanitize_filename(part)
        subdir.mkdir(parents=True, exist_ok=True) # 使用 parents=True 创建多级目录

        # 构建最终文件名和路径
        filename = f"{filename_base}.md"
        filepath = subdir / filename

        # 检查文件是否已存在（避免重复写入或覆盖）
        if filepath.exists():
            logger.info(f"⏩ File already exists, skipping: {filepath}")
            return

        # 异步写入文件
        async with aiofiles.open(filepath, "w", encoding="utf-8") as f:
            await f.write(f"# {fullUrl}\n\n{mdContent}") # 在开头添加原始URL作为标题
        logger.info(f"✅ Saved: {filepath}")

    except OSError as e:
        logger.error(f"❌ OS Error saving {fullUrl} to {filepath}: {e}")
    except Exception as e:
        logger.error(f"❌ Unexpected Error saving {fullUrl}: {e}\n{traceback.format_exc()}")


async def _handle_crawl_result(task, url, output_dir):
    """处理单个爬取任务的结果"""
    try:
        res = await task
        if res and res.success and res.markdown:
            await save_markdown(res.markdown, url, output_dir)
        elif res and not res.success:
             logger.warning(f"Crawling failed for {url}: {res.error_message}")
        elif not res:
             logger.warning(f"Crawling task for {url} returned None")
        else:
             logger.warning(f"No markdown content extracted from {url}")
    except Exception as e:
        logger.error(f"Error processing result for {url}: {e}\n{traceback.format_exc()}")


async def crawl_concurrently(urls: List[str], args: argparse.Namespace) -> None:
    """并发爬取多个URL

    Args:
        urls: 要爬取的URL列表
        args: 命令行参数
    """
    if not urls:
        logger.warning("No URLs provided for crawling.")
        return

    semaphore = asyncio.Semaphore(args.max_concurrent)
    tasks = []

    async def limited_crawl(url: str) -> Optional[Any]: # 返回类型改为 Any
        async with semaphore:
            try:
                # 每次爬取都创建一个新的 Crawler 实例可能更稳定
                async with AsyncWebCrawler(
                    config=BrowserConfig(headless=True, text_mode=True),
                ) as crawler:
                    config = CrawlerRunConfig(
                        word_count_threshold=args.min_word_count, # 使用参数
                        wait_until="networkidle",
                        page_timeout=120000, # 2 minutes
                    )
                    logger.info(f"🔗 Crawling: {url}")
                    result = await crawler.arun(
                        url=url,
                        config=config,
                    )
                    return result
            except Exception as e:
                logger.error(f"Exception during crawl of {url}: {str(e)}\n{traceback.format_exc()}")
                return None # 返回 None 表示失败

    for url in urls:
        # 创建爬取任务
        crawl_task = asyncio.create_task(limited_crawl(url))
        # 添加回调，用于处理爬取结果（无论成功或失败）
        crawl_task.add_done_callback(
            lambda t, current_url=url: asyncio.create_task(
                _handle_crawl_result(t, current_url, args.output)
            )
        )
        tasks.append(crawl_task)

    # 等待所有爬取任务完成（包括回调）
    if tasks:
        await asyncio.gather(*tasks, return_exceptions=True) # 收集异常而不是让 gather 失败
    logger.info("Concurrent crawling phase finished.")


async def main():
    # 解析命令行参数
    parser = argparse.ArgumentParser(description="Documentation Website Crawler for LLM Corpus")
    parser.add_argument("url", type=str, help="Seed URL (e.g., documentation homepage)")
    parser.add_argument("output", type=str, help="Output directory (must end with '_docs')")
    parser.add_argument("--max-concurrent", type=int, default=10, help="Maximum concurrent crawling tasks (default: 10)")
    parser.add_argument("--retry-count", type=int, default=3, help="Retry count for initial link extraction (default: 3)")
    parser.add_argument("--min-word-count", type=int, default=100, help="Minimum word count threshold for extracting markdown (default: 100)") # 添加最小字数阈值
    args = parser.parse_args()

    # 验证输出目录
    if not args.output.endswith("_docs"):
        logger.error("❌ Output directory must end with '_docs'")
        print("Error: Output directory must end with '_docs'")
        return

    # 创建输出目录
    try:
        os.makedirs(args.output, exist_ok=True)
        logger.info(f"Output directory set to: {args.output}")
    except OSError as e:
        logger.error(f"❌ Failed to create output directory {args.output}: {e}")
        print(f"Error: Failed to create output directory {args.output}: {e}")
        return

    # 步骤1：提取种子 URL 的同域链接
    logger.info(f"🔍 Step 1: Extracting same-domain links from seed URL: {args.url}")
    links = await extract_links(args.url, args.retry_count)

    if not links:
        logger.error(f"❌ No links extracted from {args.url}. Exiting.")
        print(f"Error: No links extracted from {args.url}. Please check the URL and network connection.")
        return

    logger.info(f"📥 Found {len(links)} potential same-domain links.")
    # 可选：打印部分链接用于调试
    # logger.debug(f"Sample links: {links[:5]}")

    # 步骤2：并发爬取提取到的链接并保存 Markdown
    logger.info(f"🚀 Step 2: Starting concurrent crawling of {len(links)} links...")
    await crawl_concurrently(links, args)

    logger.info("🎉 All crawling tasks initiated. Check logs for progress and errors.")
    print("🎉 Crawling process finished. Check crawler.log for details.")


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        logger.info("\n🛑 Crawler stopped by user (KeyboardInterrupt).")
        print("\n🛑 Crawler stopped.")
    except Exception as e:
        logger.critical(f"An unexpected error occurred in main: {e}\n{traceback.format_exc()}")
        print(f"An unexpected error occurred: {e}")

```
