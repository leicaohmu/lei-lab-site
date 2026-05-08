# histox.model

This module provides the [`ModelParams`][histox.model.ModelParams] class to organize model and training
parameters/hyperparameters and assist with model building, as well as the [`Trainer`][histox.model.Trainer] class that
executes model training and evaluation. [`RegressionTrainer`][histox.model.RegressionTrainer] and
[`SurvivalTrainer`][histox.model.SurvivalTrainer] are extensions of this class, supporting regression
and Cox Proportional Hazards outcomes, respectively. The function
[`build_trainer`][histox.model.build_trainer] can choose and return the correct model instance based
on the provided hyperparameters.

!!! note
    In order to support both Tensorflow and PyTorch backends, the `histox.model` module will import either
    `histox.model.tensorflow` or `histox.model.torch` according to the currently active backend,
    indicated by the environmental variable `HX_BACKEND`.

See [Training Guide](../training.md) for a detailed look at how to train models.

---

## Trainer

::: histox.model.Trainer
    options:
      members:
        - load
        - evaluate
        - predict
        - train

---

## RegressionTrainer

::: histox.model.RegressionTrainer

---

## SurvivalTrainer

::: histox.model.SurvivalTrainer

---

## Features

::: histox.model.Features
    options:
      members:
        - from_model
        - __call__

---

## Other functions

### list_extractors

::: histox.model.list_extractors

!!! note "How registration works"
    `list_extractors()` returns the union of all registered Tensorflow and PyTorch extractors.
    Extractors are registered automatically via the `@register_tf` and `@register_torch` decorators
    when `histox.model.tensorflow` or `histox.model.torch` is imported. The two backends share the
    same registry dictionaries (`_tf_extractors` and `_torch_extractors`), so any extractor defined
    in `_factory_tensorflow.py` or `_factory_torch.py` becomes immediately visible to `list_extractors()`.

    ```python
    # Example: how an extractor gets registered under the hood
    # (in _factory_tensorflow.py)
    @register_tf
    def resnet50_imagenet(tile_px, **kwargs):
        ...
    # Equivalent to:
    # _tf_extractors['resnet50_imagenet'] = resnet50_imagenet
    ```

    The full lookup flow is:

    ```
    import _factory_tensorflow / _factory_torch
            │
            ├── @register_tf / @register_torch
            │       └──→ _tf_extractors[name] = fn
            │       └──→ _torch_extractors[name] = fn
            │
    list_extractors()
            │
            ├── _tf_extractors.keys()  +  _torch_extractors.keys()
            ├── set() deduplication
            └── ['resnet50_imagenet', 'ctranspath', 'simclr', ...]
    ```

::: histox.model.build_trainer
::: histox.model.build_feature_extractor
::: histox.model.list_tensorflow_extractors
::: histox.model.list_torch_extractors
::: histox.model.load
::: histox.model.is_tensorflow_model
::: histox.model.is_tensorflow_tensor
::: histox.model.is_torch_model
::: histox.model.is_torch_tensor
::: histox.model.read_hp_sweep
::: histox.model.rebuild_extractor
