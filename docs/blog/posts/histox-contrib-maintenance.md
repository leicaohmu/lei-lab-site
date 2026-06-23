---
title: HisToX 与 HisToX-Contrib 开发与维护指南
date: 2026-05-05
authors:
  - lei
categories:
  - 开发笔记
tags:
  - histox
  - 插件开发
  - Python
description: 记录 histox 核心库与 histox-contrib 社区插件包的架构设计、注册机制及日常维护方法。
---

# HisToX 与 HisToX-Contrib 开发维护指南 🔧

在将 `slideflow-gpl` 移植为 `histox-contrib` 的过程中，我们深入分析了 `histox` 的插件注册机制，踩了不少坑。本文记录架构设计、注册原理及日常维护方法，供后续开发参考。

<!-- more -->

## 项目结构概览

| 仓库 | 说明 | 许可证 |
|------|------|--------|
| `histox` | 核心库，提供 WSI 处理、特征提取框架 | Apache-2.0 |
| `histox-contrib` | 社区插件包，提供 CTransPath、RetCCL 等提取器和 CLAM 模块 | GPL-3.0 |

两者通过 Python **Entry Points** 机制解耦，`histox-contrib` 无需修改核心库代码即可自动注册插件。

值得注意的是，`histox` 的 `_registry.py` 中已经**原生预登记**了 `histox-contrib` 提供的提取器名称：

```python
# histox/model/extractors/_registry.py
_known_extras_packages = {
    'histox-contrib': ['retccl', 'ctranspath'],
    'histox-noncommercial': ['gigapath', 'histossl', 'plip']
}
```

这意味着当用户尝试使用 `ctranspath` 但未安装 `histox-contrib` 时，`histox` 能给出精准的安装提示，而不是一个模糊的报错。

---

## 🔌 插件注册机制

### 注册字典原理

`histox` 的特征提取器依赖两个注册字典，分别对应两个后端：

```python
# histox/model/extractors/_registry.py
_tf_extractors   = dict()   # name -> factory_fn  (TensorFlow)
_torch_extractors = dict()  # name -> factory_fn  (PyTorch)
```

`register_torch` 装饰器支持两种用法：

```python
# 用法一：直接装饰，使用函数名作为注册 key
@register_torch
def ctranspath(**kwargs):
    ...

# 用法二：指定自定义 key name
@register_torch('ctranspath-v2')
def ctranspath_v2(**kwargs):
    ...
```

调用 `build_feature_extractor('ctranspath')` 时，工厂函数查字典：

```python
# histox/model/extractors/_factory.py（简化）
if is_torch_extractor(name):
    return build_torch_feature_extractor(name, **kwargs)
elif name in _extras_extractors:
    raise InvalidFeatureExtractor(
        f"{name} requires the package {_extras_extractors[name]}, "
        f"please install with 'pip install {_extras_extractors[name]}'"
    )
```

!!! warning "重要：`sys.modules` 注入对提取器注册无效"
    从 `slideflow-gpl` 移植时，原来基于 `sys.modules` 注入模块的方式在 `histox` 中**静默失效**——不报错，但提取器永远不会出现在注册表里。提取器必须通过 `@register_torch` 装饰器注册。

### 注册链路

```
pip install histox-contrib
      ↓
histox 启动时调用 load_plugins()
      ↓
通过 entry_points(group='histox.plugins') 发现插件
      ↓
调用 histox_contrib:register_extras()
      ↓
@register_torch 将工厂函数写入 _torch_extractors 字典
      ↓
hx.build_feature_extractor('ctranspath') 成功 ✅
```

### Entry Points 配置

`histox-contrib` 的 `setup.py`：

```python
entry_points={
    'histox.plugins': [
        'extras = histox_contrib:register_extras',
    ],
},
```

---

## 📦 histox-contrib 核心实现

### 目录结构

```
histox-contrib/
├── histox_contrib/
│   ├── __init__.py          # register_extras() 入口
│   ├── extractors/
│   │   ├── ctranspath.py    # CTransPathFeatures
│   │   └── retccl.py        # RetCCLFeatures
│   └── clam/                # CLAM 模块
└── setup.py
```

### `__init__.py` 核心实现

```python
def register_extras():
    from histox.model.extractors._registry import register_torch

    @register_torch
    def ctranspath(**kwargs):
        from histox_contrib.extractors.ctranspath import CTransPathFeatures
        return CTransPathFeatures(**kwargs)

    @register_torch
    def retccl(**kwargs):
        from histox_contrib.extractors.retccl import RetCCLFeatures
        return RetCCLFeatures(**kwargs)

    # 注意：CLAM 模块无需在此处理
    # histox/__init__.py 的 __getattr__ 会在用户执行
    # `import histox.clam` 时自动转发到 histox_contrib.clam
```

!!! note "CLAM 为什么不需要手动注入？"
    `histox/__init__.py` 实现了 `__getattr__`，当访问 `histox.clam` 时会自动
    执行 `importlib.import_module('histox_contrib.clam')` 并缓存结果。
    `register_extras()` 只需专注于注册特征提取器即可。

---

## ➕ 如何添加新的特征提取器

以添加 `uni` 提取器为例，分三步完成：

=== "Step 1：创建提取器类"

    ```python
    # histox_contrib/extractors/uni.py
    from histox.model import BaseFeatureExtractor  # 正确的导入路径

    class UNIFeatures(BaseFeatureExtractor):
        tag = 'uni'
        backend = 'torch'

        def __init__(self, **kwargs):
            super().__init__(**kwargs)
            self.model = ...        # 加载模型权重
            self.num_features = 1024
            self.transform = self.build_transform(img_size=224)
    ```

=== "Step 2：注册工厂函数"

    ```python
    # 在 register_extras() 中添加
    @register_torch
    def uni(**kwargs):
        from histox_contrib.extractors.uni import UNIFeatures
        return UNIFeatures(**kwargs)
    ```

=== "Step 3：安装并验证"

    ```bash
    pip install -e .
    ```

    ```python
    import histox as hx
    assert 'uni' in hx.list_torch_extractors()
    extractor = hx.build_feature_extractor('uni')
    print(extractor)  # <UNIFeatures ...>
    ```

同时，建议在 `histox` 的 `_registry.py` 中将 `uni` 补充到 `_known_extras_packages`，
以便未安装时给出友好提示：

```python
_known_extras_packages = {
    'histox-contrib': ['retccl', 'ctranspath', 'uni'],  # ← 添加 uni
    ...
}
```

---

## 🛠️ histox 核心库维护要点

### 懒加载扩展模块

`histox/__init__.py` 通过 `__getattr__` 拦截对扩展模块的访问：

```python
_NONCOMMERCIAL_MODULES = {'biscuit', 'stylegan2', 'stylegan3', 'extractors'}
_GPL_MODULES = {'clam'}

def __getattr__(name):
    if name in _GPL_MODULES:
        try:
            mod = importlib.import_module(f'histox_contrib.{name}')
            globals()[name] = mod   # 缓存，避免重复触发 __getattr__
            return mod
        except ImportError:
            raise AttributeError(
                f"histox.{name} requires the 'histox-contrib' package.\n"
                f"Install it with:  pip install histox-contrib"
            )
```

!!! danger "常见错误"
    若此处包名残留 `slideflow_gpl`，执行 `import histox.clam` 会报：
    ```
    ModuleNotFoundError: No module named 'slideflow_gpl'
    ```
    将所有 `slideflow_gpl` 替换为 `histox_contrib` 即可。

---

## 👥 多人协作开发

课题组多人共同维护时，需要在**权限管理**、**分支策略**和**发布流程**上建立规范，
避免代码冲突或误操作覆盖 PyPI 已发布版本。

### 角色分工

建议将贡献者分为两类角色：

| 角色 | GitHub 权限 | PyPI 权限 | 典型职责 |
|------|------------|----------|---------|
| **维护者**（Maintainer） | Admin / Maintain | 可上传发布 | 审查 PR、合并主干、发布新版本 |
| **贡献者**（Contributor） | Write（或 Fork） | 无 | 开发新提取器、修复 Bug、提交 PR |

课题组建议由 **1～2 名同学**担任维护者，负责最终合并与发布；其余同学以贡献者身份提交 PR。

### 分支策略

推荐使用简化的 **GitHub Flow**，适合课题组规模：

```
main ──────────────────────────────────────────▶  （始终可发布）
  │
  ├── feature/add-uni-extractor   ← 新功能分支
  ├── fix/clam-import-error       ← Bug 修复分支
  └── release/0.3.0               ← 发布准备分支（可选）
```

- `main` 分支**受保护**，不允许直接 push，只能通过 PR 合并
- 每个新功能或修复单独开一个分支，命名格式：`feature/xxx` 或 `fix/xxx`
- PR 合并前需至少 **1 名维护者 Review**

在 GitHub 仓库设置中开启分支保护：

> `Settings` → `Branches` → `Add branch ruleset`
> 勾选 **Require a pull request before merging** 和 **Require approvals (1)**

### 贡献者工作流

贡献者（无论是组内成员还是外部）按以下流程提交代码：

=== "组内成员（有 Write 权限）"

    ```bash
    # 1. 从 main 创建功能分支
    git checkout main && git pull
    git checkout -b feature/add-uni-extractor

    # 2. 开发、提交
    git add .
    git commit -m "feat: add UNI feature extractor"

    # 3. 推送并发起 PR
    git push -u origin feature/add-uni-extractor
    # 然后在 GitHub 上点击 "Compare & pull request"
    ```

=== "外部贡献者（Fork 方式）"

    ```bash
    # 1. 在 GitHub 上 Fork 仓库到自己账号

    # 2. Clone 自己的 Fork
    git clone https://github.com/<your-name>/histox-contrib.git
    cd histox-contrib

    # 3. 添加上游仓库
    git remote add upstream https://github.com/leicaohmu/histox-contrib.git

    # 4. 创建功能分支并开发
    git checkout -b feature/add-uni-extractor

    # 5. 推送到自己的 Fork 并发起 PR 到上游 main
    git push origin feature/add-uni-extractor
    ```

### PyPI Token 管理

PyPI 支持为每个包单独生成 **Scoped Token**，建议按以下方式管理：

```
pypi.org
├── 账号 Token（leicaohmu）     ← 仅维护者持有，权限最高，平时不用
├── histox 发布 Token           ← 仅限上传 histox，交给 histox 维护者
└── histox-contrib 发布 Token  ← 仅限上传 histox-contrib，交给 contrib 维护者
```

生成 Scoped Token 步骤：

> PyPI → Account Settings → API tokens → **Add API token**
> → Scope 选择 **Project: histox**（仅限该包）

!!! danger "Token 安全须知"
    - Token 只在生成时显示一次，请立即保存到安全的地方（如密码管理器）
    - **不要**将 Token 提交到 Git 仓库，不要写入代码文件
    - 若 Token 泄露，立即在 PyPI 后台撤销并重新生成

### 推荐：用 GitHub Actions 自动发布

手动发布容易出错，推荐配置 GitHub Actions，**打 Tag 时自动触发发布**，
同时避免将 Token 存储在个人电脑上：

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  push:
    tags:
      - 'v*'   # 推送 v0.2.1 这样的 tag 时触发

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Install build tools
        run: pip install build twine

      - name: Build
        run: python -m build

      - name: Publish to PyPI
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_TOKEN }}  # 存在仓库 Secrets 里
        run: twine upload dist/*
```

在 GitHub 仓库中添加 Secret：

> `Settings` → `Secrets and variables` → `Actions` → **New repository secret**
> Name: `PYPI_TOKEN`，Value: 粘贴 PyPI Scoped Token

配置完成后，发布流程简化为：

```bash
# 维护者只需修改版本号、打 tag、push
git commit -m "bump version to 0.3.0"
git tag v0.3.0
git push && git push --tags
# GitHub Actions 自动完成构建和上传 ✅
```

---

## 🚀 发布到 PyPI（手动流程）

`histox` 和 `histox-contrib` 是两个独立的包，需分别发布。发布前确保已安装构建工具：

```bash
pip install build twine
```

### 准备工作：配置 PyPI Token

首次发布前需在 [pypi.org](https://pypi.org) 注册账号并生成 API Token，
然后写入本地配置文件 `~/.pypirc`：

```ini
[distutils]
index-servers =
    pypi

[pypi]
username = __token__
password = pypi-xxxxxxxxxxxxxxxxxxxxxxxx...
```

!!! tip "推荐使用 TestPyPI 先验证"
    正式发布前，可先发布到 [test.pypi.org](https://test.pypi.org) 验证包的完整性，
    避免占用正式版本号。TestPyPI 需单独注册账号并生成独立 Token。

### 发布步骤

=== "1. 更新版本号"

    修改 `setup.py` 中的版本号，遵循语义化版本规范：

    ```python
    setup(
        name='histox',
        version='0.2.1',   # ← 修改这里
        ...
    )
    ```

    | 版本号位 | 含义 | 示例场景 |
    |---------|------|---------|
    | Major `x.0.0` | 不兼容的 API 变更 | 重构注册机制 |
    | Minor `0.x.0` | 新增功能，向后兼容 | 新增提取器支持 |
    | Patch `0.0.x` | Bug 修复 | 修复 `__getattr__` 包名错误 |

=== "2. 构建分发包"

    ```bash
    cd /path/to/histox
    rm -rf dist/ build/ *.egg-info
    python -m build
    ```

    构建完成后 `dist/` 目录下会生成：

    ```
    dist/
    ├── histox-0.2.1-py3-none-any.whl
    └── histox-0.2.1.tar.gz
    ```

=== "3. 验证并发布"

    ```bash
    # 检查包内容
    twine check dist/*

    # 先发布到 TestPyPI
    twine upload --repository testpypi dist/*
    pip install --index-url https://test.pypi.org/simple/ histox==0.2.1

    # 确认无误后正式发布
    twine upload dist/*
    ```

=== "4. 打 Git Tag"

    ```bash
    git add setup.py
    git commit -m "bump version to 0.2.1"
    git tag v0.2.1
    git push && git push --tags
    ```

!!! warning "注意发布顺序"
    若两个包同时有更新，**先发布 `histox`，再发布 `histox-contrib`**，
    因为 `histox-contrib` 的 `setup.py` 中声明了对 `histox` 的版本依赖。

---

## 🐛 常见问题排查

### Q1：提取器不在 `list_torch_extractors()` 列表中

`register_extras()` 未被调用，通常是插件没有正确安装。

```bash
python -c "
from importlib.metadata import entry_points
print(list(entry_points(group='histox.plugins')))
"
```

若输出为空列表，重新安装：`pip install -e /path/to/histox-contrib`

### Q2：调用 `build_feature_extractor('ctranspath')` 报安装提示错误

这是**正常的友好提示**，说明 `histox` 识别了该提取器名称但插件未安装，
执行 `pip install histox-contrib` 后重试即可。

### Q3：`import histox.clam` 报 `ModuleNotFoundError`

检查 `histox/__init__.py` 的 `__getattr__` 中包名是否正确，
应为 `histox_contrib` 而非 `slideflow_gpl`。

### Q4：`twine upload` 报版本号已存在

```
HTTPError: 400 Bad Request - File already exists.
```

PyPI **不允许覆盖已发布的版本号**，必须递增版本号后重新构建再上传。

### Q5：PR 合并后 Actions 未触发发布

检查 Tag 是否已推送：`git push --tags`。
Actions 仅在推送 `v*` 格式的 Tag 时触发，普通 commit push 不会触发。

---

## ✅ 快速验证清单

每次安装或修改后，运行以下脚本确认一切正常：

```python
import histox as hx

# 1. 确认插件已注册
extractors = hx.list_torch_extractors()
assert 'ctranspath' in extractors, "❌ ctranspath 未注册"
assert 'retccl' in extractors,    "❌ retccl 未注册"
print("✅ 提取器注册正常")

# 2. 确认实例化正常
extractor = hx.build_feature_extractor('ctranspath')
print(f"✅ 实例化正常: {extractor}")

# 3. 确认 CLAM 模块正常
import histox.clam
print(f"✅ CLAM 模块正常: {histox.clam}")
```

---

> 遇到其他问题？欢迎在 [GitHub Issues](https://github.com/leicaohmu/histox-contrib/issues) 提交反馈！ 🚀