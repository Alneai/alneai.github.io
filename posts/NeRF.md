---
abbrlink: ''
categories: []
date: '2024-06-25'
description: ''
tags:
- NeRF
title: NeRF
updated: '2025-02-13T17:00:50.000+08:00'
---
# 论文

## $T^3$ Bench

$T^3$ 指标

## Lgm: Large multi-view gaussian model for high-resolution 3d content creation

大模型+NeRF

## MVGamba: Unify 3D Content Generation as State Space Sequence Modeling

基于多视角图像后处理的大模型训练

a one image (or text) → mulit-view images → 3D diagram to predict 3DGS parameters

核心：现有的前馈高斯重建模型生成长序列的高斯，损失了损害多视图信息传播的完整性。**直接生成具有完整多视图信息的足够长的高斯序列对于一致且精细的 3D 生成至关重要。**

## GaussianCube

密度约束高斯拟合算法

## Mip-Splatting: Alias-free 3D Gaussian Splatting

*CVPR 2024*

引入了一个3D平滑滤波器，根据输入视图引起的最大采样频率来限制3D Gaussian primitive的大小，从而消除放大时的高频伪影。

## An Image is Worth 32 Tokens for Reconstruction and Generation

图像编解码器，压缩图像 token 数

## GradeADreamer: Enhanced Text-to-3D Generation Using Gaussian Splatting and Multi-View Diffusion

https://arxiv.org/abs/2406.09850

该管道仅使用单个 RTX 3090 GPU 即可生成高质量资产，总生成时间不到 30 分钟。我们提出的方法采用多视图扩散模型 MVDream 来生成高斯 Splats 作为先验，然后使用 StableDiffusion 细化几何和纹理。

## idea

- Mamba / else 实现低能耗，LGM/MVGamba作为Baseline，降低训练速度10倍，尝试PSNR接近，推理时间接近
- NeRF/3DGS 在大模型训练上的优劣
