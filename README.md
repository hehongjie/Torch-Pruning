<div align="center"> <h1>Torch-Pruning <br> <h3>Towards Any Structural Pruning<h3> </h1> </div>
<div align="center">
<img src="assets/intro.png" width="45%">
</div>

<p align="center">
  <a href="https://github.com/VainF/Torch-Pruning/actions"><img src="https://img.shields.io/badge/tests-passing-9c27b0.svg" alt="Test Status"></a>
  <a href="https://pytorch.org/"><img src="https://img.shields.io/badge/PyTorch-1.8.1%20%7C%202.0.0-673ab7.svg" alt="Tested PyTorch Versions"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-4caf50.svg" alt="License"></a>
  <a href="https://pepy.tech/project/Torch-Pruning"><img src="https://pepy.tech/badge/Torch-Pruning?color=2196f3" alt="Downloads"></a>
  <a href="https://github.com/VainF/Torch-Pruning/releases/latest"><img src="https://img.shields.io/badge/Latest%20Version-1.1.3-3f51b5.svg" alt="Latest Version"></a>
  <a href="https://arxiv.org/abs/2301.12900" target="_blank"><img src="https://img.shields.io/badge/arXiv-2301.12900-009688.svg" alt="arXiv"></a>
</p>


[[中文README | README in Chinese]](README_CN.md)

Torch-Pruning (TP) is a versatile library for Structural Network Pruning with the following features:
* **General-purpose Pruning Toolkit:** TP enables structural pruning for a wide range of neural networks, including *Vision Transformers, Yolov7, FasterRCNN, SSD, KeypointRCNN, MaskRCNN, ResNe(X)t, ConvNext, DenseNet, ConvNext, RegNet, FCN, DeepLab, etc*. Different from [torch.nn.utils.prune](https://pytorch.org/tutorials/intermediate/pruning_tutorial.html) that zeroizes parameters through masking, Torch-Pruning employs a (non-deep) graph algorithm called DepGraph to physically remove coupled parameters (channels) from models. 
* **Reproducible [Performance Benchmark](benchmarks) and [Prunability Benchmark](benchmarks/prunability):** Currently, TP is able to prune approximately **77/85=90.6%** of the models from Torchvision 0.13.1. Check out [practical_structural_pruning.md](practical_structural_pruning.md) for an up-to-date list of practical pruning techniques. 


For more technical details, please refer to our CVPR'23 paper:
> [**DepGraph: Towards Any Structural Pruning**](https://arxiv.org/abs/2301.12900)   
> [Gongfan Fang](https://fangggf.github.io/), [Xinyin Ma](https://horseee.github.io/), [Mingli Song](https://person.zju.edu.cn/en/msong), [Michael Bi Mi](https://dblp.org/pid/317/0937.html), [Xinchao Wang](https://sites.google.com/site/sitexinchaowang/)   

Please do not hesitate to open a [discussion](https://github.com/VainF/Torch-Pruning/discussions) or [issue](https://github.com/VainF/Torch-Pruning/issues) if you encounter any problems with the library or the paper. 

### **Features:**
- [x] Structural (Channel) pruning for [CNNs](benchmarks/prunability/torchvision_pruning.py#L19) (e.g. ResNet, DenseNet), [Transformers](benchmarks/prunability/torchvision_pruning.py#L11) (e.g. ViT) and Detectors (e.g. [Yolov7](benchmarks/prunability/yolov7_train_pruned.py#L102), [FasterRCNN, SSD](benchmarks/prunability/torchvision_pruning.py#L92))
- [x] High-level pruners: [MagnitudePruner](https://arxiv.org/abs/1608.08710), [BNScalePruner](https://arxiv.org/abs/1708.06519), [GroupNormPruner](https://arxiv.org/abs/2301.12900) (a simple pruner used in our paper), RandomPruner, etc.
- [x] Computational Graph Tracing and Dependency Modeling.
- [x] Supported modules: Conv, Linear, Normalization, Transposed Conv, PReLU, Embedding, MultiheadAttention, nn.Parameters and [customized modules](tests/test_customized_layer.py).
- [x] Supported operations: split, concatenation, skip connection, flatten, reshape, view, all element-wise ops, etc.
- [x] [Low-level pruning functions](torch_pruning/pruner/function.py)
- [x] [Benchmarks](benchmarks) and [tutorials](tutorials)
- [x] A [resource list](practical_structural_pruning.md) for practical structrual pruning.

### **TODO List:**
- [ ] A benchmark for [Torchvision](https://pytorch.org/vision/stable/models.html) compatibility (**77/85=90.6%**, :heavy_check_mark:) and [timm](https://github.com/huggingface/pytorch-image-models) compatibility.
- [ ] More Detectors (We are working on the pruning of YOLO series such as YOLOv7 :heavy_check_mark:, YOLOv8)
- [ ] Pruning from Scratch / at Initialization.
- [ ] Language, Speech and Generative Models.
- [ ] More high-level pruners like [FisherPruner](https://arxiv.org/abs/2108.00708), [GrowingReg](https://arxiv.org/abs/2012.09243), etc.
- [ ] More standard layers: GroupNorm, InstanceNorm, Shuffle Layers, etc.
- [ ] More Transformers like Vision Transformers (:heavy_check_mark:), Swin Transformers, PoolFormers.
- [ ] Block/Layer/Depth Pruning
- [ ] Pruning benchmarks for CIFAR, ImageNet and COCO.

## Installation
```bash
pip install torch-pruning # v1.1.3
```
or
```bash
git clone https://github.com/VainF/Torch-Pruning.git # recommended
```

## Quickstart
  
Here we provide a quick start for Torch-Pruning. More explained details can be found in [tutorals](./tutorials/)

### 0. How It Works

In structural pruning, **a ``Group`` constitutes the minimal prunable unit within deep networks**. Each group typically comprises several interdependent parameters that must be removed simultaneously to maintain the integrity of the resulting structures. However, deep networks often present complex dependencies among parameters, making structural pruning a challenging endeavor. This work addresses this challenge by offering an automated mechanism for parameter grouping, which facilitates effortless pruning for a wide range of deep networks.

<div align="center">
<img src="assets/dep.png" width="100%">
</div>

### 1. A Minimal Example

```python
import torch
from torchvision.models import resnet18
import torch_pruning as tp

model = resnet18(pretrained=True).eval()

# 1. build dependency graph for resnet18
DG = tp.DependencyGraph().build_dependency(model, example_inputs=torch.randn(1,3,224,224))

# 2. Specify the to-be-pruned channels. Here we prune those channels indexed by [2, 6, 9].
group = DG.get_pruning_group( model.conv1, tp.prune_conv_out_channels, idxs=[2, 6, 9] )

# 3. prune all grouped layers that are coupled with model.conv1 (included).
if DG.check_pruning_group(group): # avoid full pruning, i.e., channels=0.
    group.prune()

# 4. save & load the pruned model 
torch.save(model, 'model.pth') # save the model object
model_loaded = torch.load('model.pth') # no load_state_dict
```
  
The above example demonstrates the fundamental pruning pipeline using DepGraph. The target layer resnet.conv1 is coupled with several layers, which requires simultaneous removal in structural pruning. Let's print the group and observe how a pruning operation "triggers" other ones. In the following outputs, ``A => B`` means the pruning operation ``A`` triggers the pruning operation ``B``. group[0] refers to the pruning root specified by ``DG.get_pruning_group``.

```
--------------------------------
          Pruning Group
--------------------------------
[0] prune_out_channels on conv1 (Conv2d(3, 64, kernel_size=(7, 7), stride=(2, 2), padding=(3, 3), bias=False)) => prune_out_channels on conv1 (Conv2d(3, 64, kernel_size=(7, 7), stride=(2, 2), padding=(3, 3), bias=False)), idxs=[2, 6, 9] (Pruning Root)
[1] prune_out_channels on conv1 (Conv2d(3, 64, kernel_size=(7, 7), stride=(2, 2), padding=(3, 3), bias=False)) => prune_out_channels on bn1 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)), idxs=[2, 6, 9]
[2] prune_out_channels on bn1 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)) => prune_out_channels on _ElementWiseOp(ReluBackward0), idxs=[2, 6, 9]
[3] prune_out_channels on _ElementWiseOp(ReluBackward0) => prune_out_channels on _ElementWiseOp(MaxPool2DWithIndicesBackward0), idxs=[2, 6, 9]
[4] prune_out_channels on _ElementWiseOp(MaxPool2DWithIndicesBackward0) => prune_out_channels on _ElementWiseOp(AddBackward0), idxs=[2, 6, 9]
[5] prune_out_channels on _ElementWiseOp(MaxPool2DWithIndicesBackward0) => prune_in_channels on layer1.0.conv1 (Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)), idxs=[2, 6, 9]
[6] prune_out_channels on _ElementWiseOp(AddBackward0) => prune_out_channels on layer1.0.bn2 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)), idxs=[2, 6, 9]
[7] prune_out_channels on _ElementWiseOp(AddBackward0) => prune_out_channels on _ElementWiseOp(ReluBackward0), idxs=[2, 6, 9]
[8] prune_out_channels on _ElementWiseOp(ReluBackward0) => prune_out_channels on _ElementWiseOp(AddBackward0), idxs=[2, 6, 9]
[9] prune_out_channels on _ElementWiseOp(ReluBackward0) => prune_in_channels on layer1.1.conv1 (Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)), idxs=[2, 6, 9]
[10] prune_out_channels on _ElementWiseOp(AddBackward0) => prune_out_channels on layer1.1.bn2 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)), idxs=[2, 6, 9]
[11] prune_out_channels on _ElementWiseOp(AddBackward0) => prune_out_channels on _ElementWiseOp(ReluBackward0), idxs=[2, 6, 9]
[12] prune_out_channels on _ElementWiseOp(ReluBackward0) => prune_in_channels on layer2.0.downsample.0 (Conv2d(64, 128, kernel_size=(1, 1), stride=(2, 2), bias=False)), idxs=[2, 6, 9]
[13] prune_out_channels on _ElementWiseOp(ReluBackward0) => prune_in_channels on layer2.0.conv1 (Conv2d(64, 128, kernel_size=(3, 3), stride=(2, 2), padding=(1, 1), bias=False)), idxs=[2, 6, 9]
[14] prune_out_channels on layer1.1.bn2 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)) => prune_out_channels on layer1.1.conv2 (Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)), idxs=[2, 6, 9]
[15] prune_out_channels on layer1.0.bn2 (BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)) => prune_out_channels on layer1.0.conv2 (Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)), idxs=[2, 6, 9]
--------------------------------
```
For more details about grouping, please refer to [tutorials/2 - Exploring Dependency Groups](https://github.com/VainF/Torch-Pruning/blob/master/tutorials/2%20-%20Exploring%20Dependency%20Groups.ipynb)

#### How to scan all groups:
Just like what we do in the [MetaPruner](https://github.com/VainF/Torch-Pruning/blob/b607ae3aa61b9dafe19d2c2364f7e4984983afbf/torch_pruning/pruner/algorithms/metapruner.py#L197), one can use ``DG.get_all_groups(ignored_layers, root_module_types)`` to scan all groups sequentially. Each group will begin with a layer that matches a type in the "root_module_types" parameter. Note that DG.get_all_groups is only responsible for grouping and does not have any knowledge or understanding of which parameters should be pruned. Therefore, it is necessary to specify the pruning idxs using  ``group.prune(idxs=idxs)``.

```python
for group in DG.get_all_groups(ignored_layers=[model.conv1], root_module_types=[nn.Conv2d, nn.Linear]):
    # handle groups in sequential order
    idxs = [2,4,6] # your pruning indices
    group.prune(idxs=idxs)
    print(group)
```


### 2. High-level Pruners

Leveraging the DependencyGraph, we developed several high-level pruners in this repository to facilitate effortless pruning. By specifying the desired channel sparsity, you can prune the entire model and fine-tune it using your own training code. For detailed information on this process, we encourage you to consult the [this tutorial](https://github.com/VainF/Torch-Pruning/blob/master/tutorials/1%20-%20Customize%20Your%20Own%20Pruners.ipynb), which demonstrates how to implement a [slimming](https://arxiv.org/abs/1708.06519) pruner from scratch. Additionally, you can find more practical examples in [benchmarks/main.py](benchmarks/main.py).

```python
import torch
from torchvision.models import resnet18
import torch_pruning as tp

model = resnet18(pretrained=True)

# Importance criteria
example_inputs = torch.randn(1, 3, 224, 224)
imp = tp.importance.MagnitudeImportance(p=2)

ignored_layers = []
for m in model.modules():
    if isinstance(m, torch.nn.Linear) and m.out_features == 1000:
        ignored_layers.append(m) # DO NOT prune the final classifier!

iterative_steps = 5 # progressive pruning
pruner = tp.pruner.MagnitudePruner(
    model,
    example_inputs,
    importance=imp,
    iterative_steps=iterative_steps,
    ch_sparsity=0.5, # remove 50% channels, ResNet18 = {64, 128, 256, 512} => ResNet18_Half = {32, 64, 128, 256}
    ignored_layers=ignored_layers,
)

base_macs, base_nparams = tp.utils.count_ops_and_params(model, example_inputs)
for i in range(iterative_steps):
    pruner.step()
    macs, nparams = tp.utils.count_ops_and_params(model, example_inputs)
    # finetune your model here
    # finetune(model)
    # ...
```

#### Sparse Training
Some pruners like [BNScalePruner](https://github.com/VainF/Torch-Pruning/blob/dd59921365d72acb2857d3d74f75c03e477060fb/torch_pruning/pruner/algorithms/batchnorm_scale_pruner.py#L45) and [GroupNormPruner](https://github.com/VainF/Torch-Pruning/blob/dd59921365d72acb2857d3d74f75c03e477060fb/torch_pruning/pruner/algorithms/group_norm_pruner.py#L53) require sparse training before pruning. This can be easily achieved by inserting just one line of code ``pruner.regularize(model)`` in your training script. The pruner will update the gradient of trainable parameters.
```python
for epoch in range(epochs):
    model.train()
    for i, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)
        optimizer.zero_grad()
        out = model(data)
        loss = F.cross_entropy(out, target)
        loss.backward()
        pruner.regularize(model) # <== for sparse learning
        optimizer.step()
```

#### Interactive Pruning
All high-level pruners support interactive pruning. You can use ``pruner.step(interactive=True)`` to get all groups and interactively prune them by calling ``group.prune()``. This feature is useful if you want to control/monitor the pruning process.

```python
for i in range(iterative_steps):
    for group in pruner.step(interactive=True): # Warning: groups must be handled sequentially. Do not keep them as a list.
        print(group) 
        # do whatever you like with the group 
        # ...
        group.prune() # remeber to call the group.prune()
        # group.prune(idxs=[0, 2, 6]) # It is even possible to change the pruning behaviour with the idxs parameter
    macs, nparams = tp.utils.count_ops_and_params(model, example_inputs)
    # finetune your model here
    # finetune(model)
    # ...
```

#### Group-level Pruning

With DepGraph, it is easy to design some "group-level" criteria to estimate the importance of a whole group rather than a single layer. In our paper, we extend the classic norm-based algorithm and introduce a simple GroupNormPruner, which learns group-level sparsity for pruning.

<div align="center">
<img src="assets/group_sparsity.png" width="80%">
</div>


### 3. Low-level Pruning Functions

While it is possible to manually prune your model using low-level functions, this approach can be quite laborious, as it requires careful management of the associated dependencies. As a result, we recommend utilizing the aforementioned high-level pruners to streamline the pruning process.

```python
tp.prune_conv_out_channels( model.conv1, idxs=[2,6,9] )

# fix the broken dependencies manually
tp.prune_batchnorm_out_channels( model.bn1, idxs=[2,6,9] )
tp.prune_conv_in_channels( model.layer2[0].conv1, idxs=[2,6,9] )
...
```

The following pruning functions are available:
```python
'prune_conv_out_channels',
'prune_conv_in_channels',
'prune_depthwise_conv_out_channels',
'prune_depthwise_conv_in_channels',
'prune_batchnorm_out_channels',
'prune_batchnorm_in_channels',
'prune_linear_out_channels',
'prune_linear_in_channels',
'prune_prelu_out_channels',
'prune_prelu_in_channels',
'prune_layernorm_out_channels',
'prune_layernorm_in_channels',
'prune_embedding_out_channels',
'prune_embedding_in_channels',
'prune_parameter_out_channels',
'prune_parameter_in_channels',
'prune_multihead_attention_out_channels',
'prune_multihead_attention_in_channels',
'prune_groupnorm_out_channels',
'prune_groupnorm_in_channels',
'prune_instancenorm_out_channels',
'prune_instancenorm_in_channels',
```

### 4. Customized Layers

Please refer to [tests/test_customized_layer.py](https://github.com/VainF/Torch-Pruning/blob/master/tests/test_customized_layer.py).

### 5. Benchmarks

Our results on {ResNet-56 / CIFAR-10 / 2.00x}

| Method | Base (%) | Pruned (%) | $\Delta$ Acc (%) | Speed Up |
|:--    |:--:  |:--:    |:--: |:--:      |
| NIPS [[1]](#1)  | -    | -      |-0.03 | 1.76x    |
| Geometric [[2]](#2) | 93.59 | 93.26 | -0.33 | 1.70x |
| Polar [[3]](#3)  | 93.80 | 93.83 | +0.03 |1.88x |
| CP  [[4]](#4)   | 92.80 | 91.80 | -1.00 |2.00x |
| AMC [[5]](#5)   | 92.80 | 91.90 | -0.90 |2.00x |
| HRank [[6]](#6) | 93.26 | 92.17 | -0.09 |2.00x |
| SFP  [[7]](#7)  | 93.59 | 93.36 | +0.23 |2.11x |
| ResRep [[8]](#8) | 93.71 | 93.71 | +0.00 |2.12x |
||
| Ours-L1 | 93.53 | 92.93 | -0.60 | 2.12x |
| Ours-BN | 93.53 | 93.29 | -0.24 | 2.12x |
| Ours-Group | 93.53 | 93.77 | +0.38 | 2.13x |

Please refer to [benchmarks](benchmarks) for more details.

## Citation
```
@article{fang2023depgraph,
  title={DepGraph: Towards Any Structural Pruning},
  author={Fang, Gongfan and Ma, Xinyin and Song, Mingli and Mi, Michael Bi and Wang, Xinchao},
  journal={The IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year={2023}
}
```
