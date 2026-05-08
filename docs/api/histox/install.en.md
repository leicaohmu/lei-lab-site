# Installation

![histox installation](../../assets/histox.png)

HistoX is tested on **Linux-based systems** (Ubuntu, CentOS, Red Hat, and Raspberry Pi OS) and **macOS** (Intel and Apple). Windows support is experimental.

## Requirements

- Python >= 3.7 (<3.10 if using [cuCIM](https://docs.rapids.ai/api/cucim/stable/))
- [PyTorch](https://pytorch.org/) (1.9+) *or* [Tensorflow](https://www.tensorflow.org/) (2.5-2.11)
    - Core functionality, including tile extraction, data processing, and tile-based model training, is supported for both PyTorch and Tensorflow. Additional advanced tools, such as Multiple-Instance Learning (MIL), GANs, and pretrained foundation models, require PyTorch.

### Optional

- [Libvips >= 8.9](https://libvips.github.io/libvips/) (alternative slide reader, adds support for \*.scn, \*.mrxs, \*.ndpi, \*.vms, and \*.vmu files)
- Linear solver (for site-preserved cross-validation):
  - [CPLEX 20.1.0](https://www.ibm.com/docs/en/icos/12.10.0?topic=v12100-installing-cplex-optimization-studio) with [Python API](https://www.ibm.com/docs/en/icos/12.10.0?topic=cplex-setting-up-python-api)
  - *or* [Pyomo](http://www.pyomo.org/installation) with [Bonmin](https://anaconda.org/conda-forge/coinbonmin) solver

---

## Download with pip

HistoX can be installed with PyPI. To install via pip:

```bash
# Update to latest pip
pip install --upgrade pip wheel

# Current stable release, Tensorflow backend
pip install histox[tf] cucim cupy-cuda11x

# Alternatively, install with PyTorch backend
pip install histox[torch] cucim cupy-cuda11x
```

The `cupy` package name depends on the installed CUDA version; [see here](https://docs.cupy.dev/en/stable/install.html#installing-cupy) for installation instructions. `cucim` and `cupy` are not required if using Libvips.

---

## Build from source

To build HistoX from source, clone the repository from the project [Github page](https://github.com/leicaohmu/histox/tree/master):

```bash
git clone https://github.com/leicaohmu/histox
cd histox
conda env create -f environment.yml
conda activate histox
python setup.py bdist_wheel
pip install dist/histox* cupy-cuda11x
```

---

## Extensions

The core HistoX package is licensed under the **Apache-2.0** license. Additional functionality, such as pretrained foundation models, are distributed in separate packages according to their licensing terms. Available extensions include:

- **Slideflow-GPL**: GPL-3.0 licensed extensions ([GitHub](https://github.com/slideflow/slideflow-gpl))
    - Includes: [RetCCL](https://www.sciencedirect.com/science/article/abs/pii/S1361841522002730), [CTransPath](https://www.sciencedirect.com/science/article/abs/pii/S1361841522002043), and [CLAM](https://www.nature.com/articles/s41551-020-00682-w).
- **Slideflow-NonCommercial**: CC BY-NC 4.0 licensed extensions for non-commercial use ([GitHub](https://github.com/slideflow/slideflow-noncommercial))
    - Includes: [HistoSSL](https://www.medrxiv.org/content/10.1101/2023.07.21.23292757v2.full.pdf), [PLIP](https://www.nature.com/articles/s41591-023-02504-3), [GigaPath](https://aka.ms/gigapath), [UNI](https://www.nature.com/articles/s41591-024-02857-3), [BISCUIT](https://www.nature.com/articles/s41467-022-34025-x), and [StyleGAN3](https://nvlabs-fi-cdn.nvidia.com/stylegan3/stylegan3-paper.pdf).

These extensions can be installed via pip. The GigaPath feature extractor has additional, more restrictive dependencies that must be installed separately.

```bash
# Install Slideflow-GPL and Slideflow-NonCommercial
pip install slideflow-gpl slideflow-noncommercial

# Install GigaPath dependencies, if desired
pip install slideflow-noncommercial[gigapath] git+ssh://git@github.com/prov-gigapath/prov-gigapath
```

> **📝 Note:** The Slideflow-GPL and Slideflow-NonCommercial extensions are not included in the default Slideflow package due to their licensing terms. Please review the licensing terms of each extension before use.

---

## PyTorch vs. Tensorflow

Histox supports both PyTorch and Tensorflow, with cross-compatible TFRecord storage. Histox will default to using PyTorch if both are available, but the backend can be manually specified using the environmental variable `SF_BACKEND`. For example:

```bash
export SF_BACKEND=tensorflow
```

---

## cuCIM vs. Libvips

By default, Histox reads whole-slide images using [cuCIM](https://docs.rapids.ai/api/cucim/stable/). Although much faster than other openslide-based frameworks, it supports fewer slide scanner formats. Histox also includes a [Libvips](https://libvips.github.io/libvips/) backend, which adds support for \*.scn, \*.mrxs, \*.ndpi, \*.vms, and \*.vmu files. You can set the active slide backend with the environmental variable `SF_SLIDE_BACKEND`:

```bash
export SF_SLIDE_BACKEND=libvips
```

> **⚠️ Warning:** A bug in the pixman library (version=0.38) will corrupt downsampled slide images, resulting in large black boxes across the slide. We have provided a patch for version 0.38 that has been tested for Ubuntu, which is provided in the project [Github page](https://github.com/leicaohmu/histox) (`pixman_repair.sh`), although it may not be suitable for all environments and we make no guarantees regarding its use. The [HistoX docker images](https://hub.docker.com/repository/docker/leicaohmu/histox) already have this applied. If you are installing from source, have pixman version 0.38, and are unable to apply this patch, the use of downsampled image layers must be disabled to avoid corruption (pass `enable_downsample=False` to tile extraction functions).