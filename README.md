# vxpg
code for SIGGRAPH 2024 paper "Real-Time Path Guiding Using Bounding Voxel Sampling".

project page: [https://suikasibyl.github.io/vxpg](https://suikasibyl.github.io/vxpg)

---
```
Noteice: The current codebase is a bit messy, and there could be
problems to build the project. Please contact me through hal128@ucsd.edu.
I'll probably upload a clean and more easy to buildd version latter.
```

Our implementation bases on my toy renderer [SIByL2023](https://github.com/SuikaSibyl/SIByLEngine2023).

Here is a guidance through related files:

### Render Graphs

The render pipelines in SIByL are scheduled with Render Dependency Graph (RDG).

We present 3 variants of vxpg pipeline for our works:

#### 1. vanilla vxpg

#### 2. ReSTIR GI + vxpg

#### 3. A-SVGF + vxpg


### Passes
