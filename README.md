# 质点-弹簧布料模拟系统
姓名：唐悦婷 学号：202311061051 专业：计算机科学与技术（公费师范）

基于 Taichi 框架实现的 GPU 加速布料物理模拟系统，包含三种数值积分方法对比实验。

---

## 项目结构

```
.
├── mass_spring.py    # 主程序 - 完整的布料物理模拟实现
└── README.md         # 项目说明文档
```

---

## 实验目标

1. **掌握动态场景渲染**：使用 Taichi 框架构建 3D 场景，学习 Taichi GGUI 交互面板编程
2. **理解质点-弹簧模型**：掌握基于物理的弹力与阻尼力计算方法，处理数值爆炸问题（速度钳制）
3. **对比数值积分方法**：实现三种常见积分求解器（显式欧拉、半隐式欧拉、隐式欧拉），观察稳定性差异
4. **理解 GPU 编程基础**：学习 `ti.kernel` 与 `ti.func`，了解并行计算中的状态同步与 Kernel 启动开销优化

---

## 功能特性

### 物理模型

- **结构弹簧（Structural）**：连接相邻质点，维持布料基本形状
- **剪切弹簧（Shear）**：对角线方向连接，防止剪切变形
- **弯曲弹簧（Bending）**：隔一个质点连接，增加弯曲刚度
- **球体碰撞检测**：布料与球体的碰撞响应

### 数值积分方法

| 方法 | 稳定性 | 特点 |
|------|--------|------|
| 显式欧拉 | 极易发散 | 计算简单，但时间步长受限 |
| 半隐式欧拉 | 相对稳定 | 保持能量守恒，适合大多数场景 |
| 隐式欧拉 | 无条件稳定 | 使用定点迭代法，有阻尼效果 |

### 交互功能

- 实时切换三种积分方法
- 暂停/恢复模拟
- 重置布料状态
- 相机视角控制（旋转、平移、缩放）

---

## 快速开始

### 环境要求

- Python 3.8+
- Taichi 1.7.4+

### 安装依赖

```bash
pip install taichi
```

### 运行程序

```bash
python mass_spring.py
```

---

## 操作指南

### GUI 控制面板

| 按钮 | 功能 |
|------|------|
| Explicit Euler | 切换到显式欧拉方法（不稳定） |
| Semi-Implicit Euler | 切换到半隐式欧拉方法（推荐） |
| Implicit Euler | 切换到隐式欧拉方法（有阻尼） |
| Pause Simulation | 暂停模拟 |
| Resume Simulation | 恢复模拟 |
| Reset Cloth | 重置布料到初始状态 |

---

## 物理原理

### 质点-弹簧模型

布料被离散化为网格状的质点集合，质点之间通过弹簧相连。根据胡克定律，两个质点之间的弹力为：

```
f_spring = -k_s * (|x_a - x_b| - l) * (x_a - x_b) / |x_a - x_b|
```

其中：
- `k_s` 为弹簧劲度系数
- `l` 为弹簧原长
- `x_a`, `x_b` 为两个质点的位置

### 阻尼力

为防止系统能量无限增加导致发散，引入阻尼力：

```
f_damping = -k_d * v
```

其中 `k_d` 为阻尼系数，`v` 为质点速度。

### 数值积分

根据牛顿第二定律 `F = ma`，在离散时间步 `dt` 内更新质点状态：

**1. 显式欧拉（Explicit Euler）**
```
x_{t+1} = x_t + v_t * dt
v_{t+1} = v_t + a_t * dt
```
完全使用当前时刻状态，易发散。

**2. 半隐式欧拉（Semi-Implicit Euler）**
```
v_{t+1} = v_t + a_t * dt
x_{t+1} = x_t + v_{t+1} * dt
```
先用旧力更新速度，再用新速度更新位置，相对稳定。

**3. 隐式欧拉（Implicit Euler）**
```
v_{t+1} = v_t + a_{t+1} * dt
x_{t+1} = x_t + v_{t+1} * dt
```
使用未来时刻的力，通过定点迭代法近似求解，无条件稳定。

---

## 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `N` | 20 | 布料网格分辨率（N × N） |
| `mass` | 1.0 | 质点质量 |
| `dt` | 5e-4 | 时间步长 |
| `k_s` | 10000.0 | 弹簧劲度系数 |
| `k_d` | 1.0 | 阻尼系数 |
| `gravity` | [0, -9.8, 0] | 重力加速度向量 |
| `max_velocity` | 50.0 | 速度上限（防止数值爆炸） |

---

## 技术实现

### GPU 并行计算

- 使用 `@ti.kernel` 定义并行计算内核
- 使用 `@ti.func` 定义内联函数，减少函数调用开销
- 使用 `ti.atomic_add` 确保多线程写入安全

### 状态同步

初始化操作拆分为多个 kernel 顺序调用，确保 GPU 计算状态同步：

1. `init_positions()` - 初始化质点位置和固定状态
2. `init_springs()` - 初始化弹簧拓扑结构
3. `init_spring_indices()` - 同步渲染索引

### 性能优化

- 每帧仅启动一次 kernel，最小化启动开销
- 使用 `ti.static` 在编译期展开循环
- 速度钳制防止数值爆炸

---

## 实验对比

### 不同阻尼系数效果

| 阻尼系数 | 效果 |
|----------|------|
| 1.0 | 布料摆动明显，能量耗散较慢 |
| 5.0 | 布料很快稳定，摆动幅度小 |
| 20.0 | 布料摆动幅度最小 |

（无小球）阻尼系数为1 （无小球）阻尼系数为5 

<img width="320" height="480" alt="converted+1" src="https://github.com/user-attachments/assets/07966c1e-8a0b-46d6-856d-bac86e8c48fa" /> <img width="320" height="480" alt="converted+小球5" src="https://github.com/user-attachments/assets/8b5f1ba6-4d2f-4ead-81cf-23a02869027b" />


（有小球）阻尼系数为5 （有小球）阻尼系数为20

<img width="480" height="480" alt="converted+小球5+(2)" src="https://github.com/user-attachments/assets/b9074283-af6d-4df8-b8fc-287a04733755" /> <img width="480" height="480" alt="converted+小球20" src="https://github.com/user-attachments/assets/9ffcf8d7-765d-4209-8fbe-d2f1b7b01643" />



### 积分方法稳定性

- **显式欧拉**：在较大时间步或高刚度下容易发散
- **半隐式欧拉**：保持较好的能量守恒，稳定性适中
- **隐式欧拉**：无条件稳定，但会引入数值阻尼

---

## 选做内容

本项目已实现以下选做功能：

1. **剪切弹簧与弯曲弹簧**：在结构弹簧基础上添加了对角线和隔点连接
2. **球体碰撞检测**：场景中放置球体，实现布料与球体的碰撞响应

---

## 参考资料

- [Taichi 官方文档](https://docs.taichi-lang.org/)
- [Games101 计算机图形学入门](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)
- [Mass-Spring System in Computer Graphics](https://en.wikipedia.org/wiki/Mass-spring_system)


