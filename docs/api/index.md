# 代码库概览

本页面汇总课题组维护的 Python 代码库及其 API 文档。

---

## histox

> 面向数字病理学的深度学习库，基于 Slideflow 构建。

**安装：**

```bash
pip install histox
# 或安装 PyTorch 版本
pip install histox[torch]
```

**GitHub：** [leicaohmu/histox](https://github.com/leicaohmu/histox)

| 模块 | 说明 |
|------|------|
| [Project](histox/project.md) | 项目管理、切片提取、模型训练 |
| [ModelParams](histox/model_params.md) | 模型超参数配置 |
| [Heatmap](histox/heatmap.md) | 热力图与可视化 |
| [工具函数](histox/utils.md) | 辅助工具函数 |
