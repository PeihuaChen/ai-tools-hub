---
title: "ComfyUI 入门完全指南：从零开始的 Stable Diffusion 节点式工作流 - AI知识宇宙 - 用AI采集，用AI生成"
date: 2026-05-19
category: "AI 工具实战"
description: ""
read_time: "2分钟"
tags: [AI 工具，AI 工具实战]
---

AI图像生成
                2026-05-16 · 5 分钟
            
            
## 引言：为什么 ComfyUI 正在改变 AI 图像生成的方式


在 Stable Diffusion 生态中，AUTOMATIC1111 的 WebUI 曾经是绝对的主流选择。然而，2025-2026 年间，ComfyUI 以惊人的速度崛起，正在成为越来越多 AI 创作者的首选工具。ComfyUI 采用节点式工作流（Node-based Workflow）的设计理念，将图像生成过程拆解为一个个可视化的功能节点，通过连线连接形成完整的工作流管道。这种「可视化编程」的方式，将为追求精确控制和高效率的 AI 创作者带来全新的体验。如果你是 Stable Diffusion 的初学者，或者正在从 WebUI 转向 ComfyUI，本文将是你的最佳起点。

## 第一章：ComfyUI 的核心概念

### 什么是节点式工作流？


传统的 AI 图像生成工具（如 Midjourney、DALL-E 3）将整个图像生成过程封装成一个「黑盒」——你输入提示词，点击生成，得到图像。这种方式简单直观，但你无法干预中间过程。ComfyUI 的做法恰恰相反：它将图像生成过程拆解为多个独立的「节点」，每个节点负责一个特定的功能——加载模型、编写提示词、设置采样参数、进行图像放大等。用户通过拖拽和连线，将这些节点连接成一个可视化的工作流。这样一来，你不仅知道每一步发生了什么，还可以随时调整任意一个节点的参数。这种透明度和控制力，正是追求极致的 AI 创作者所需要的。

### 基本节点类型


在 ComfyUI 中，你会遇到以下核心节点类型：


- **Checkpoint Loader（模型加载器）**：加载 Stable Diffusion 基础模型

- **CLIP Text Encode（文本编码器）**：将提示词转换为模型理解的向量表示

- **KSampler（采样器）**：核心的图像生成节点，控制采样过程

- **VAE Decode/Encode（变分自编码器）**：图像的编码和解码

- **Empty Latent Image（空潜空间图像）**：定义生成图像的尺寸和批次

- **Save Image（图像保存）**：将生成的图像保存到本地





每个节点都有输入端口（左侧）和输出端口（右侧），通过连线将一个节点的输出连接到另一个节点的输入，就构成了完整的数据流管道。这种设计让整个图像生成过程变得完全透明和可控。

## 第二章：环境搭建与安装

### 系统要求


ComfyUI 对硬件的要求与 WebUI 基本一致：


- **GPU**：NVIDIA GTX 1060 6GB 或更高（推荐 RTX 3060 12GB+）

- **内存**：16GB RAM

- **存储**：至少 20GB 可用空间（模型文件逐渐增大）

- **操作系统**：Windows 10/11、Linux、macOS（Apple Silicon 支持良好）




### 安装步骤


**Windows 用户**最简便的方式是使用整合包。推荐从 ComfyUI 的官方 GitHub 仓库下载 Windows 直装版：

```git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
python main.py

```



ComfyUI 会自动检测你的 Python 环境和 GPU 配置，完成依赖安装后即可启动。浏览器访问 http://127.0.0.1:8188 即可进入工作界面。


**macOS（Apple Silicon）用户**需要注意安装原生支持的 PyTorch 版本。推荐使用以下命令：

```pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/nightly/cpu

```


### 模型安装


ComfyUI 的模型目录结构与 WebUI 类似：


- 基础模型放在 ComfyUI/models/checkpoints/

- LoRA 模型放在 ComfyUI/models/loras/

- ControlNet 模型放在 ComfyUI/models/controlnet/

- VAE 模型放在 ComfyUI/models/vae/





如果你已经安装了 WebUI，可以通过创建符号链接的方式共享模型文件，避免重复下载。这是一个非常实用的技巧，可以节省大量的磁盘空间和下载时间。

## 第三章：第一个工作流——基础文生图

### 步骤 1：创建基本节点


打开 ComfyUI 后，你首先会看到一个空白的画布。右键点击空白处，选择「Add Node」。你需要添加以下节点：


- **Checkpoint Loader**（从「loaders」分类中找到）

- **CLIP Text Encode × 2**（用于正面和负面提示词）

- **Empty Latent Image**（设置图像尺寸）

- **KSampler**（核心采样节点）

- **VAE Decode**（将潜空间数据解码为图像）

- **Save Image**（保存输出）




### 步骤 2：连接节点


将节点按照以下顺序连接（从输出端口拖到输入端口）：


- Checkpoint Loader（model）→ KSampler（model）

- Checkpoint Loader（clip）→ CLIP Text Encode（clip）

- CLIP Text Encode（正面提示词）→ KSampler（positive）

- CLIP Text Encode（负面提示词）→ KSampler（negative）

- Empty Latent Image（latent）→ KSampler（latent_image）

- KSampler（latent）→ VAE Decode（samples）

- Checkpoint Loader（vae）→ VAE Decode（vae）

- VAE Decode（image）→ Save Image（images）




### 步骤 3：设置参数并生成


在每个节点中设置参数：


- **正面提示词**：输入你想要的图像描述

- **负面提示词**：输入你不想要的元素

- **KSampler**：设置 seed（随机种子）、steps（采样步数，推荐 20-30）、cfg（推荐 7-8）、sampler_name（推荐 DPM++ 2M Karras）

- **Empty Latent Image**：设置宽高（推荐 1024×1024）





点击右下角的「Queue Prompt」按钮，开始生成！恭喜你，你已经完成了第一个 ComfyUI 工作流。

## 第四章：进阶工作流实战

### 工作流一：图生图（Image-to-Image）


图生图工作流与文生图的区别在于，需要增加一个「Load Image」节点和一个「VAE Encode」节点：加载参考图像 → VAE 编码 → 连接 KSampler 的 latent_image 输入。然后调整 KSampler 的 denoise 参数（去噪强度）：0.3-0.5 为轻微修改，0.5-0.7 为中度修改，0.7-0.9 为大幅重绘。图生图在风格迁移、图像修复和创意改编等场景中有着广泛的应用。

### 工作流二：高清放大（Upscale）


使用「Model Merge」节点或者「Ultimate SD Upscale」节点实现：先加载生成的低分辨率图像，通过放大节点处理，可配合 ControlNet Tile 节点保持细节。放大倍数建议设置为 2x 或 4x，过高的放大倍数可能导致失真。高清放大是 ComfyUI 的优势领域，因为你可以通过节点组合实现比 WebUI 更精细的放大控制。

### 工作流三：ControlNet 系统


以 Canny ControlNet 为例：加载参考图像 → 通过 Canny 边缘检测节点提取边缘 → 连接 ControlNetLoader 加载模型 → 将控制信号输入 KSampler 的额外输入端口。ControlNet 是 ComfyUI 中最能体现节点式优势的功能。通过可视化连线，你可以清晰地看到控制信号如何影响生成过程，调整起来直观方便。

## 第五章：效率提升技巧

### 工作流模板化


创建好一个工作流后，点击「Save」按钮保存为 JSON 文件。下次需要时直接加载，无需重新搭建。你还可以从社区（如 OpenArt、Civitai）下载其他人分享的工作流模板导入使用。这极大地降低了 ComfyUI 的学习成本。

### 批量生成


使用「Batch」功能可以批量生成不同变体的图像。你可以将种子设置为「-1」（随机），或者使用「Latent Batch」节点同时生成多张图像。批量生成时，建议先在小规模测试中确定最佳参数，再进行大规模批量生产。

### 性能优化


- 使用 --xformers 启动参数启用内存优化

- 使用 --lowvram 参数在 VRAM 不足时自动优化

- 在 KSampler 中选择「LCM」或「Turbo」采样器可以大幅减少采样步数（4-8 步即可）

- 合理使用 Latent Cache 可以避免重复计算，提升批量生成效率





**👉 如果你的本地硬件配置不足，推荐使用云 GPU 服务。通过我们的专属链接注册 RunPod，快速开始 ComfyUI 之旅：https://example.com/runpod-comfyui**

## 第六章：社区资源推荐


ComfyUI 的社区生态在 2026 年已经非常成熟。推荐以下资源：ComfyUI Examples GitHub 提供了官方示例工作流集合；Civitai 的 ComfyUI 板块有大量模型和对应工作流可供下载；Reddit 的 r/ComfyUI 是活跃的社区讨论区；YouTube 上搜索「ComfyUI tutorial for beginners」可以找到丰富的视频教程资源。遇到问题时，GitHub Issues 和 Discord 社区也是寻求帮助的好去处。社区的发展速度非常快，几乎每天都有新的工作流和插件发布，保持关注可以帮助你始终处于技术前沿。

## 总结：值得投入学习的强大工具


ComfyUI 的学习曲线确实比 WebUI 更陡峭，但一旦你理解了节点式工作流的思维方式，就会发现它带来的创作自由度和效率提升是值得付出的。对于需要精确控制生成过程、构建复杂工作流或进行批量生产的用户来说，ComfyUI 是最佳选择。我们建议初学者先从基础的文生图工作流开始，逐步尝试图生图、ControlNet 和批量生成等进阶功能。随着经验的积累，你将能够构建出越来越复杂和高效的工作流，将 AI 图像生成的能力发挥到极致。


*本文由 AI工具评测组 原创，教程内容基于 2026 年 5 月的 ComfyUI 最新版本。*
            
            
            
                

🛒 京东 挑好物逛京东(Skechers杯具)
                

京东大促进行中，超值好物等你来选
                [去京东看看 →](https://u.jd.com/H1BFc81)
                

* 京东联盟推广链接，你不需要多花钱，我们获得小额佣金