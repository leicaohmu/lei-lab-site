# Heatmap

热力图与可视化模块，用于生成注意力图、显著性图等可解释性可视化结果。

---

## 示例

```python
import histox as hx

heatmap = hx.Heatmap.from_project(
    project=project,
    outcome_name='subtype',
    model_idx=0
)
heatmap.save('/path/to/output/heatmap.png')
```

## API 参考

::: histox.Heatmap
