# Mass Spring Cloth Simulation

A GPU-accelerated cloth simulation using Taichi framework, implementing mass-spring model with three numerical integration methods.

## Features

- **Three Integration Methods**: Explicit Euler, Semi-Implicit Euler, and Implicit Euler
- **Spring Types**: Structural, Shear, and Bending springs
- **Collision Detection**: Sphere collision for cloth simulation
- **Interactive GUI**: Real-time method switching, pause, and reset controls
- **GPU Acceleration**: Powered by Taichi Vulkan backend

## Installation

```bash
pip install taichi
```

## Running

```bash
python mass_spring.py
```

## Controls

- **Left Mouse Button + Drag**: Rotate camera
- **Right Mouse Button + Drag**: Move camera
- **Scroll**: Zoom in/out

### GUI Panel

- Switch between three integration methods
- Pause/Resume simulation
- Reset cloth

## Physics Model

### Mass-Spring System

Each cloth is modeled as a grid of mass points connected by springs:

- **Structural Springs**: Connect adjacent points (horizontal and vertical)
- **Shear Springs**: Connect diagonal points
- **Bending Springs**: Connect points with one point in between

### Force Calculation

Hooke's Law for spring force:
```
f_spring = -k_s * (|x_a - x_b| - l) * (x_a - x_b) / |x_a - x_b|
```

Damping force:
```
f_damping = -k_d * v
```

### Numerical Integration

1. **Explicit Euler**: `x_{t+1} = x_t + v_t * dt`, `v_{t+1} = v_t + a_t * dt`
2. **Semi-Implicit Euler**: `v_{t+1} = v_t + a_t * dt`, `x_{t+1} = x_t + v_{t+1} * dt`
3. **Implicit Euler**: Solved using fixed-point iteration

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| N | 20 | Grid resolution |
| mass | 1.0 | Particle mass |
| dt | 5e-4 | Time step |
| k_s | 10000.0 | Spring stiffness |
| k_d | 1.0 | Damping coefficient |
| gravity | [0, -9.8, 0] | Gravity vector |
| max_velocity | 50.0 | Velocity clamp |

## Project Structure

```
.
├── mass_spring.py    # Main simulation code
└── README.md         # This file
```

## Experiment Objectives

1. Master dynamic scene rendering with Taichi GGUI
2. Understand mass-spring model with physics-based forces
3. Compare numerical integration methods (Explicit/Semi-Implicit/Implicit Euler)
4. Learn GPU programming basics with `ti.kernel` and `ti.func`

## References

- Taichi Framework: https://github.com/taichi-dev/taichi
- Games101 Assignment 4: Mass Spring System
