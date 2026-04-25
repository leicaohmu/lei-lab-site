# Lei-Lab Knowledge Platform

课题组知识共享平台，基于 [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 构建。

## 本地开发

```bash
# 安装依赖
pip install mkdocs-material mkdocstrings[python]
pip install histox

# 本地预览
mkdocs serve

# 浏览器打开 http://127.0.0.1:8000
```

## 部署

推送到 `main` 分支后，GitHub Actions 自动部署到 GitHub Pages。

```bash
git push origin main
```

## 自动同步 histox 文档

当 [histox](https://github.com/leicaohmu/histox) 库更新并推送到 main 或发布新版本时，
文档站会自动触发重新部署，无需手动操作。

详见 `for-histox-repo/trigger-docs.yml`。

## 目录结构

```
lei-lab-site/
├── mkdocs.yml                        # 站点配置
├── docs/
│   ├── index.md                      # 首页
│   ├── assets/                       # 静态资源（logo 等）
│   ├── courses/                      # 课程
│   ├── tools/                        # 演示工具
│   ├── api/                          # API 文档
│   │   └── histox/                   # histox 库文档
│   ├── papers/                       # 文献抄读
│   ├── blog/                         # 博客
│   └── team/                         # 课题组介绍
├── .github/workflows/
│   └── deploy.yml                    # 自动部署 CI
└── for-histox-repo/
    └── trigger-docs.yml              # 复制到 histox 仓库的触发文件
```
