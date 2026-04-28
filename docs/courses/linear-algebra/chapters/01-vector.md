---
title: 第1章 向量与向量空间
date: 2026-04-27
authors:
  - name: Lei
    title: 课题组负责人
    avatar: https://github.com/leicaohmu.png
---

## 1.1 向量的定义

向量是既有**大小**又有**方向**的量。在 $n$ 维空间中，向量表示为：

$\mathbf{v} = \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix}$

---

## 1.2 向量运算

### 加法

$\mathbf{u} + \mathbf{v} = \begin{pmatrix} u_1 + v_1 \\ u_2 + v_2 \end{pmatrix}$

### 数乘

$c\mathbf{v} = \begin{pmatrix} cv_1 \\ cv_2 \end{pmatrix}$

### 内积（点积）

$\mathbf{u} \cdot \mathbf{v} = \sum_{i=1}^{n} u_i v_i = \|\mathbf{u}\| \|\mathbf{v}\| \cos\theta$

---

## 1.3 向量空间

满足以下条件的集合 $$V$$ 称为**向量空间**：

1. 对加法封闭：$\mathbf{u}, \mathbf{v} \in V \Rightarrow \mathbf{u} + \mathbf{v} \in V$
2. 对数乘封闭：$\mathbf{v} \in V, c \in \mathbb{R} \Rightarrow c\mathbf{v} \in V$

### 常见子空间

| 子空间 | 定义 |
|--------|------|
| 列空间 $C(A)$ | 矩阵 $A$ 所有列向量的线性组合 |
| 零空间 $N(A)$ | 满足 $A\mathbf{x}=\mathbf{0}$ 的所有 $\mathbf{x}$ |
| 行空间 $C(A^T)$ | 矩阵 $$A$$ 所有行向量的线性组合 |

---

## 1.4 线性相关与无关

若存在不全为零的系数使得：

$c_1\mathbf{v}_1 + c_2\mathbf{v}_2 + \cdots + c_k\mathbf{v}_k = \mathbf{0}$

则称这组向量**线性相关**，否则称**线性无关**。

---

!!! tip "小结"
    本章介绍了向量的基本运算和向量空间的概念。下一章将进入矩阵运算。