# Project

项目管理核心模块，负责 WSI 切片提取、模型训练与结果分析。

---

## create_project

```python
import histox as hx

project = hx.create_project(
    root='/path/to/project',
    annotations='/path/to/annotations.csv',
    slides='/path/to/slides',
    tfrecords='/path/to/tfrecords'
)
```

## API 参考

::: histox.Project
