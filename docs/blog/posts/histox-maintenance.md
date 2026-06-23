---
title: HisToX多人维护与发布指南
date: 2026-05-05
authors:
  - lei
categories:
  - 开发笔记
tags:
  - histox
  - 插件开发
  - Python
description: 记录 histox 核心库多人维护和发布流程。
---

# histox 多人维护与发布教程

本文档用于说明 `histox` 项目的多人协作开发、版本管理、TestPyPI 测试发布、正式 PyPI 发布以及 GitHub tag / release 的推荐流程。
<!-- more -->

本项目推荐采用：

```text
feature branch -> Pull Request -> master -> release PR -> TestPyPI -> tag -> PyPI -> GitHub Release
```

核心原则：

```text
多人可以参与开发，但发布权限需要严格控制。
```

---

# 1. 角色分工

多人维护 Python 包时，建议区分以下角色。

## 1.1 普通开发者 / Contributor

普通开发者可以：

- 创建功能分支
- 修改代码
- 提交 Pull Request
- 参与 code review
- 编写测试和文档

普通开发者不建议：

- 直接 push 到 `master` / `main`
- 直接发布 TestPyPI / PyPI
- 修改 GitHub Secrets
- 强制修改已有 tag
- 在本地使用 PyPI token 上传包

## 1.2 维护者 / Maintainer

维护者可以：

- 审查 Pull Request
- 合并 Pull Request
- 管理 issue
- 创建 release 分支
- 参与版本发布准备

## 1.3 发布负责人 / Release Manager

发布负责人一般由 1-2 个核心维护者担任。

发布负责人负责：

- 确定新版本号
- 更新版本号
- 更新 changelog
- 发布到 TestPyPI
- 验证 TestPyPI 安装
- 创建 GitHub tag
- 发布到正式 PyPI
- 创建 GitHub Release

---

# 2. 仓库权限建议

为了避免误操作，建议开启 GitHub 分支保护。

路径：

```text
GitHub 仓库 -> Settings -> Branches -> Branch protection rules
```

对 `master` 或 `main` 分支设置保护规则。

建议开启：

```text
Require a pull request before merging
Require approvals
Require status checks to pass before merging
Do not allow force pushes
Do not allow deletions
```

这样可以避免未经审查的代码直接进入主分支。

---

# 3. 日常开发流程

假设主分支是 `master`。

开发者不要直接在 `master` 分支上开发，而是从 `master` 创建新的功能分支。

## 3.1 更新本地主分支

```bash
git checkout master
git pull origin master
```

## 3.2 创建功能分支

分支命名建议：

```text
feature/功能名称
fix/问题名称
docs/文档名称
refactor/重构名称
```

例如：

```bash
git checkout -b feature/add-new-loader
```

或者：

```bash
git checkout -b fix/import-error
```

## 3.3 修改代码并提交

修改代码后，先查看变更：

```bash
git status
```

添加文件：

```bash
git add .
```

提交：

```bash
git commit -m "feat: add new loader"
```

常见 commit message 前缀：

```text
feat:     新功能
fix:      修复 bug
docs:     文档修改
style:    代码格式修改，不影响逻辑
refactor: 重构
test:     测试相关
chore:    构建、依赖、CI 等杂项修改
release:  版本发布相关
```

## 3.4 推送分支到 GitHub

```bash
git push origin feature/add-new-loader
```

然后在 GitHub 页面创建 Pull Request：

```text
feature/add-new-loader -> master
```

---

# 4. Pull Request 审查流程

Pull Request 创建后，维护者需要检查：

- 代码逻辑是否正确
- 是否破坏现有 API
- 是否需要增加测试
- 是否需要更新 README
- 是否需要更新依赖
- 是否影响打包和发布
- 是否符合项目风格

对于普通功能 PR，通常不要在每个 PR 中修改版本号。

例如这些 PR 可以先合并：

```text
feat: add image loader
fix: correct import bug
docs: update installation guide
refactor: simplify preprocessing code
```

等准备发版时，再由发布负责人统一修改版本号。

---

# 5. 版本号规则

本项目建议使用语义化版本号：

```text
主版本.次版本.修订版本
```

例如：

```text
0.2.1
```

## 5.1 修复 bug

如果只是修复 bug：

```text
0.2.1 -> 0.2.2
```

## 5.2 新增功能

如果新增功能，但没有破坏兼容性：

```text
0.2.1 -> 0.3.0
```

## 5.3 不兼容修改

如果修改会破坏旧代码兼容性：

```text
0.2.1 -> 0.3.0
```

如果项目已经稳定到 1.x，可以使用：

```text
1.2.3 -> 2.0.0
```

## 5.4 重要规则

PyPI 和 TestPyPI 都不允许重复上传同一个版本。

例如，如果已经发布过：

```text
histox==0.2.1
```

那么不能再次上传另一个 `0.2.1`。

下次发布必须改成新版本，例如：

```text
histox==0.2.2
```

---

# 6. 准备发布新版本

假设当前版本是：

```text
0.2.1
```

准备发布：

```text
0.2.2
```

发布负责人先从最新 `master` 创建 release 分支。

## 6.1 创建 release 分支

```bash
git checkout master
git pull origin master
git checkout -b release/v0.2.2
```

## 6.2 修改版本号

检查版本号在哪些文件中定义。

常见位置包括：

```text
setup.py
pyproject.toml
histox/__init__.py
```

如果项目使用 `setup.py`，则修改：

```python
version="0.2.2",
```

如果 `histox/__init__.py` 中有：

```python
__version__ = "0.2.1"
```

也要同步改为：

```python
__version__ = "0.2.2"
```

## 6.3 更新 CHANGELOG

如果项目有 `CHANGELOG.md`，建议添加：

```markdown
## 0.2.2 - 2026-06-23

### Added
- Add xxx feature.

### Fixed
- Fix xxx bug.

### Docs
- Update README usage.
```

如果项目暂时没有 `CHANGELOG.md`，可以先跳过，但长期建议维护。

## 6.4 本地构建检查

在发布前建议本地构建并检查包。

```bash
rm -rf dist build *.egg-info
python -m pip install --upgrade build twine
python -m build
python -m twine check dist/*
```

如果输出类似：

```text
PASSED
```

说明包的元数据基本正常。

## 6.5 提交 release 分支

```bash
git add .
git commit -m "release: bump version to 0.2.2"
git push origin release/v0.2.2
```

然后在 GitHub 创建 Pull Request：

```text
release/v0.2.2 -> master
```

PR 标题建议：

```text
release: bump version to 0.2.2
```

---

# 7. 合并 release PR

release PR 需要至少一名维护者 review。

合并前确认：

- 版本号已更新
- changelog 已更新
- README 没有明显错误
- CI 测试通过
- 打包配置没有问题

合并后，确保本地 `master` 是最新的：

```bash
git checkout master
git pull origin master
```

---

# 8. 发布到 TestPyPI

本项目使用 GitHub Actions 发布到 TestPyPI。

工作流文件通常是：

```text
.github/workflows/publish-to-test-pypi.yml
```

推荐触发方式：

```yaml
on:
  workflow_dispatch:
```

这表示只有手动点击时才会发布，普通 `git push` 不会自动发布。

## 8.1 手动运行 TestPyPI workflow

打开 GitHub 仓库页面：

```text
Actions -> publish to TestPyPI -> Run workflow
```

分支选择：

```text
master
```

然后点击运行。

## 8.2 确认 workflow 成功

在 GitHub Actions 日志中确认：

- Build distributions 成功
- Publish distribution to TestPyPI 成功
- 没有认证错误
- 没有版本重复错误

如果出现版本重复错误，例如：

```text
File already exists
```

说明这个版本号已经上传过，必须提升版本号后重新发布。

---

# 9. 验证 TestPyPI 安装

TestPyPI 发布成功后，建议创建干净环境测试安装。

## 9.1 创建虚拟环境

使用 venv：

```bash
python -m venv histox-test
source histox-test/bin/activate
python -m pip install --upgrade pip
```

或者使用 conda：

```bash
conda create -n histox-test python=3.9 -y
conda activate histox-test
python -m pip install --upgrade pip
```

## 9.2 从 TestPyPI 安装

以 `0.2.2` 为例：

```bash
pip install --no-cache-dir \
  --index-url https://test.pypi.org/simple/ \
  --extra-index-url https://pypi.org/simple \
  histox==0.2.2
```

说明：

```text
--index-url https://test.pypi.org/simple/
```

表示从 TestPyPI 安装 `histox`。

```text
--extra-index-url https://pypi.org/simple
```

表示依赖包可以从正式 PyPI 安装。

## 9.3 测试 import

```bash
python - <<'PY'
import histox

print("histox import OK")
print(histox)

if hasattr(histox, "__version__"):
    print("histox version:", histox.__version__)
else:
    print("histox has no __version__ attribute")
PY
```

## 9.4 测试命令行入口

如果项目提供 CLI，可以测试：

```bash
histox --help
```

或者：

```bash
python -m histox --help
```

如果 TestPyPI 安装和基本功能都正常，就可以进入正式发布流程。

---

# 10. 创建 Git tag

Git tag 用来标记某个版本对应的代码快照。

例如：

```text
v0.2.2
```

表示 PyPI 上的 `histox==0.2.2` 对应 GitHub 上的某个 commit。

## 10.1 创建 tag

在本地确认 `master` 最新：

```bash
git checkout master
git pull origin master
```

创建 tag：

```bash
git tag v0.2.2
```

推送 tag 到 GitHub：

```bash
git push origin v0.2.2
```

## 10.2 不要修改旧 tag

不建议强制更新已有 tag。

不推荐：

```bash
git tag -f v0.2.2
git push origin -f v0.2.2
```

如果发布后发现问题，正确做法是发布新版本：

```text
0.2.2 -> 0.2.3
```

然后创建新 tag：

```bash
git tag v0.2.3
git push origin v0.2.3
```

---

# 11. 发布到正式 PyPI

TestPyPI 验证通过，并且 tag 已创建后，可以发布到正式 PyPI。

正式 PyPI workflow 通常是：

```text
.github/workflows/publish-to-pypi.yml
```

推荐触发方式：

```yaml
on:
  workflow_dispatch:
```

## 11.1 手动运行 PyPI workflow

打开 GitHub 仓库页面：

```text
Actions -> publish to PyPI -> Run workflow
```

分支选择：

```text
master
```

点击运行。

## 11.2 确认发布成功

workflow 成功后，打开：

```text
https://pypi.org/project/histox/
```

确认可以看到新版本，例如：

```text
0.2.2
```

---

# 12. 验证正式 PyPI 安装

建议重新创建一个干净环境测试正式安装。

```bash
python -m venv histox-pypi-test
source histox-pypi-test/bin/activate
python -m pip install --upgrade pip
```

安装正式版本：

```bash
pip install --no-cache-dir histox==0.2.2
```

测试导入：

```bash
python - <<'PY'
import histox

print("histox import OK")
print(histox)

if hasattr(histox, "__version__"):
    print("histox version:", histox.__version__)
PY
```

如果一切正常，正式发布完成。

---

# 13. 创建 GitHub Release

正式 PyPI 发布后，建议创建 GitHub Release。

打开：

```text
GitHub 仓库 -> Releases -> Draft a new release
```

选择 tag：

```text
v0.2.2
```

Release title：

```text
v0.2.2
```

Release 内容示例：

```markdown
## What's Changed

### Added
- Add xxx feature.

### Fixed
- Fix xxx bug.

### Documentation
- Update usage examples.

## Install

```bash
pip install histox==0.2.2
```
```

保存后，用户就可以在 GitHub Release 页面看到版本说明。

---

# 14. GitHub Secrets 管理

本项目发布不建议在个人电脑上使用 token 上传。

推荐使用 GitHub Actions + GitHub Secrets。

需要的 Secrets：

```text
TEST_PYPI_API_TOKEN
PYPI_API_TOKEN
```

## 14.1 TestPyPI token

从 TestPyPI 创建：

```text
https://test.pypi.org/manage/account/token/
```

保存到 GitHub：

```text
GitHub 仓库 -> Settings -> Secrets and variables -> Actions
```

Secret 名称：

```text
TEST_PYPI_API_TOKEN
```

## 14.2 PyPI token

从正式 PyPI 创建：

```text
https://pypi.org/manage/account/token/
```

保存到 GitHub：

```text
GitHub 仓库 -> Settings -> Secrets and variables -> Actions
```

Secret 名称：

```text
PYPI_API_TOKEN
```

## 14.3 注意事项

不要把 token 写进代码。

不要这样做：

```yaml
password: pypi-xxxxxxxxxxxxxxxx
```

应该这样写：

```yaml
password: ${{ secrets.PYPI_API_TOKEN }}
```

TestPyPI 对应：

```yaml
password: ${{ secrets.TEST_PYPI_API_TOKEN }}
```

---

# 15. 推荐的 GitHub Actions 配置

## 15.1 TestPyPI workflow 示例

文件：

```text
.github/workflows/publish-to-test-pypi.yml
```

示例：

```yaml
name: publish to TestPyPI

on:
  workflow_dispatch:

jobs:
  build-n-publish:
    name: Build and publish Python distributions to TestPyPI
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python 3.9
        uses: actions/setup-python@v5
        with:
          python-version: "3.9"

      - name: Install build dependencies
        run: |
          python -m pip install --upgrade pip
          python -m pip install --upgrade build wheel twine
          python -m pip install -r requirements.txt

      - name: Initialize submodule
        run: |
          git submodule update --init --recursive

      - name: Build distributions
        run: |
          rm -rf dist build *.egg-info
          python -m build
          ls -lh dist

      - name: Check distributions
        run: |
          python -m twine check dist/*

      - name: Publish distribution to TestPyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          user: __token__
          password: ${{ secrets.TEST_PYPI_API_TOKEN }}
          repository-url: https://test.pypi.org/legacy/
```

## 15.2 正式 PyPI workflow 示例

文件：

```text
.github/workflows/publish-to-pypi.yml
```

示例：

```yaml
name: publish to PyPI

on:
  workflow_dispatch:

jobs:
  build-n-publish:
    name: Build and publish Python distributions to PyPI
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python 3.9
        uses: actions/setup-python@v5
        with:
          python-version: "3.9"

      - name: Install build dependencies
        run: |
          python -m pip install --upgrade pip
          python -m pip install --upgrade build wheel twine
          python -m pip install -r requirements.txt

      - name: Initialize submodule
        run: |
          git submodule update --init --recursive

      - name: Build distributions
        run: |
          rm -rf dist build *.egg-info
          python -m build
          ls -lh dist

      - name: Check distributions
        run: |
          python -m twine check dist/*

      - name: Publish distribution to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          user: __token__
          password: ${{ secrets.PYPI_API_TOKEN }}
          repository-url: https://upload.pypi.org/legacy/
```

---

# 16. 常见问题

## 16.1 普通 push 会自动发布包吗？

如果 workflow 是：

```yaml
on:
  workflow_dispatch:
```

那么不会。

普通：

```bash
git push origin master
```

只会更新 GitHub 上的代码，不会发布 TestPyPI / PyPI。

## 16.2 GitHub tag 会自动更新吗？

不会。

tag 必须手动创建：

```bash
git tag v0.2.2
git push origin v0.2.2
```

tag 创建后会固定指向某个 commit，不会自动跟随 `master` 更新。

## 16.3 可以重复上传同一个版本吗？

不可以。

如果已经上传过：

```text
histox==0.2.2
```

就不能再次上传另一个 `0.2.2`。

需要改成：

```text
histox==0.2.3
```

## 16.4 如果发布到 TestPyPI 后发现问题怎么办？

不要覆盖原版本。

正确流程：

```text
0.2.2 有问题 -> 修复代码 -> 改成 0.2.3 -> 重新发布 TestPyPI
```

## 16.5 如果正式 PyPI 发布后发现严重 bug 怎么办？

正确做法是尽快发修复版本：

```text
0.2.2 -> 0.2.3
```

不建议删除 PyPI 包，也不建议复用旧版本号。

---

# 17. 完整发布清单

每次发布前可以按下面清单检查。

## 17.1 发布前

- [ ] 所有需要合并的 PR 已合并
- [ ] 本地 `master` 已更新
- [ ] 新建了 release 分支
- [ ] 已修改版本号
- [ ] 已更新 changelog
- [ ] 本地测试通过
- [ ] 本地 `python -m build` 成功
- [ ] 本地 `python -m twine check dist/*` 成功
- [ ] release PR 已创建
- [ ] release PR 已 review
- [ ] release PR 已合并

## 17.2 TestPyPI

- [ ] 手动运行 TestPyPI workflow
- [ ] workflow 成功
- [ ] TestPyPI 页面能看到新版本
- [ ] 从 TestPyPI 安装成功
- [ ] import 测试成功
- [ ] 核心功能测试成功

## 17.3 正式发布

- [ ] 创建 Git tag
- [ ] 推送 Git tag
- [ ] 手动运行 PyPI workflow
- [ ] workflow 成功
- [ ] PyPI 页面能看到新版本
- [ ] 从正式 PyPI 安装成功
- [ ] import 测试成功
- [ ] 创建 GitHub Release

---

# 18. 推荐的一次完整发布示例

假设准备从：

```text
0.2.1
```

发布到：

```text
0.2.2
```

## 18.1 创建 release 分支

```bash
git checkout master
git pull origin master
git checkout -b release/v0.2.2
```

## 18.2 修改版本号和 changelog

修改：

```text
setup.py
histox/__init__.py
CHANGELOG.md
```

## 18.3 本地构建检查

```bash
rm -rf dist build *.egg-info
python -m pip install --upgrade build twine
python -m build
python -m twine check dist/*
```

## 18.4 提交 release PR

```bash
git add .
git commit -m "release: bump version to 0.2.2"
git push origin release/v0.2.2
```

在 GitHub 上创建 PR：

```text
release/v0.2.2 -> master
```

review 后合并。

## 18.5 发布 TestPyPI

打开：

```text
GitHub -> Actions -> publish to TestPyPI -> Run workflow
```

安装测试：

```bash
pip install --no-cache-dir \
  --index-url https://test.pypi.org/simple/ \
  --extra-index-url https://pypi.org/simple \
  histox==0.2.2
```

## 18.6 创建 tag

```bash
git checkout master
git pull origin master
git tag v0.2.2
git push origin v0.2.2
```

## 18.7 发布正式 PyPI

打开：

```text
GitHub -> Actions -> publish to PyPI -> Run workflow
```

安装测试：

```bash
pip install --no-cache-dir histox==0.2.2
```

## 18.8 创建 GitHub Release

打开：

```text
GitHub -> Releases -> Draft a new release
```

选择：

```text
v0.2.2
```

填写版本说明并发布。

---

# 19. 最简流程总结

日常开发：

```text
feature branch -> Pull Request -> review -> merge into master
```

准备发布：

```text
release branch -> bump version -> changelog -> release PR -> merge
```

测试发布：

```text
Run TestPyPI workflow -> install from TestPyPI -> verify
```

正式发布：

```text
git tag vX.Y.Z -> Run PyPI workflow -> install from PyPI -> GitHub Release
```

最重要的三条规则：

```text
1. 普通开发不要直接发布 PyPI
2. 每次发布必须使用新版本号
3. 已经发布的 tag 和版本不要随意修改
```