# vxpg
code for SIGGRAPH 2024 paper "Real-Time Path Guiding Using Bounding Voxel Sampling".

project page: [https://suikasibyl.github.io/vxpg](https://suikasibyl.github.io/vxpg)

![](https://suikasibyl.github.io/files/vxpg/teaser.webp "teaser")

---
```
Noteice: The current codebase is a bit messy,
and there could be problems to build the project.
Please contact me through hal128@ucsd.edu for any questions.
I'll probably upload a clean and more easy to build version latter.
```

Our implementation bases on my toy renderer [SIByL2023](https://github.com/SuikaSibyl/SIByLEngine2023), and <u>the code is already in the 👈 repo</u>.

Here is a walkthrough of relevant files and codes:

---
### Render Graphs

The render pipelines in ```SIByL``` are defined and scheduled with Render Dependency Graph (RDG).

In the paper, we present 3 variants of vxpg pipeline:

#### 1. vanilla vxpg [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Sandbox/src/CustomPipeline.hpp#L1063)]
| High-Level Pipeline | Actual Pass | Description | Code |
|------|-------|-------|----|
| Render VBuffer | RayTraceVBuffer | render and hold V-Buffer for further reuse of 1st shading points | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VBuffer/Private/SE.Addon.VBuffer.cpp#L5)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vbuffer/raytraced-vbuffer.slang)] |
| Geometry Injection | PrebakeDummyPass | load prebake geometry information from external resources | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Public/SE.Addon.VXGuiding.hpp#L183)]|
|  | VXGuiderClearPass | clear relevant resources for geometry & light injection | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L9)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/vxguiding-clear.slang)] |
|  | VXGuiderGeometryDynamicPass | dynamic objects geometry injection | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L1716)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/vxguiding-geome-injection.slang)] 
| Light Injection | VXGuider1stBounceInjection  | draw 1st path sample, inject the irradiance to voxels | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L2286)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/vxguiding-camera-injection.slang)]|
|  | VXGuiderCompactPass  | compact all populated voxels for faster manupulation later | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L1516)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/vxguiding-compact.slang)]|
| Superpixel Clustering | InitClusterCenterPass | set 64x64 pixel tile center as cluster center |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/gSLICr/Private/SE.Addon.gSLICr.cpp#L5)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/gSLICr/init-cluster-centers.slang)] |
|   | FindCenterAssociationPass | use gSLICr to find nearst pixel cluster in local 3x3 field | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/gSLICr/Private/SE.Addon.gSLICr.cpp#L75)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/gSLICr/find-center-association.slang)]|
| Supervoxel Clustering  | VXInfoClearPass | clear data relevant to voxel clustering | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5081)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/vxinfo-clear.slang)]|
|  | RowColumnPresamplePass | randomly select 1 pixel per superpixel & 1 vertex per populated voxel | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L7478)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/row-column-presample-ex.slang)]|
|  | RowVisibilityPass | for each voxel, use the selected vertex to evaluate the mutual visibility with every representative pixels proposed in 👆 pass; then pack the visiblity as a unsigned int bitfield |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5189)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/mrcs/row-visibility.slang)]|
|  | RowKmppCenterPass | use kmpp to initialize voxel cluster center by visibility bitfield | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5259)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/column-kmpp-seeding.slang)]|
|  | RowFindCenterPass | find the nearest voxel cluster, which has most similar visibility bitfield  |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5334)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/column-find-center.slang)] |
| Supervoxel Merging | VXTreeEncodePass | encode the voxels with cluster and position information  |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4542)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/column-find-center.slang)] |
|   | BitonicSort Subgraph | sort the voxels as tree leaves for parallel tree building  |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/BitonicSort/Private/SE.Addon.BitonicSort.cpp)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/bitonicsort)] |
|  | VXTreeIIntializePass | initialize tree leaves and nodes for parallel tree building |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4542)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/tree/tree-initial-pass.slang)] |
|  | VXTreeInternalPass | and this is parallel binary tree building |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4306)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/tree/tree-internal-pass.slang)] |
|  | VXTreeMergePass | merge voxel nodes along the binary tree to get cluster information (bounding box, irradiance sum)|  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4378)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/tree/tree-merge-pass.slang)] |
| LightSlice Building |  VXInfoRearrangePass | rearange voxel information for further visibility check |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5441)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/mrcs/vxinfo-rearrange.slang)] |
|   |  SPixelClearPass | clear the data structure relavent to superpixel samples |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4749)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/visibility/cvis-clear.slang)] |
|   |  SPixelGatherPass | each superpixel draw some sample pixels from its domain |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4813)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/visibility/cvis-assignment.slang)] |
|   |  SPixelVisibilityEXPass | and we compute mutual average visibility and contributions between each pair of superpixel and supervoxel |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L5643)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/visibility/cvis-visibility-check-comp.slang)] |
|   |  VXTreeTopLevelPass | build the sampling distribution for LightSlice |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L4444)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/tree/tree-top-level-constr.comp)] |
|  Path Tracing!! |  VXGuiderGIPass | do adaptive guided sample, and MIS here |  [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VXGuiding/Private/SE.Addon.VXGuiding.cpp#L133)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vxguiding/vxguiding-gi.slang)] |

#### 2. ReSTIR GI + vxpg[[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Sandbox/src/CustomPipeline.hpp#L1458)]

| High-Level Pipeline | Actual Pass | Description | Code |
|------|-------|-------|----|
| Render VBuffer | RayTraceVBuffer | render and hold V-Buffer for further reuse of 1st shading points | [[cpp](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Source/Addon/VBuffer/Private/SE.Addon.VBuffer.cpp#L5)][[slang](https://github.com/SuikaSibyl/SIByLEngine2023/blob/8c1adad28c16e07592bc79122cb9d0c9738699bc/Engine/Shaders/SRenderer/addon/vbuffer/raytraced-vbuffer.slang)] |

#### 3. A-SVGF + vxpg


### Passes
