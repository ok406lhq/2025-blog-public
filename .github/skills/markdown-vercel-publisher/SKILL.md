---
name: markdown-vercel-publisher
description: Use when the user wants to publish a new Markdown file or text on the internet through the current Vercel site, asks to "展示这个 md", "把这个文件挂到网上", "快速生成一个链接", "发布 markdown", "挂到 vercel", "生成可访问链接", "把文章放到博客里", or asks to expose a local Markdown document as a shareable public URL.
---

# Markdown Vercel Publisher

将用户提供的 Markdown 内容，尽快挂到当前工作区已部署的站点里，并返回可分享链接。

## 目标

- 优先走最短路径，不重新设计新页面
- 优先复用现有 public 目录、博客路由、Markdown 渲染能力
- 同时提供“原始 Markdown 直链”和“适合分享的网页链接”时，优先两者都做
- 如果仓库没有现成博客渲染链路，至少保证原始 Markdown 可公开访问

## 默认工作流

### 1. 先检查现有站点结构

优先读取这些位置，确认最快落点：

- README.md
- package.json
- src/app 下是否已有 blog、share、write、about 等页面
- public 目录结构，尤其是 public/blogs、public/files
- 任何现成的 Markdown 加载或渲染逻辑，比如 load-blog、markdown-renderer

重点判断：

- 是否已有 public/blogs/{slug}/index.md + config.json 的约定
- 是否已有 /blog/[id] 一类页面可直接渲染 Markdown
- 是否已有 public 下的静态文件直出能力
- 是否能从 README、环境变量或代码中推断线上域名

### 2. 决定发布路径

按下面顺序选最省事方案：

1. 如果已有博客 Markdown 路由：
   - 创建 public/blogs/{slug}/index.md
   - 创建 public/blogs/{slug}/config.json
   - 更新 public/blogs/index.json
   - 如项目依赖分类列表且确有需要，再检查 categories.json 是否需要调整

2. 如果 public 可静态暴露：
   - 创建 public/files/{slug}.md 作为原始 Markdown 直链

3. 如果两条都可行：
   - 两条都做
   - 网页版用于转发阅读
   - 原始 md 用于直接下载或引用

除非用户明确要求，不要额外新建复杂页面、编辑器功能或上传系统。

## slug 规则

- 优先使用用户给的文件名，转为安全 slug
- 去掉扩展名、空格和明显不适合作为路径的字符
- 保持简短、稳定、可复用
- 如果与现有 slug 冲突，再加短后缀

如果用户明确给了标题，可在 config.json 中使用更自然的中文标题；slug 不必追求完全一致。

## 推荐输出物

### A. 原始 Markdown 直链

创建：

- public/files/{slug}.md

用途：

- 最快生成可公开 URL
- 适合“只要一个能打开的链接”

### B. 网页版文章

创建：

- public/blogs/{slug}/index.md
- public/blogs/{slug}/config.json
- public/blogs/index.json 中追加一项

config.json 一般至少包含：

```json
{
  "title": "文章标题",
  "tags": ["标签1"],
  "date": "2026-04-01",
  "summary": "一句话摘要"
}
```

如果用户给的是原始吐槽稿、草稿或对话式内容，可在不改变核心意思的前提下，整理为更适合网页阅读的版本。原稿则保留在 public/files。

## 更新索引时的要求

- 保持现有 JSON 风格和字段顺序
- 只做最小改动
- 不要重排整个数组
- 不要顺手修 unrelated 内容
- 如果索引项目依赖 category，沿用仓库已有值

## 验证

完成后至少做这些检查：

1. 校验新增 JSON 文件合法
2. 校验 public/blogs/index.json 合法
3. 如果能快速验证，再检查项目错误面板里是否有与你改动相关的新错误

可用轻量方式，例如 Node 直接 JSON.parse。

## 链接生成规则

最终优先返回两类链接：

- https://<domain>/files/{slug}.md
- https://<domain>/blog/{slug}

域名来源按这个顺序判断：

1. 用户明确给出的生产域名
2. 仓库里配置的 SITE_URL 或 NEXT_PUBLIC_SITE_URL
3. README 中示例或已知 Vercel 域名
4. 如果无法确认真实域名，只返回路径格式，并明确说明需要等待 Vercel 部署完成后替换域名

## 回答方式

最终回答应简洁，明确告诉用户：

- 已创建哪些文件
- 部署后可访问的链接格式
- 如果当前无法知道真实线上域名，直接说明这一点，不要编造
- 如果只是本地改动，提醒用户 push 后等待 Vercel 自动部署

## 不该做的事

- 不要为了发布一个 md 去重构整站
- 不要默认新增数据库、CMS、上传服务
- 不要改动与当前发布流程无关的页面
- 不要假设用户一定想要复杂排版，优先先让链接可用

## 这类请求的典型表达

- “把这个 md 展示在互联网”
- “我有个新的 markdown，帮我挂到 vercel 上”
- “快速生成一个公开链接”
- “把这个文件变成网页链接”
- “把这篇文章放进当前博客里”
- “给我一个能直接访问的地址”