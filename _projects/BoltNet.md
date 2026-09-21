---
layout: page
title: BoltNet
description: An Ultra-Lightweight Convolutional Network for On-Device Plant Species Identification
img: assets/img/projects/BoltNet/boltnet_architecture.png
importance: 2
category: research
related_publications: rossi2026boltnet
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/boltnet_architecture.png" title="BoltNet architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The whole network: a small stem, four stages of SRB blocks, and the LPS head that turns the last feature map into 1,081 species scores. Channels become spatial resolution, and every value is kept.
</div>

<div class="row text-center mt-4">
  <div class="col-6 col-md-3 mb-3">
    <div class="card h-100 p-3">
      <h3 class="mb-0">0.682</h3>
      <p class="mb-1" style="color: var(--global-theme-color)"><strong>F<sub>1</sub></strong></p>
      <p class="mb-0 font-weight-light" style="color: var(--global-text-color-light); font-size: 0.85rem">the best score of any model under 2 MB on Pl@ntNet-300K</p>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card h-100 p-3">
      <h3 class="mb-0">1.37</h3>
      <p class="mb-1" style="color: var(--global-theme-color)"><strong>MB</strong></p>
      <p class="mb-0 font-weight-light" style="color: var(--global-text-color-light); font-size: 0.85rem">341K parameters and 56M FLOPs, fully convolutional</p>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card h-100 p-3">
      <h3 class="mb-0">2,354</h3>
      <p class="mb-1" style="color: var(--global-theme-color)"><strong>FPS/W</strong></p>
      <p class="mb-0 font-weight-light" style="color: var(--global-text-color-light); font-size: 0.85rem">on the Hailo-8 NPU, and first on the Jetson GPU too</p>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card h-100 p-3">
      <h3 class="mb-0">−77%</h3>
      <p class="mb-1" style="color: var(--global-theme-color)"><strong>params</strong></p>
      <p class="mb-0 font-weight-light" style="color: var(--global-text-color-light); font-size: 0.85rem">fewer weights than the block it starts from, at a 3.1% F<sub>1</sub> cost</p>
    </div>
  </div>
</div>

## Abstract

**BoltNet** is an ultra-lightweight fully convolutional architecture for fine-grained, high-cardinality image classification on constrained hardware, accepted at the **CVPPA workshop at ECCV 2026** {% cite rossi2026boltnet %}.

The design rests on two **parameter-free** rearrangements: the **Spatial Redistribution Bottleneck (SRB)** in the backbone, and **Logit Pre-Sampling (LPS)** before the classifier. Both trade channels for spatial resolution through a lossless bijection, cutting a large number of parameters while leaving the remaining work as dense, hardware-friendly convolutions. What decides the outcome is _where the capacity sits_, rather than how much of it is cut away.

## The problem

Identifying plant species from field photographs is a demanding fine-grained recognition task, and [Pl@ntNet-300K](https://github.com/plantnet/PlantNet-300K) is the hard version of it:

- **1,081 species** to tell apart, many of them close lookalikes, in photos taken outdoors by volunteers.
- **11% of the images** cover the rarest 80% of the species, and those rare ones are exactly what a botanical survey needs to catch.
- **Under 2 MB** is the budget on the board, and the weights, the activations _and_ the supported operators all have to fit inside it.

Real capacity is needed for the label space. The device that carries the model has none to spare. And model size is not the whole cost: what fills memory during inference are the intermediate activations, and a FLOP count says little about latency or energy once data movement and operator support come into play. So every model was put on the three boards and measured there, rather than ranked by complexity metrics.

## Methodology

### Spatial Redistribution Bottleneck (SRB)

The parameter count of a convolutional network grows with its channel width, primarily because of pointwise convolutions, whose cost scales with the number of input and output channels. This dominates in the later stages of a pyramidal backbone. The inverted residual bottleneck (IRB) of MobileNetV2 leans on pointwise convolutions and channel expansion, and so becomes parameter-heavy exactly where the network is widest.

The SRB redistributes part of the channel content into the spatial domain _before_ the bottleneck. With $$X \in \mathbb{R}^{H \times W \times C}$$ partitioned into groups of $$G$$ channels, a deterministic operator $$f$$ induces a bijection between index sets that preserves cardinality:

$$
H \cdot W \cdot G = (s_h \cdot H) \cdot (s_w \cdot W) \cdot \frac{G}{s_h \cdot s_w}
$$

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/srb.png" title="IRB vs SRB" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    (a) The inverted residual block beside the SRB, where yellow is the rearrangement and pink the pooling that keeps the skip aligned. (b) Channels becoming positions: <code>H × W × C → 2H × 2W × C/4</code>, with no weights of its own and every value kept.
</div>

The redistribution is **structured and lossless**: nothing is compressed or discarded. In the paper $$f$$ is instantiated with a parameter-free sub-pixel rearrangement (`nn.PixelShuffle`). With upscale factor 2 the channel count drops fourfold while each spatial side doubles, so the pointwise convolutions that follow see four times fewer channels.

One SRB block:

| Step | Operator                                              | Effect                                                    |
| ---- | ----------------------------------------------------- | --------------------------------------------------------- |
| 1    | `PixelShuffle(2)`                                     | `C → C/4` channels, `H×W → 2H×2W` (parameter-free)        |
| 2    | `Conv2d 1×1` + BN + HardSwish                         | expansion factor 1.2                                      |
| 3    | `Conv2d 5×5` depth-wise, stride 2 + BN + HardSwish    | reads the enlarged grid, restores resolution for the skip |
| 4    | `Conv2d 1×1` + BN                                     | linear projection to `out_channels`                       |
| skip | `Conv2d 1×1` + BN when `in ≠ out`, identity otherwise | residual                                                  |

By temporarily increasing the spatial resolution, the depth-wise convolution operates on a denser feature map and captures finer local patterns at low parameter cost. This introduces a **second scaling axis** alongside depth and channel width: instead of adding layers or channels, the SRB reallocates part of the budget to spatial resolution.

### Logit Pre-Sampling (LPS)

In high-cardinality classification the final linear layer can dominate the budget: with $$C$$ pre-logit channels and $$N_{cls}$$ classes it holds $$C \cdot N_{cls}$$ weights. On 1,081 classes a plain head over the final 368 channels needs about **0.40M weights, more than the entire rest of the network**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/lps_head.png" title="Classifier head cost" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    On 1,081 classes the plain head alone outweighs everything before it. LPS divides it by r², bringing 0.40M classifier weights down to 0.10M.
</div>

LPS applies the same parameter-free rearrangement to the final feature tensor _before_ global average pooling:

$$
X' = P_r(X), \qquad z = \operatorname{GAP}(X') \in \mathbb{R}^{C/r^2}, \qquad y = Wz + b
$$

The rearrangement is bijective and preserves all feature values. The subsequent pooling aggregates groups of pre-logit channels in a structured manner, reducing the classifier cost from $$C \cdot N_{cls}$$ to $$C \cdot N_{cls} / r^2$$. With $$r = 2$$ the head shrinks to **100,533** parameters (`Linear(92, 1081)`). LPS adds no learnable parameters and leaves the backbone unchanged.

### Full model

| Component         | Specification                                                                                                        |
| ----------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Stem**          | `Conv2d 3×3`, 32 channels, stride 2 → `Conv2d 3×3` depth-wise, stride 2 (BN + HardSwish on both), output `H/4 × W/4` |
| **Stages**        | 4 stages of **4, 4, 4, 3** SRB blocks, output widths **24, 56, 152, 368**                                            |
| **Stage closing** | `MaxPool2d` after each of the first three stages                                                                     |
| **Head**          | LPS (`r = 2`) → global average pooling → `Linear(92, N_cls)`                                                         |
| **Activation**    | HardSwish everywhere except the output layer                                                                         |
| **Input**         | 224 × 224                                                                                                            |

### Experimental setup

- **Benchmarks**: Pl@ntNet-300K, 306,146 images over 1,081 classes (primary), AIDERv2, 16k over 4, and CLRS, 15k over 25.
- **Boards**: Raspberry Pi 5 (Cortex-A76 CPU), Jetson Orin Nano (Tegra Ampere GPU), Hailo-8 (dataflow NPU).
- **Training**: from scratch, 300 epochs, SGD with momentum 0.9, batch size 256, cross-entropy with label smoothing 0.1, cosine annealing from 0.05 to 8 × 10⁻⁵, 224² pixels. Same split, augmentation and seed for every model.

## Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/accuracy_vs_size.png" title="F1 vs model size on Pl@ntNet-300K" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Weighted F₁ against model size on Pl@ntNet-300K, log scale. Every model trained from scratch under the same protocol. The shaded band is the ultra-lightweight regime, the dashed line the 2 MB deployable budget.
</div>

**Pl@ntNet-300K, 1,081 classes, ultra-lightweight group**

| Model        |  Parameters | Size (MB) |        F1 | FLOPs (G) |
| ------------ | ----------: | --------: | --------: | --------: |
| EmergencyNet |     369,647 |      1.48 |     0.636 |     0.116 |
| TakuNet      |     297,001 |      1.19 |     0.483 |     0.032 |
| **BoltNet**  | **341,254** |  **1.37** | **0.682** | **0.056** |

BoltNet is the most accurate model below 2 MB: **+0.046 F₁** over EmergencyNet with 8% fewer parameters and half the FLOPs, and far ahead of TakuNet, which was designed for a much lower-cardinality task. Against larger networks it is competitive rather than dominant. It sits two F₁ points behind RegNetX-002, which has eight times the parameters, and ahead of FBNetV3 at one twenty-fifth of its size.

Secondary evidence of transfer across environmental image classification: **0.958** F1 on AIDERv2 (4 classes) and **0.825** on CLRS (25 classes), in both cases ahead of every ultra-lightweight rival, and on CLRS almost on par with MobileViT V2 (0.826) at about 1/5 of the parameters.

### Ablation

|   SRB   |   LPS   |                    Parameters |                       FLOPs |                         F1 |       ACT |
| :-----: | :-----: | ----------------------------: | --------------------------: | -------------------------: | --------: |
|         |         |                     1,486,029 |                      154.4M |                      0.704 |     1.000 |
| ✓ (r=2) |         |     639,610 <sub>−56.9%</sub> |     56.1M <sub>−63.7%</sub> |     0.687 <sub>−2.4%</sub> |     1.460 |
| ✓ (r=3) |         |     541,930 <sub>−63.5%</sub> |     40.8M <sub>−73.6%</sub> |    0.628 <sub>−10.8%</sub> |     0.856 |
|         | ✓ (r=2) |   1,187,673 <sub>−20.1%</sub> |                      154.1M |     0.715 <sub>+1.6%</sub> |     1.305 |
|         | ✓ (r=3) |   1,136,183 <sub>−23.5%</sub> |                      154.2M |     0.721 <sub>+2.4%</sub> |     1.426 |
| ✓ (r=2) | ✓ (r=2) | **341,254** <sub>−77.0%</sub> | **55.8M** <sub>−63.9%</sub> | **0.682** <sub>−3.1%</sub> | **1.938** |

The first row is the inverted-bottleneck-only baseline, the last row is BoltNet. The SRB is the main actor of backbone compression: at upscale factor 2 it removes 56.9% of the parameters and 63.7% of the FLOPs for a 2.4% relative F₁ drop, whereas factor 3 costs 10.8%, because the redistribution has a capacity floor and must be sized to the task. LPS on the full backbone slightly _improves_ accuracy while removing a fifth of the parameters, because shrinking an oversized head also regularises it. Placed on top of the already-compressed SRB backbone, that gain no longer transfers. The two components are **complementary in what they compress**, the backbone and the head, rather than additive in accuracy, and together they reach −77% parameters at −3.1% F₁.

## Embedded Performance

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/energy_efficiency.png" title="Energy efficiency across CPU, GPU and NPU" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Model-only measurements: frames per second per watt, higher is better. BoltNet is first on the GPU and the NPU, and within 0.3 FPS/W of the leader on the CPU.
</div>

| Model        | RPi 5 FPS |   FPS/W | Hailo-8 FPS |      FPS/W | Orin Nano FPS |    FPS/W |
| ------------ | --------: | ------: | ----------: | ---------: | ------------: | -------: |
| MobileViT V2 |      27.9 |     3.0 |       130.3 |      123.4 |         148.0 |     12.7 |
| RegNetX      |      72.2 |     7.7 |  **5794.7** |     2317.9 |         302.6 |     25.2 |
| EmergencyNet |      79.8 |     8.8 |      1698.1 |      962.6 |         284.3 |     23.9 |
| TakuNet      | **104.6** | **9.2** |      1440.4 |      988.6 |         252.4 |     22.1 |
| **BoltNet**  |      94.0 |     8.9 |      3778.8 | **2354.4** |     **325.4** | **30.7** |

The ranking shifts with the execution model, and each competitor shows a different limit. **TakuNet** takes the scalar CPU, then leaves wide accelerators half idle, and its efficiency falls to roughly 0.4× and 0.7× BoltNet's on NPU and GPU. **RegNetX** has the top raw NPU frame rate, because regular convolutions map cleanly onto the dataflow array, but sustains it at 2.5 W against BoltNet's 1.6 W. **MobileViT V2** comes last everywhere, about 19× below BoltNet on the NPU, because attention has no efficient edge kernel.

BoltNet is the only network that holds up on all three. The differences follow **arithmetic intensity** rather than nominal cost: what the SRB leaves to compute is dense convolution, so the accelerator stays busy.

## Accuracy-Compression Tradeoff

$$
\mathrm{ACT} = \left(\frac{\mathrm{acc}_{\mathrm{comp}}}{\mathrm{acc}_{\mathrm{orig}}}\right)^{k} \cdot \log_{2}\left(1 + \frac{p_{\mathrm{orig}}}{p_{\mathrm{comp}}}\right)
$$

ACT is a dimensionless diagnostic: the fidelity term penalises accuracy loss non-linearly, with the exponent $$k$$ setting how strongly fidelity outweighs size, and the efficiency term rewards compression with diminishing returns.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BoltNet/act.png" title="ACT across model families" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    In each family the thick orange line is the member that leads at k = 7, and the dashed grey line is the reference it is measured against.
</div>

The exponent is calibrated rather than assumed: at **k = 7** the metric settles on **ResNet-50** within the ResNet family and moves the EfficientNet peak from B0 to **B3**, both of which match the accepted practical choice in those families, and that same k is then applied to the BoltNet ablation. ACT is intended **exclusively as a diagnostic tool**, to identify within one family of models the architecture that best exploits its parameter budget.

## Reproducibility

The released checkpoints are evaluated on the full official test sets, 31,112 images for Pl@ntNet-300K, 1,654 for AIDERv2 and 3,000 for CLRS, with a simple resize to 224 × 224 and dataset-specific normalisation, no test-time augmentation. The repository ships with [AutoDock]({{ '/projects/AutoDock/' | relative_url }}) for environment setup:

```bash
cd AutoDock
./build.sh
./run.sh -d /path/to/data

cd src
python main.py --config-path configs/local/plantnet300k/BoltNet.yml
```

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://codeberg.org/danielrossi/BoltNet" class="btn btn-primary z-depth-1">
            <i class="fas fa-code-branch"></i> Source Code
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/BoltNet" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> GitHub
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://arxiv.org/abs/2608.11844" class="btn btn-primary z-depth-1">
            <i class="fas fa-file-pdf"></i> Read Paper
        </a>
    </div>
</div>
