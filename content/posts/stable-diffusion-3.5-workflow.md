---
title: "Stable Diffusion 3.5 完整工作流指南：从入门到精通的实用教程 - AI知识宇宙 - 用AI采集，用AI生成"
date: 2026-05-19
category: "AI 工具实战"
description: ""
read_time: "1分钟"
tags: [AI 工具，AI 工具实战]
---

AI图像生成
                2026-05-16 · 5 分钟
            
            
## 引言：开源 AI 图像生成的王者归来


Stable Diffusion 3.5 在 2026 年的发布，标志着开源 AI 图像生成模型迈入了一个全新的阶段。相比前代产品，SD 3.5 在图像质量、文本渲染能力和语义理解方面实现了质的飞跃，同时在硬件要求上保持了相对友好的门槛。对于追求自定义和可控制性的创作者来说，Stable Diffusion 仍然是不可替代的选择。本文将从零开始，手把手教你搭建和使用 Stable Diffusion 3.5 的完整工作流。

## 第一章：SD 3.5 的新特性概览

### 全新架构：MMDiT 与增强的文本编码器


Stable Diffusion 3.5 采用了改进的 MMDiT（多模态扩散 Transformer）架构，这是一种专门针对文本到图像生成优化的神经网络结构。相比于 SD XL 的 UNet 架构，MMDiT 在处理复杂文本描述时的表现大幅提升，特别是在文本渲染（生成图像中的文字）和空间关系理解方面。此外，SD 3.5 使用了三文本编码器系统——CLIP L/14、T5-XXL 和 OpenCLIP bigG——的组合，这使得模型对中文、日文等非拉丁文字的支持得到了显著改善。对于需要使用中文提示词的用户来说，这无疑是一个重大的利好消息。我们专门测试了 SD 3.5 中文提示词的效果：直接用中文描述「一只橘猫坐在窗台上，窗外是雪景」——模型准确地理解了中文含义并生成了符合预期的图像，这是之前版本难以做到的。

### 硬件要求与模型变体


SD 3.5 提供了多种模型变体以适应不同的硬件配置：


- **SD 3.5 Medium（25亿参数）**：消费级 GPU（8GB+ VRAM）即可运行

- **SD 3.5 Large（80亿参数）**：推荐 16GB+ VRAM，质量最佳

- **SD 3.5 Turbo（蒸馏版本）**：仅需 4 步采样即可生成高质量图像，适合实时应用





对于大多数用户，我们推荐从 Medium 版本开始尝试，在熟悉工作流后再升级到 Large 版本以获得最佳图像质量。Turbo 版本则适合需要快速迭代和实时反馈的场景，比如在直播或在线演示中使用。

## 第二章：本地环境搭建

### Step 1：基础环境配置


要运行 Stable Diffusion 3.5，你需要准备以下硬件和软件：


**硬件要求（最低）**：
- GPU：NVIDIA GeForce RTX 3060 或更高，至少 8GB VRAM
- 内存：16GB RAM
- 存储：50GB 可用空间（用于模型文件和生成内容）


**软件要求**：
- Python 3.10 或更高版本
- Git
- NVIDIA 驱动和 CUDA 工具包

### Step 2：安装 Stable Diffusion WebUI


目前最推荐的 SD 3.5 运行方式是使用 AUTOMATIC1111 的 Stable Diffusion WebUI（已更新适配 SD 3.5）或 ComfyUI。以下是基于 WebUI 的安装步骤：

```git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
python launch.py --xformers

```



首次启动时需要下载基础模型。推荐从 Hugging Face 下载 Stability AI 官方发布的 SD 3.5 Medium 模型文件，放置到 models/Stable-diffusion 目录下。

### Step 3：安装 ControlNet 等扩展


为了让 SD 3.5 发挥最大潜力，建议安装以下扩展：


- **ControlNet**：提供姿态控制、边缘检测、深度图等精确控制能力

- **ADetailer**：自动面部和手部修复，大幅提升人物生成质量

- **Dynamic Thresholding**：改善 CFG 缩放

- **Ultimate SD Upscale**：高质量图像放大




## 第三章：核心工作流详解

### 工作流一：文生图（Text-to-Image）


文生图是最基础的工作流。在 SD 3.5 中，推荐以下参数设置作为起点：


- **采样器**：Euler a 或 DPM++ 2M Karras

- **采样步数**：20-30 步（Medium）/ 25-40 步（Large）

- **CFG Scale**：7.0-9.0（提示词文本复杂时建议降低到 4.0-6.0）

- **图像尺寸**：1024×1024（推荐），或 1024×768、768×1024

- **Negative Prompt**：建议写入不希望出现的元素（如「低质量、模糊、畸形、多余的手指」等）





参数的选择需要根据具体的生成任务进行调整。我们建议初学者先使用默认参数，然后逐步调整以找到最适合自己的配置组合。记录每次生成的参数设置和结果，有助于你快速建立经验。

### 工作流二：图生图（Image-to-Image）


图生图工作流允许你以已有图像为基础进行修改和再创作。核心参数是 Denoising Strength（去噪强度）：


- **0.3-0.5**：轻微修改，保持原图结构

- **0.5-0.7**：中度修改，改变风格和细节

- **0.7-0.9**：大幅修改，几乎完全重绘





图生图功能在风格迁移、图像修复和创意改编等场景中有着广泛的应用。你可以用它将一张普通的照片转化为油画风格的图像，也可以在已有草图的基础上让 AI 补充细节。

### 工作流三：ControlNet 精确控制


ControlNet 是 SD 生态中最强大的功能之一。以下是几种常用的控制方式：


**Canny（边缘检测）**：从参考图像提取边缘线稿，控制生成图像的构图。适合需要精确控制物体轮廓的场景。**OpenPose（姿态估计）**：检测人物姿态骨骼图，确保生成的人物姿势符合要求。对于角色设计非常有用。**Depth（深度图）**：使用深度信息控制图像的空间结构，保证透视和前后关系的准确性。**Scribble（涂鸦）**：你简单画几笔线条，AI 就能根据涂鸦生成完整的图像。非常适合快速概念可视化。这些控制方式的组合使用可以产生令人惊叹的效果，比如用 OpenPose 控制人物姿势、同时用 Canny 控制背景构图。

### 工作流四：高清放大


SD 3.5 原生生成的 1024×1024 图像有时需要放大以用于打印或高清展示。推荐使用 Hires.fix 进行第一轮放大，然后使用 Ultimate SD Upscale 进行第二轮放大。配合 RealESRGAN 模型，可以将图像放大到 4K 甚至 8K 分辨率而不损失细节。这对于需要大幅面输出或印刷用途的用户来说至关重要。

## 第四章：高级技巧与最佳实践

### 提示词工程


高质量的提示词是生成好图像的关键。SD 3.5 的提示词结构建议如下：先写主体描述，然后是环境背景、光照条件、颜色方案、艺术风格、材质质感、构图角度和情绪氛围。将这些要素按照优先级从高到低的顺序组织起来，可以帮助模型更好地理解你的意图。例如：a serene Japanese garden in autumn, golden maple leaves reflecting on a koi pond, soft afternoon sunlight filtering through branches, watercolor style, painterly textures, aerial view, peaceful and contemplative mood

### LoRA 模型的训练与使用


LoRA（低秩适应）是 SD 生态中不可或缺的一部分。你可以下载社区训练的 LoRA 模型来添加特定风格（如吉卜力风格、水墨画风格）或特定角色/人物特征。LoRA 模型放置在 models/Lora 目录下，在 WebUI 的附加网络面板中加载即可使用。如果你想训练自己的 LoRA，推荐使用 Kohya's GUI 工具。训练一个高质量的 LoRA 通常需要 20-50 张高质量的训练图像。

## 第五章：社区资源与学习路径


Stable Diffusion 的最大优势之一是其活跃的社区生态。推荐以下资源：Civitai（https://civitai.com）是最大的 SD 模型和 LoRA 分享平台，你可以在上面浏览和下载成千上万的社区模型；Hugging Face（https://huggingface.co）提供官方模型和社区模型仓库；Reddit 的 r/StableDiffusion 是技术讨论和作品分享的活跃社区；YouTube 上搜索「SD 3.5 workflow tutorial」可找到大量视频教程。加入这些社区不仅可以帮助你解决使用中遇到的问题，还能让你及时了解最新的模型和技巧。


**👉 想要获得更好的 GPU 算力支持？通过我们的推荐链接注册 RunPod，首充 50 美元赠送 25 美元：https://example.com/runpod**

## 总结：值得投入时间学习的强大工具


Stable Diffusion 3.5 是当前最强大的开源 AI 图像生成工具。虽然它的初始学习曲线比 Midjourney 或 DALL-E 3 更陡峭，但一旦掌握了基本工作流，你将获得无与伦比的创作自由度。从精确的构图控制到独特的艺术风格，从本地部署的隐私保护到零内容限制的创作自由，SD 3.5 为认真对待 AI 艺术的创作者提供了最完整的工具集。我们鼓励每一位对 AI 图像生成感兴趣的读者都花一些时间来学习 SD 3.5，它所提供的创作自由度和控制能力是其他工具无法比拟的。


*本文由 AI工具评测组 原创，教程内容基于 2026 年 5 月的 SD 3.5 最新版本。*
            
            
            
                

🛒 京东 尊尼获加&元气森林
                

京东大促进行中，超值好物等你来选
                [去京东看看 →](https://u.jd.com/Hrps3pr)
                

* 京东联盟推广链接，你不需要多花钱，我们获得小额佣金