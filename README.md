# 2D 地球 · 实时流体风场 / 2D Earth · Real-time Fluid Wind Field

一个纯前端（HTML/CSS/JS）实现的全球实时风场可视化项目。无需 Node.js 或构建工具，直接用浏览器打开 HTML 即可运行。读取云端 Zarr 气象数据，基于 WebGL 渲染动态粒子流线，视觉参考 earth.nullschool。
*A pure frontend (HTML/CSS/JS) global real-time wind field visualization project. No Node.js or build tools required. Reads cloud Zarr meteorological data and renders dynamic particle streamlines via WebGL. Visual reference: earth.nullschool.*

> **声明**：本项目代码、调试过程及文档由 AI (大语言模型) 辅助生成与修复。
> *Disclaimer: The code, debugging process, and documentation of this project are assisted and repaired by AI (Large Language Model).*

## 特性 / Features

- **零后端 / Zero Backend**：依赖通过 CDN (esm.sh, unpkg) 加载，直接连接 Copernicus 云端 Catalog 拉取数据。
  *Dependencies are loaded via CDN (esm.sh, unpkg), directly connecting to the Copernicus Cloud Catalog to fetch data.*
- **动态渲染 / Dynamic Rendering**：基于 `@fxi/zartigl` 和 `MapLibre GL JS`，渲染高密度粒子流线。
  *Based on `@fxi/zartigl` and `MapLibre GL JS` to render high-density particle streamlines.*
- **色彩分级 / Color Grading**：8 阶气象色谱映射（深蓝->青->绿->黄->橙->红），直观反映风速梯度。
  *8-step meteorological color spectrum mapping (Dark Blue -> Cyan -> Green -> Yellow -> Orange -> Red), intuitively reflecting wind speed gradients.*
- **基础交互 / Basic Interaction**：坐标跳转、点击拾取经纬度、刷新实时数据。
  *Coordinate jump, click to pick longitude/latitude, refresh real-time data.*

## 技术栈 / Tech Stack

- 地图引擎 / Map Engine：MapLibre GL JS v5
- 渲染引擎 / Render Engine：@fxi/zartigl v0.5.0
- 数据源 / Data Source：Copernicus Zarr Catalog

## 调试历史 / Debugging History (按问题分类 / Categorized by Issue)

1. **依赖加载与初始化报错 / Dependency Loading & Initialization Error**
   - 现象 / Symptom：控制台报错 `Cannot read properties of undefined (reading 'layers')`。
     *Console error: `Cannot read properties of undefined (reading 'layers')`.*
   - 原因 / Cause：`@fxi/zartigl` 初始化时必须显式传入 `catalog` 对象，且旧版子路径引入在 `esm.sh` 解析失败。
     *`@fxi/zartigl` requires the `catalog` object to be explicitly passed during initialization, and old sub-path imports fail to resolve in `esm.sh`.*
   - 修复 / Fix：统一改用根路径引入，显式 import 并在构造函数中挂载 `catalog`。
     *Unified to use root path imports, explicitly import and mount `catalog` in the constructor.*

2. **极地网格插值失真 / Polar Grid Interpolation Distortion**
   - 现象 / Symptom：高纬度地区（如格陵兰岛附近）出现异常规则的平行线/同心圆。
     *Abnormal regular parallel lines/concentric circles in high latitudes (e.g., near Greenland).*
   - 原因 / Cause：墨卡托投影在极地拉伸严重，放大了低分辨率数据网格的插值错误。
     *Mercator projection stretches severely at the poles, amplifying interpolation errors of low-resolution data grids.*
   - 修复 / Fix：硬编码限制 `maxZoom: 3`，限制放大级别以掩盖数据精度不足。
     *Hardcoded limit `maxZoom: 3` to restrict zoom level and mask insufficient data precision.*

3. **色彩映射失效 / Color Mapping Failure**
   - 现象 / Symptom：全屏颜色偏向青绿色，无法区分气旋强风区和中心弱风区。
     *Screen color is biased towards cyan-green, unable to distinguish strong wind areas and weak wind centers of cyclones.*
   - 修复 / Fix：调整 `palette` 调色板，拉开低风速（暗色）和中等风速（亮色）的对比度。
     *Adjusted the `palette` color palette to increase the contrast between low wind speeds (dark colors) and medium wind speeds (bright colors).*

4. **极地风眼直线伪影 / Polar Eyewall Straight-line Artifact**
   - 现象 / Symptom：风眼中心出现贯穿的直蓝色线。
     *A straight blue line penetrates the center of the eyewall.*
   - 原因 / Cause：Zarr 数据在特定经纬度网格边界缺测（`0` 或 `NaN`），导致粒子经过时速度归零并原地堆积。
     *Zarr data is missing (0 or NaN) at specific latitude/longitude grid boundaries, causing particles to drop to zero speed and accumulate in place when passing through.*
   - 尝试 / Attempts：暗化颜色 -> 缩短拖尾 (`fadeOpacity: 0.85`) -> 提高消亡率 (`dropRate: 0.015`)。
     *Darkened color -> Shortened trail (`fadeOpacity: 0.85`) -> Increased death rate (`dropRate: 0.015`).*
   - 结论 / Conclusion：通过高频动态噪点打散了线条的视觉观感，但未彻底消除（详见 Issue）。
     *High-frequency dynamic noise has visually dispersed the line, but it is not completely eliminated (see Issue).*

## 快速启动 / Quick Start

1. 下载本项目代码。
   *Download the project code.*
2. 确保网络畅通（需访问 CDN 和云端 Zarr 数据）。
   *Ensure a stable network connection (requires access to CDN and cloud Zarr data).*
3. 双击 `index.html`在浏览器中运行。(注意若以本地文件运行可能受浏览器限制,建议挂在建议web服务,例:"python -m http.server 8000")
   *Double-click `index.html` to run in the browser. (Note: Running as a local file may be restricted by the browser. It is recommended to host it on a web service, e.g., "python -m http.server 8000")*

## Issue 模板 / Issue Template (针对未能解决的 Bug / For Unresolved Bugs)

**Title:** [Bug] 极地风眼中心存在无法消除的直线粒子伪影 / Unremovable straight-line particle artifact in polar eyewall center

**Description:**
在观察高纬度气旋（如斯瓦尔巴群岛附近）时，风眼中心会出现一条笔直的深色断层线。
*When observing high-latitude cyclones (e.g., near Svalbard), a straight dark fault line appears in the center of the eyewall.*

**环境 / Environment:**
- 浏览器 / Browser：Chrome / Edge
- 缩放级别 / Zoom Level：Zoom 3 (达到最大限制 / Reached max limit)

**复现步骤 / Steps to Reproduce:**
1. 打开项目页面。
   *Open the project page.*
2. 跳转至坐标：Lng 22.7, Lat 77.8 (或缩放至极地风眼)。
   *Jump to coordinates: Lng 22.7, Lat 77.8 (or zoom to the polar eyewall).*
3. 观察风眼中心。
   *Observe the center of the eyewall.*

**技术分析 / Technical Analysis:**
- 底层 Zarr 数据源在此处网格边缘存在 NaN 或 0 缺测值。
  *The underlying Zarr data source has NaN or 0 missing values at this grid boundary.*
- 粒子渲染到该坐标时，因缺乏风场向量导致失去动力。
  *When particles render to this coordinate, they lose momentum due to the lack of wind field vectors.*
- 由于粒子滞留在原地，外加拖尾缓存，导致该经线/纬线上形成密集的直线。
  *Because particles stagnate in place, combined with the trail buffer, a dense straight line forms along this longitude/latitude.*

**已尝试方案及失败原因 / Attempted Solutions & Failures:**
- 调整 palette 将低风速颜色设为背景色 #0a0e14：无效，风眼轮廓也会消失，且线条依然可见。
  *Adjusting `palette` to set low wind speed color to background color `#0a0e14`: Ineffective, the eyewall structure disappears, and the line remains visible.*
- 增加 dropRate 并降低 fadeOpacity：有效减轻，但在最大缩放级别下仍然有一条可见的直线断带。
  *Increasing `dropRate` and decreasing `fadeOpacity`: Effectively mitigated, but a visible straight line fault band still exists at max zoom level.*

**期望结果 / Expected Result:**
需要一种能在底层数据缺测时，让粒子直接“瞬移/重生”而不原地堆积的渲染机制，或者提供数据插值平滑方案（当前 Zartigl 暂无此 API）。
*Need a rendering mechanism that allows particles to "teleport/respawn" instead of accumulating in place when underlying data is missing, or provide a data interpolation smoothing solution (currently unavailable in Zartigl's API).*
