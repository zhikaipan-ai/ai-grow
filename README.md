# AI Grow V1.1 · 全球 AI 情报 + 轻量学习

一款以 iPhone 为核心体验的个人 AI 学习 PWA。**V1.1 的重点是真实来源的资讯和可部署的每日自动采集，而非系统推送。**

## 直接体验

直接下载根目录旁的 `AI_Grow_V1.1_直接体验.html` 并用浏览器打开。它包含完整内嵌样式和脚本、以及 2026-10-09 已核对过原文的真实资讯快照。无需登录，不用联网也能浏览初始资讯和已内置课程。

**注意：** 下载版只是静态日期快照；浏览器出于安全限制，不会在本地 `file://` 下直接请求 RSS，不能做到每天更新。截图 OCR 需要联网加载 Tesseract.js。建议正式使用前部署多文件版。

## V1.1 完成功能

- 今日 AI 情报：真实且可查证的新闻原文、发布日期、中文标题摘要及学习角度，主题筛选、收藏到复习库、手动更新入口与“资讯可能已过时”提醒。
- 定时采集：Python 脚本从官方/媒体的多家 RSS 采集新闻，日期去重、验证链接、统一格式；GitHub Actions 每天**北京时间 08:30** 尝试更新，并自动部署 GitHub Pages。上传 `main` 分支后也会触发首次部署。
- 中文摘要：如果在 GitHub Actions **Secrets** 中设置 `OPENAI_API_KEY`，服务器侧使用 OpenAI API 将最新 RSS 标题和摘要准确改写为简体中文（可能产生 API 费用）。不设置密钥也能自动更新英文原文标题/摘要，且会明确标注没有中文整理。**绝对不要把密钥写进 app.js、index.html 或 news.json。**
- V1.0 不变：14 节课程、互动小测、成长 XP、六大能力模块、每周自由完成 3 次实践；收藏粘贴/标签/图片 OCR、间隔复习与 JSON 备份。
- 同一个浏览器和相同网站域名上，沿用 `ai-grow-personal-v1` 本地存储；课程、笔记和 XP 数据结构不变。

## 上线自动更新（GitHub Pages，步骤）

1. 在 GitHub 新建一个 **private 或 public** 仓库（需要可运行 GitHub Actions 且支持 Pages；可选择 public 仓库便于入门）。
2. **把 `AI_Grow_V1_1/` 文件夹里的全部内容上传到仓库根目录**，包含隐藏的 `.github/workflows/daily-news.yml`、`scripts/update_news.py` 和 `news.json`；不要只上传单个 HTML。`index.html` 必须在仓库根目录。
3. 打开仓库 Settings → Pages → Build and deployment → **Source: GitHub Actions**。
4. 上传 `main` 分支后应会自动部署；也可打开 Actions →「AI Grow 每日资讯更新与网页发布」→ Run workflow 手动运行。如果工作流提示没有写入仓库权限，在 Settings → Actions → General 把 Workflow permissions 调整为 Read and write permissions（或按仓库策略授予必要权限）。
5. 在 Actions 的成功记录里点击部署网址，即可访问可安装的 PWA。之后定时任务会尝试每日更新并重新发布页面。GitHub Actions 的 schedule 会因排队而延迟，**不保证准时 08:30**。
6. （可选，推荐）Settings → Secrets and variables → Actions → New repository secret：名称 `OPENAI_API_KEY`，值为你自己的 API Key。这样**新采集的**英语新闻才会被自动整理成中文。

> 如果你想用其他时区，请调整 `.github/workflows/daily-news.yml` 的 `cron`；当前示例按北京时间 UTC+8 计算（`30 0 * * *`）。

## 现在运行手动资讯采集

需要能访问互联网的电脑或 GitHub Actions 环境；无需安装第三方 Python 依赖。

```bash
python3 scripts/update_news.py
```

脚本仅更新 `news.json`，**不是系统通知或聊天机器人**。当全部来源访问失败、或没有最近 21 天有效文章时，脚本会返回错误，保留旧简报不覆盖。

## 当前限制与提醒

- **没有主动系统推送 / iPhone 通知权限流程：** 网页里的“08:30”代表部署后资讯抓取目标时间，不代表手机通知时间。iOS 主屏幕应用真正的 Web Push 需要独立服务端推送方案及用户授权；目前未交付。
- RSS 内容来自各网站，抓取质量、更新速度、内容完整性视来源而定，不承诺覆盖每条重大新闻。简报会显示自己的生成时间，用户仍应阅读原文验证关键事实。
- **如果没有 OPENAI_API_KEY**，最新 RSS 内容将保留英语标题/摘要，而非生成未经证实的“中文解读”；初始快照是人工核验后的中文内容。
- 本地学习记录只保存在该浏览器的 localStorage，不跨设备同步。`file://` 和 HTTPS 属于不同存储环境；迁移时从旧版导出 JSON 后到新版导入。清理浏览器数据前请备份。
- 图片 OCR 为浏览器端处理，需联网加载外部识别组件，提取结果需要人工校对。

## 数据结构

`news.json`：`{version, generated_at, source_mode, items:[{id, source, category, title, summary, why, url, date, publishedAt, translated}]}`。页面只从这个 JSON 加载安全的 http(s) 链接、原始发布日期及摘要；出现加载错误时使用上次的内置快照。

## 核心文件

- `index.html`, `styles.css`, `app.js`：多文件网页应用。
- `news.json`：资讯快照 + 自动更新后数据文件。
- `scripts/update_news.py`：RSS 获取与可选中文整理脚本。
- `.github/workflows/daily-news.yml`：每日定时更新及 GitHub Pages 部署。
- `sw.js`, `manifest.webmanifest`, `icons/`：可安装 PWA 资源。

### 上线简明步骤

见 `上线操作说明.md`，包含上传、Pages 设置、时区、iPhone 主屏幕以及隐私提醒。
