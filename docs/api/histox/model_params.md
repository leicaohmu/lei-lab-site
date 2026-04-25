# ModelParams

模型超参数配置类。

---

## 示例

```python
import histox as hx

params = hx.ModelParams(
    tile_px=299,
    tile_um=302,
    batch_size=32,
    model='xception',
    learning_rate=0.0001,
    epochs=50,
    validation_fraction=0.2
)
```

## API 参考

::: histox.ModelParams
