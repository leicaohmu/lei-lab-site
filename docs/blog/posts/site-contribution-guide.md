---
title: 课题组网站协作指南
date: 2026-05-08
authors:
  - lei
categories:
  - 教程
tags:
  - Git
  - GitHub
  - MkDocs
description: 本文介绍如何通过 Git + GitHub PR 流程向课题组网站提交博客文章，适用于所有组员。
---

# 课题组网站协作指南

> **技术栈**：MkDocs Material + GitHub + Cloudflare Pages

本文介绍如何向课题组网站提交内容。为了保证网站稳定，所有内容更新都通过 **"分支开发 → PR 审核 → 合并发布"** 的流程进行。

<!-- more -->

---

## 一、整体架构

```
GitHub 仓库 (leicaohmu/lei-lab-site)
    │
    ├── main 分支  ──→  Cloudflare 自动构建  ──→  正式网站
    │
    └── feature 分支  ──→  Cloudflare 自动构建  ──→  预览网站（临时）
```

每次 `main` 分支有新 commit，Cloudflare 会**自动触发构建并发布**，无需手动操作。

---

## 二、首次使用：环境准备

确保本地已安装 Git，然后克隆仓库并安装依赖：

```bash
# 克隆仓库
git clone https://github.com/leicaohmu/lei-lab-site.git
cd lei-lab-site

# 安装依赖（建议在 conda/venv 环境中执行）
pip install mkdocs-material

# 本地预览
mkdocs serve
# 浏览器打开 http://127.0.0.1:8000
```

---

## 三、组员：提交博客的完整流程

### Step 1：拉取最新代码

每次开始写新内容前，先同步远程最新状态：

```bash
git checkout main
git pull origin main
```

### Step 2：新建分支

!!! warning "不要在 main 分支上直接修改"
    仓库开启了分支保护，直接 push main 会被拒绝。请务必新建分支。

```bash
git checkout -b docs/你的内容名称
# 例如：
git checkout -b docs/ctranspath-paper-reading
```

### Step 3：写博客

在 `docs/blog/posts/` 目录下新建 `.md` 文件，参考已有文章的格式编写。

文章头部需要包含 Front Matter：

```yaml
---
title: 文章标题
date: 2026-05-08
authors:
  - yourname
categories:
  - 文献抄读
---
```

本地预览效果：

```bash
mkdocs serve
```

### Step 4：提交并推送

```bash
git add .
git commit -m "docs: add ctranspath paper reading"
git push origin docs/ctranspath-paper-reading
```

### Step 5：发起 Pull Request

1. 打开 [GitHub 仓库页面](https://github.com/leicaohmu/lei-lab-site)
2. 点击黄色提示条 **"Compare & pull request"**
3. 填写简要说明，点击 **"Create pull request"**
4. Cloudflare 会自动在 PR 评论区生成**预览链接**，可发给 Lei 确认排版
5. **等待 Lei 审核通过后合并**，网站自动更新 ✅

### Step 6：合并后清理

```bash
git checkout main
git pull origin main

# 可选：删除已合并的本地分支
git branch -d docs/ctranspath-paper-reading
```

---

## 四、分支命名规范

| 前缀 | 用途 | 示例 |
| :--- | :--- | :--- |
| `docs/` | 写文章、改文档 | `docs/histox-blog` |
| `feat/` | 新增功能或插件 | `feat/add-search` |
| `fix/` | 修复错误 | `fix/broken-link` |
| `update/` | 批量内容更新 | `update/homepage` |

---

## 五、常见问题

??? question "push 被拒绝，提示 protected branch？"
    你在 `main` 分支上直接 push 了。先把改动搬到新分支：
    ```bash
    git checkout -b fix/my-change
    git push origin fix/my-change
    ```
    然后正常发 PR 即可。

??? question "本地改了很多，忘记新建分支怎么办？"
    不用担心，直接从当前状态新建分支，所有改动会自动带过去：
    ```bash
    git checkout -b docs/my-changes
    git push origin docs/my-changes
    ```
    然后正常发 PR 即可。

??? question "PR 显示 Review required / Merging is blocked？"
    需要维护者（Lei）在 GitHub 上点 **Approve** 并 **Merge**，请联系 Lei 处理。

??? question "如何查看 Cloudflare 预览链接？"
    PR 发起后，Cloudflare 机器人会自动在 PR 评论区贴出预览链接。也可登录
    [Cloudflare Dashboard](https://dash.cloudflare.com/) 查看构建状态。

??? question "和别人改了同一个文件，出现冲突怎么办？"
    GitHub 会在 PR 页面提示 **"Conflicts"**。在本地执行：
    ```bash
    git merge main
    # 手动编辑冲突文件，解决后：
    git add .
    git commit -m "fix: resolve conflicts"
    git push
    ```

---

## 六、一句话总结

> **先拉取，再开分支；写完推，发起 PR；Lei 审核，点合并；网站自动变。** 🎉