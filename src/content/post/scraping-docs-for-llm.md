背景
平常开发的时候经常要阅读第三方包文档。通常是粗读整个文档，对各个模块有概念了以后就上手写，写的时候再去翻文档找调用细节。

这里面其实有两个点可以用 ai 加速，一个是让它介绍开发包，一个是让它基于第三方包直接实现某个功能函数。而且大部分情况下，并不需要用到第三方包全部的功能，ai 可以很好的帮你过滤不需要的信息。

那么问题就来了，即便是联网模型，也经常基于互联网上过时或低质量资料胡说八道。

所以最好的方式就是用非联网模型基于第三方包的官方文档进行问答。

@kangfenmao 大佬的 cherry 的知识库很好用，可惜只能抓单个url，或者抓站点的 sitemap，有些没有 sitemap 的站点就难受了

所以我糊了一个脚本，去抓文档型站点，生成 markdown 作为姿势库。

脚本
脚本原理是选择一个种子 url 作为起点，爬取种子 url 下所有相同域名的 url，开爬，生成 md 文件，可以支持客户端渲染的站点。

已知的的问题：
没有做并发限制，因为爬的都是静态文档站，而且种子url中的 url 不会太多吧？各位用的时候悠着点，:rofl::rofl::rofl:
写文件没有使用类似于 aiofiles 库做并发处理。
我抓了两个站点做测试，客户端用 cherry ，问答用 r1 ，嵌入模型用 beg-m3 ，效果不错。不知道 @kangfenmao 大佬未来考虑支持 rerank 不:rofl:

使用说明
脚本基于 https://crawl4ai.com/mkdocs/ ，使用前请自行安装。（这个库很强大，各位佬有兴趣请研究:rofl:）
脚本极其不完善，本着抛砖引玉的目的发出来的，轻喷 :rofl:
自行更改第10行种子 url 和 第11行输出目录即可运行
``` python
import asyncio
import os
import argparse
import logging
from typing import List, Optional, Dict, Any
from urllib.parse import urlparse
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
        logging.FileHandler("crawler.log", encoding='utf-8')  # 添加编码参数
    ]
)
logger = logging.getLogger(__name__)


async def extract_links(url: str, retry_count: int) -> List[str]:
    """使用Crawl4AI提取链接
    
    Args:
        url: 要提取链接的URL
        retry_count: 重试次数
        
    Returns:
        提取到的链接列表
    """
    for attempt in range(retry_count):
        try:
            async with AsyncWebCrawler() as crawler:
                result = await crawler.arun(url=url)
                if not result.success:
                    logger.warning(f"Failed to extract links from {url}")
                    return []

                # 提取内部链接
                links = result.links.get("internal", [])

                # 将links去重, 并处理可能的字典类型
                unique_links = list(
                    set([link["href"] if isinstance(link, dict) else link for link in links])
                )
                logger.info(f"Extracted {len(unique_links)} links from {url}")
                return unique_links
        except Exception as e:
            logger.error(f"Attempt {attempt + 1} failed for {url}: {str(e)}")
            if attempt == retry_count - 1:
                logger.error(f"All attempts failed for {url}")
                return []
            await asyncio.sleep(2 ** attempt)  # 指数退避


def sanitize_filename(filename):
    """清理文件名中的非法字符"""
    return re.sub(r'[\\/*?:"<>|]', "", filename)


async def save_markdown(mdContent, fullUrl, output_dir):
    """将结果保存为Markdown文件
    Args:
        mdContent: Markdown内容
        fullUrl: 完整的URL
        output_dir: 输出目录路径
    """
    print(f"🔄 Processing: {fullUrl}")

    if not mdContent:
        print("❌ Empty content, skipping")
        return

    # 解析URL，提取路径部分，忽略查询参数和锚点
    parsed_url = urlparse(fullUrl)
    url_path = parsed_url.path.strip("/")  # 去掉首尾的斜杠
    path_parts = url_path.split("/")  # 分割路径

    # 构建子目录路径
    subdir = Path(output_dir)
    for part in path_parts[:-1]:  # 除了最后一部分作为文件名
        if part:  # 跳过空的部分（例如双斜杠情况）
            subdir = subdir / part
            subdir.mkdir(exist_ok=True)  # 创建子目录

    # 构建文件名
    filename = path_parts[-1] + ".md"
    filepath = subdir / filename

    # 处理文件名冲突
    if filepath.exists():
        return

    # 异步写入文件
    try:
        async with aiofiles.open(filepath, "w", encoding="utf-8") as f:
            await f.write(mdContent)
        print(f"✅ Saved: {filepath}")
    except Exception as e:
        print(f"❌ Error saving {filepath}: {e}")


async def _handle_crawl_result(task, url, output_dir):
    try:
        res = await task
        await save_markdown(res.markdown, url, output_dir)
    except Exception as e:
        print(f"Error processing {url}: {e}")
        print("Traceback:")
        print(traceback.format_exc())


async def crawl_concurrently(urls: List[str], args: argparse.Namespace) -> None:
    """并发爬取多个URL
    
    Args:
        urls: 要爬取的URL列表
        args: 命令行参数
    """
    semaphore = asyncio.Semaphore(args.max_concurrent)
    
    async def limited_crawl(url: str) -> Optional[Dict[str, Any]]:
        async with semaphore:
            try:
                async with AsyncWebCrawler(
                    config=BrowserConfig(headless=True, text_mode=True),
                ) as crawler:
                    config = CrawlerRunConfig(
                        word_count_threshold=200,
                        wait_until="networkidle",
                        page_timeout=120000,
                    )
                    logger.info(f"🔗 Crawling: {url}")
                    result = await crawler.arun(
                        url=url,
                        config=config,
                    )
                    return result
            except Exception as e:
                logger.error(f"Error crawling {url}: {str(e)}")
                return None

    tasks = []
    for url in urls:
        task = asyncio.create_task(limited_crawl(url))
        task.add_done_callback(
            lambda t, url=url: asyncio.create_task(
                _handle_crawl_result(t, url, args.output)
            )
        )
        tasks.append(task)
    
    await asyncio.gather(*tasks)


async def main():
    # 解析命令行参数
    parser = argparse.ArgumentParser(description="网页爬虫")
    parser.add_argument("url", type=str, help="种子URL")
    parser.add_argument("output", type=str, help="输出目录（必须以_docs结尾）")
    parser.add_argument("--max-concurrent", type=int, default=10, help="最大并发任务数，默认10")
    parser.add_argument("--retry-count", type=int, default=3, help="重试次数，默认3")
    args = parser.parse_args()

    # 验证输出目录
    if not args.output.endswith("_docs"):
        print("❌ 输出目录必须以'_docs'结尾")
        return

    # 创建输出目录
    os.makedirs(args.output, exist_ok=True)

    # 步骤1：提取侧边栏链接
    print("🔍 Extracting sidebar links...")
    links = await extract_links(args.url, args.retry_count)
    print(f"📥 Found {len(links)} links")

    # 步骤2：并发爬取并保存
    print("🚀 Starting concurrent crawling...")
    await crawl_concurrently(links, args)

    print("🎉 All done!")


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n🛑 爬虫已停止")
```