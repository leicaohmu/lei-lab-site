# 安装

![histox 安装](../../assets/histox.png)

HistoX 已在 **基于 Linux 的系统**（Ubuntu、CentOS、Red Hat 和 Raspberry Pi OS）以及 **macOS**（Intel 和 Apple 芯片）上完成测试。对 Windows 的支持目前处于实验阶段。

## 环境要求

- Python >= 3.7（若使用 [cuCIM](https://docs.rapids.ai/api/cucim/stable/)，则需 <3.10）
- [PyTorch](https://pytorch.org/)（1.9+）*或* [Tensorflow](https://www.tensorflow.org/)（2.5-2.11）
    - 核心功能（包括图像块提取、数据处理和基于图像块的模型训练）同时支持 PyTorch 和 Tensorflow。多实例学习（MIL）、GAN 及预训练基础模型等高级功能仅支持 PyTorch。

### 可选依赖

- [Libvips >= 8.9](https://libvips.github.io/libvips/)（备用病理图像读取器，新增对 \*.scn、\*.mrxs、\*.ndpi、\*.vms 和 \*.vmu 格式的支持）
- 线性求解器（用于保留站点的交叉验证）：
  - [CPLEX 20.1.0](https://www.ibm.com/docs/en/icos/12.10.0?topic=v12100-installing-cplex-optimization-studio) 配合 [Python API](https://www.ibm.com/docs/en/icos/12.10.0?topic=cplex-setting-up-python-api)
  - *或* [Pyomo](http://www.pyomo.org/installation) 配合 [Bonmin](https://anaconda.org/conda-forge/coinbonmin) 求解器

---

## 通过 pip 安装

HistoX 可通过 PyPI进行安装。通过 pip 安装的方式如下：

```bash
# 升级到最新版 pip
pip install --upgrade pip wheel

# 当前稳定版，使用 Tensorflow 后端
pip install histox[tf] cucim cupy-cuda11x

# 或者，使用 PyTorch 后端安装
pip install histox[torch] cucim cupy-cuda11x
```

`cupy` 的包名取决于已安装的 CUDA 版本，请[参阅此处](https://docs.cupy.dev/en/stable/install.html#installing-cupy)获取安装说明。若使用 Libvips，则无需安装 `cucim` 和 `cupy`。

---

## 从源码构建

如需从源码构建 HistoX，请从项目 [Github 页面](https://github.com/leicaohmu/histox/tree/master) 克隆仓库：

```bash
git clone https://github.com/leicaohmu/histox
cd histox
conda env create -f environment.yml
conda activate histox
python setup.py bdist_wheel
pip install dist/histox* cupy-cuda11x
```

---

## 扩展包

HistoX 核心包基于 **Apache-2.0** 协议授权。预训练基础模型等附加功能根据各自的许可条款分发于独立的扩展包中。可用扩展包括：

- **Slideflow-GPL**：基于 GPL-3.0 协议的扩展（[GitHub](https://github.com/slideflow/slideflow-gpl)）
    - 包含：[RetCCL](https://www.sciencedirect.com/science/article/abs/pii/S1361841522002730)、[CTransPath](https://www.sciencedirect.com/science/article/abs/pii/S1361841522002043) 和 [CLAM](https://www.nature.com/articles/s41551-020-00682-w)。
- **Slideflow-NonCommercial**：基于 CC BY-NC 4.0 协议、仅供非商业使用的扩展（[GitHub](https://github.com/slideflow/slideflow-noncommercial)）
    - 包含：[HistoSSL](https://www.medrxiv.org/content/10.1101/2023.07.21.23292757v2.full.pdf)、[PLIP](https://www.nature.com/articles/s41591-023-02504-3)、[GigaPath](https://aka.ms/gigapath)、[UNI](https://www.nature.com/articles/s41591-024-02857-3)、[BISCUIT](https://www.nature.com/articles/s41467-022-34025-x) 和 [StyleGAN3](https://nvlabs-fi-cdn.nvidia.com/stylegan3/stylegan3-paper.pdf)。

上述扩展均可通过 pip 安装。GigaPath 特征提取器有额外的限制性依赖，需单独安装。

```bash
# 安装 Slideflow-GPL 和 Slideflow-NonCommercial
pip install slideflow-gpl slideflow-noncommercial

# 如需安装 GigaPath 依赖
pip install slideflow-noncommercial[gigapath] git+ssh://git@github.com/prov-gigapath/prov-gigapath
```

> **📝 注意：** Slideflow-GPL 和 Slideflow-NonCommercial 扩展因其许可条款限制，未包含在默认的 Slideflow 包中。使用前请仔细阅读各扩展的许可条款。

---

## PyTorch 与 Tensorflow

HistoX 同时支持 PyTorch 和 Tensorflow，并兼容 TFRecord 存储格式。若两者均已安装，HistoX 默认使用 PyTorch，但可通过环境变量 `SF_BACKEND` 手动指定后端。例如：

```bash
export SF_BACKEND=tensorflow
```

---

## cuCIM 与 Libvips

默认情况下，HistoX 使用 [cuCIM](https://docs.rapids.ai/api/cucim/stable/) 读取全切片图像。虽然其速度远快于其他基于 OpenSlide 的框架，但支持的切片扫描仪格式较少。HistoX 同时提供 [Libvips](https://libvips.github.io/libvips/) 后端，可新增对 \*.scn、\*.mrxs、\*.ndpi、\*.vms 和 \*.vmu 格式的支持。可通过环境变量 `SF_SLIDE_BACKEND` 设置当前使用的病理图像后端：

```bash
export SF_SLIDE_BACKEND=libvips
```

> **⚠️ 警告：** pixman 库（版本 0.38）存在一个 Bug，会导致下采样切片图像损坏，在切片上出现大面积黑色区域。我们针对 0.38 版本提供了一个已在 Ubuntu 上测试的补丁，可在项目 [Github 页面](https://github.com/leicaohmu/histox) 中找到（`pixman_repair.sh`），但该补丁可能不适用于所有环境，我们不对其使用效果作任何保证。[HistoX Docker 镜像](https://hub.docker.com/repository/docker/leicaohmu/histox) 已预先应用了此补丁。若您从源码安装、pixman 版本为 0.38 且无法应用此补丁，则必须禁用下采样图像层以避免损坏（在图像块提取函数中传入 `enable_downsample=False`）。