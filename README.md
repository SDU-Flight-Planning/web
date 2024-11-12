# 地图标记和路径生成项目

此项目是一个基于 Vue 和 AMap（高德地图）的多层地图应用。它提供了标记创建、编辑、路径生成和多边形绘制的功能，支持在不同的地图模式（卫星地图、标准地图、建筑物地图）之间切换。路径生成使用扫掠线算法，动态生成穿越凸多边形的路径，并支持实时编辑。

## 项目结构

项目的主要结构如下：

```plaintext
├── src
│   ├── App.vue                   # 主应用组件
│   ├── main.ts                   # 入口文件
│   ├── components
│   │   ├── Sidebar.vue           # 侧边栏组件
│   │   └── InfoWindow.vue        # 弹出信息窗口组件
│   ├── views
│   │   ├── SatelliteMap.vue      # 卫星地图组件，提供路径和标记的动态编辑
│   │   ├── StandardMap.vue       # 标准地图组件
│   │   └── BuildingsMap.vue      # 3D 楼块地图组件，仅展示
│   └── assets                    # 静态资源
│
└── README.md                     # 项目说明文件
```

## 文件说明

### 1. `App.vue`

`App.vue` 是主应用组件，负责整合和管理所有地图视图和组件。它的主要职责包括：
- 维护标记列表和多边形路径的状态。
- 控制不同地图视图之间的切换。
- 处理添加、更新、删除标记的逻辑。
- 更新和计算多边形路径和生成路径。

#### 功能模块

- **地图视图切换**：使用 `currentMapComponent` 变量控制显示的地图视图。
- **标记管理**：包括标记的添加、删除、位置更新，并计算和维护多边形路径。
- **多边形路径计算**：使用凸包算法自动计算多边形路径，并实时更新多边形。

#### 关键函数

- `addMarker(marker)`: 添加标记到 `markers` 列表中，并更新多边形路径。
- `updateMarkerPosition(id, lat, lng)`: 更新指定标记位置。
- `removeMarker(id)`: 删除指定标记并更新路径。
- `clearAllMarkers()`: 清除所有标记和路径信息。
- `computeConvexHull(points)`: 凸包算法，用于多边形路径的计算。

---

### 2. `components/Sidebar.vue`

侧边栏组件，提供地图视图的切换和标记清除按钮。包括以下功能：

- **地图切换**：提供卫星图、标准图和 3D 楼块图的切换按钮。
- **清除标记**：一键清除所有地图标记、路径和多边形路径。

#### 事件
- `switchMap(mapType)`: 切换地图视图。
- `clearAllMarkers()`: 清除地图上的所有标记和路径。

---

### 3. `components/InfoWindow.vue`

信息窗口组件，显示标记的经纬度信息，并支持标记的删除和位置调整。

- **属性**：
  - `lat` 和 `lng`：当前标记的经纬度。
- **功能**：
  - `deleteMarker()`: 删除当前标记。
  - `startDrag()`: 支持拖动信息窗口。
  - `closeWindow()`: 关闭信息窗口。

此窗口还支持通过 `WASD` 键对标记位置进行微调，能够实时更新标记位置和多边形的连线。

---

### 4. `views/SatelliteMap.vue`

卫星地图视图，主要负责标记创建、路径生成、动态编辑等交互功能。提供了用户在地图上点击以生成标记，查看标记详细信息，并支持生成的路径的实时编辑。

#### 功能模块

- **标记管理**：
  - 用户点击地图添加标记。
  - 标记信息窗口显示其经纬度，并支持删除和微调功能。
- **多边形绘制**：自动根据标记生成凸多边形，并在各边显示距离信息。
- **路径生成**：使用扫掠线算法生成一条路径贯穿凸多边形，用于展示完整的遍历路径，生成的路径转折点为绿色，路径线为黄色。
- **路径编辑**：支持路径上转折点的实时调整，路径线会随之更新。

#### 关键函数

- `addMarkerToMap(markerData)`: 在地图上添加标记。
- `drawPolygon(path)`: 绘制多边形，路径少于三个标记时不显示。
- `displayDistances(path)`: 计算并显示多边形各边的距离。
- `drawPathLine(points)`: 使用扫掠线算法生成路径。
- `clearAllMarkers()`: 删除所有标记、距离标签、路径和多边形。

---

### 5. `views/StandardMap.vue`

标准地图视图，具备类似于 `SatelliteMap.vue` 的功能，显示所有标记和路径，支持标记的创建、查看和删除功能，但主要用于地图视图的替代显示。

#### 功能模块

- **标记管理**：点击地图添加标记，显示标记的经纬度。
- **多边形和路径显示**：在地图上绘制多边形路径，并显示路径转折点和多边形边界的距离。
- **编辑支持**：支持标记的动态编辑，路径和多边形会自动更新。

#### 关键函数

- `drawPolygon(path)`: 在地图上绘制多边形。
- `displayDistances(path)`: 显示多边形各边的距离。
- `drawPathLine(points)`: 生成并绘制路径线，转折点为绿色。

---

### 6. `views/BuildingsMap.vue`

3D 楼块地图视图，仅用于展示多边形和路径，支持地图的 3D 视角旋转和缩放。此视图不提供任何标记或路径的编辑功能。

#### 功能模块

- **3D 视图**：支持 `Q`、`E` 键旋转和 `W`、`S` 键调整视角。
- **展示多边形和路径**：在 3D 楼块图层上展示多边形及路径，无编辑功能。
- **静态展示**：仅展示路径和多边形，不提供标记的创建、删除或编辑。

#### 关键函数

- `drawPolygon(path)`: 绘制静态多边形。
- `drawPathLine(points)`: 展示路径线，不提供路径点的交互。

---

## 项目安装和启动

1. **克隆项目**
   ```bash
   git clone <repository-url>
   cd <project-directory>
   ```

2. **安装依赖**
   ```bash
   npm install
   ```

3. **启动项目**
   ```bash
   npm run dev
   ```

4. **构建项目**
   若要构建项目以用于生产环境：
   ```bash
   npm run build
   ```

## 功能演示

### 标记管理
- 用户可以在地图上点击以生成标记。
- 每个标记会弹出一个信息窗口，显示其纬度和经度。
- 标记可通过 `WASD` 键微调位置，实时更新与其他标记的连接和多边形距离。

### 多边形路径计算
- 由凸包算法生成的多边形路径，将标记围绕形成凸形区域。
- 多边形边缘的中点显示两标记点间的距离。
- 若删除标记，路径和距离会自动更新。

### 路径生成和动态编辑
- 使用扫掠线算法生成遍历整个多边形的路径，路径为黄色，转折点为绿色。
- 转折点可通过拖拽或 `WASD` 键微调，路径线会实时更新。

### 地图模式切换
- 提供三个地图视图：卫星地图、标准地图和 3D 楼块地图。
- 在 3D 楼块视图中路径和多边形为静态展示，支持视角的旋转和俯仰调整。

### 路径和标记的清除
- 一键清除所有标记和路径，同时清除多边形边缘的距离显示。

