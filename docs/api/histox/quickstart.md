# 快速上手

## 安装

```bash
# 基础安装
pip install histox

# 安装 PyTorch 支持
pip install histox[torch]

# 安装 TensorFlow 支持
pip install histox[tf]

# GPU 加速（CUDA 12.x）
pip install histox[torch,cucim-cuda12]
```

## 5 步完成肺癌分类训练

```python
import histox as hx
import os

project_root      = '/path/to/project'
annotations_file  = '/path/to/annotations.csv'
slides_directory  = '/path/to/slides'
tfrecords_dir     = os.path.join(project_root, 'tfrecords')

# 1. 创建项目
project = hx.create_project(
    root=project_root,
    annotations=annotations_file,
    slides=slides_directory,
    tfrecords=tfrecords_dir
)

# 2. 提取切片
project.extract_tiles(tile_px=299, tile_um=302, workers=8)

# 3. 配置模型参数
params = hx.ModelParams(
    tile_px=299, tile_um=302,
    batch_size=32, model='xception',
    learning_rate=0.0001, epochs=50,
    validation_fraction=0.2
)

# 4. 训练模型
project.train('subtype', params=params, save_predictions=True, multi_gpu=True)

# 5. 生成可解释性图
project.generate_heatmaps()
project.generate_mosaic_maps()
```

## 数据目录结构

```
project_root/
├── slides/               # WSI 文件 (.svs, .ndpi 等)
│   ├── TCGA-83-5908-01Z.svs
│   └── ...
├── annotations.csv       # 切片标签
└── tfrecords/            # 处理后数据（自动创建）
```

## annotations.csv 格式

```csv
patient,subtype,site,slide
TCGA-83-5908,adenocarcinoma,Site-28,TCGA-83-5908-01Z-00-DX1
TCGA-62-A46V,adenocarcinoma,Site-124,TCGA-62-A46V-01Z-00-DX1
TCGA-44-2655,squamous,Site-29,TCGA-44-2655-01Z-00-DX1
```
