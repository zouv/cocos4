# Cocos 4 项目架构详解

## 整体架构

Cocos 4 采用**分层模块化架构**，整体架构图如下：

```
┌─────────────────────────────────────────────────────────────┐
│                     应用层 (Application)                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   游戏逻辑   │  │  交互系统   │  │  业务代码   │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              ↓↑
┌─────────────────────────────────────────────────────────────┐
│                   TypeScript 层 (cocos/)                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │   2D/3D  │ │  渲染    │ │  物理    │ │  动画    │      │
│  │   模块   │ │  模块    │ │  模块    │ │  模块    │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │   UI     │ │  音频    │ │  XR      │ │  场景    │      │
│  │   模块   │ │  模块    │ │  模块    │ │  管理    │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓↑
┌─────────────────────────────────────────────────────────────┐
│                 平台抽象层 (PAL - Platform Abstraction Layer)│
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  系统信息 │ │  屏幕    │ │  输入     │ │  文件    │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  音频    │ │  网络    │ │  WASM    │ │  小游戏  │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓↑
┌─────────────────────────────────────────────────────────────┐
│                   C++ Native 层 (native/)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  渲染器  │ │  数学库  │ │  资源    │ │  场景    │      │
│  │          │ │          │ │  管理    │ │  管理    │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  物理    │ │  音频    │ │  网络    │ │  平台    │      │
│  │  引擎    │ │  引擎    │ │  模块    │ │  适配    │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓↑
┌─────────────────────────────────────────────────────────────┐
│                       外部依赖 (external/)                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  Box2D   │ │  Bullet  │ │  PhysX   │ │  Spine   │      │
│  │ (2D物理) │ │ (3D物理) │ │ (高级物理)│ │ (骨骼动画)│      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 双语言架构原理

### 设计理念

Cocos 4 采用**双语言架构**，将性能敏感的逻辑用 C++ 实现，业务逻辑用 TypeScript 实现，通过 JSBinding 实现双向通信。

```
┌──────────────────┐         ┌──────────────────┐
│   TypeScript     │ ←─────→ │   C++ Native     │
│   (业务逻辑层)     │         │   (性能层)        │
├──────────────────┤         ├──────────────────┤
│ • 游戏逻辑        │         │ • 渲染管线        │
│ • 场景图管理      │         │ • 物理计算        │
│ • 资源加载        │         │ • 数学运算        │
│ • 动画控制        │         │ • 内存管理        │
│ • UI 布局        │         │ • 平台适配        │
│ • 事件系统        │         │ • 音频处理        │
└──────────────────┘         └──────────────────┘
           ↓↑                       ↓↑
           └──────── JS Binding ────────┘
```

### 数据流向

```
用户交互
    ↓
TypeScript (输入处理、业务逻辑)
    ↓
JSBinding (调用 C++ 功能)
    ↓
C++ Native (高性能计算)
    ↓
GPU/硬件 (渲染、物理计算)
    ↓
C++ Native (结果回调)
    ↓
JSBinding (结果返回)
    ↓
TypeScript (更新游戏状态)
    ↓
渲染 (绘制画面)
```

## 模块依赖关系

### 核心依赖图

```
                    ┌──────────────┐
                    │   cocos/core  │
                    │   (核心基础)   │
                    └───────┬──────┘
                            │
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ cocos/game   │   │ cocos/gfx   │   │cocos/scene-graph│
│   (游戏)      │   │   (渲染)     │   │   (场景图)    │
└───────┬──────┘   └──────┬───────┘   └───────┬──────┘
        │                 │                   │
        └────────┬────────┴────────┬───────────┘
                 ↓                 ↓
        ┌────────────────┐ ┌────────────────┐
        │  cocos/2d     │ │   cocos/3d     │
        │  (2D 模块)     │ │  (3D 模块)      │
        └────────────────┘ └────────────────┘
                 ↓                 ↓
        ┌────────────────┐ ┌────────────────┐
        │  UI/粒子/动画  │ │ 光照/模型/地形  │
        └────────────────┘ └────────────────┘
```

### 模块分层

```
第一层: 核心基础层
├── cocos/core              # 核心类、系统、工具
├── cocos/asset             # 资源管理
└── cocos/serialization     # 序列化

第二层: 功能模块层
├── cocos/2d                # 2D 功能
├── cocos/3d                # 3D 功能
├── cocos/animation         # 动画系统
├── cocos/physics           # 物理引擎
├── cocos/rendering         # 渲染管线
└── cocos/tween             # 补间动画

第三层: 组件层
├── cocos/ui                # UI 组件
├── cocos/video             # 视频组件
├── cocos/web-view          # WebView 组件
├── cocos/particle          # 粒子系统
└── cocos/spine             # Spine 动画

第四层: 平台适配层
├── pal/*                   # 平台抽象层
└── native/*                # 原生平台实现
```

## 目录结构详解

### 根目录结构

```
cocos4/
│
├── cocos/                  # TypeScript 核心引擎 ⭐
│   ├── 2d/                # 2D 渲染组件
│   ├── 3d/                # 3D 渲染组件
│   ├── animation/         # 动画系统
│   ├── audio/             # 音频系统
│   ├── core/              # 核心功能
│   ├── game/              # 游戏主循环
│   ├── gfx/               # 图形抽象层
│   ├── physics/           # 物理引擎
│   ├── rendering/         # 渲染管线
│   ├── scene-graph/       # 场景图管理
│   ├── tween/             # 补间动画
│   ├── ui/                # UI 系统
│   └── ...
│
├── native/                 # C++ 原生实现 ⭐
│   ├── cocos/             # C++ 引擎核心
│   │   ├── base/          # 基础工具
│   │   ├── math/          # 数学库
│   │   ├── renderer/      # 渲染器
│   │   ├── scene/         # 场景管理
│   │   └── platform/      # 平台适配
│   ├── CMakeLists.txt     # CMake 构建配置
│   └── ...
│
├── pal/                    # 平台抽象层 ⭐
│   ├── audio/             # 音频抽象
│   ├── input/             # 输入抽象
│   ├── screen-adapter/    # 屏幕适配
│   ├── system-info/       # 系统信息
│   └── wasm/              # WASM 支持
│
├── exports/                # 模块导出配置
│   ├── base.ts            # 基础模块导出
│   ├── 2d.ts              # 2D 模块导出
│   ├── 3d.ts              # 3D 模块导出
│   └── ...
│
├── native/external/        # 原生外部依赖
│   ├── box2d/             # 2D 物理引擎
│   ├── bullet/            # 3D 物理引擎
│   ├── physx/             # NVIDIA PhysX
│   └── spine/             # Spine 动画库
│
├── scripts/                 # 构建脚本
│   ├── build-*.js         # 各种构建脚本
│   └── native-pack-tool/  # 原生打包工具
│
├── editor/                  # 编辑器扩展
│   ├── inspector/         # 属性检查器
│   ├── dashboard/          # 启动面板
│   └── i18n/              # 国际化
│
├── templates/               # 项目模板
│   ├── android/           # Android 模板
│   ├── ios/               # iOS 模板
│   ├── mac/               # macOS 模板
│   └── windows/           # Windows 模板
│
├── tests/                   # 单元测试
│   ├── core/              # 核心模块测试
│   ├── physics/           # 物理模块测试
│   └── ui/                # UI 模块测试
│
├── @types/                 # TypeScript 类型定义
│   ├── pal/               # PAL 类型
│   ├── jsb.d.ts           # JS Binding 类型
│   └── globals.d.ts       # 全局类型
│
└── docs/                   # 文档
    ├── CPP_CODING_STYLE.md # C++ 编码规范
    └── TS_CODING_STYLE.md # TS 编码规范
```

## 核心模块介绍

### 1. cocos/core - 核心基础

**功能职责**: 提供引擎的核心基础功能，包括类系统、事件系统、数学库等。

**主要子模块**:
- `math/` - 数学库（Vec2, Vec3, Vec4, Mat4, Quat, Color, Rect, Size）
- `data/` - 数据类系统、属性装饰器
- `event/` - 事件系统
- `geometry/` - 几何计算（AABB, Ray, Plane, Sphere）
- `memop/` - 内存管理（Pool, Memop）
- `platform/` - 平台相关功能
- `utils/` - 通用工具函数

**核心类**:
```typescript
// 节点和组件系统
class Node
class Component

// 数学类型
class Vec3
class Mat4
class Quat
class Color

// 事件系统
class EventTarget
function Eventify<T>(obj: T): T
```

### 2. cocos/game - 游戏主循环

**功能职责**: 管理游戏的主循环、场景切换、帧率控制等。

**核心功能**:
```typescript
// 游戏主循环管理
class Game {
  // 启动游戏
  run(): void

  // 场景管理
  loadScene(sceneName: string): void

  // 帧率控制
  setFrameRate(fps: number): void
  getFrameRate(): number

  // 事件
  on(constructor: Function, callback: Function): void
}
```

### 3. cocos/gfx - 图形抽象层

**功能职责**: 提供统一的图形 API 抽象，支持多种图形后端。

**后端支持**:
- WebGL
- WebGL2
- WebGPU
- Metal (Native)
- Vulkan (Native)
- Empty (Headless)

**核心接口**:
```typescript
interface Device {
  createBuffer(info: BufferInfo): Buffer
  createTexture(info: TextureInfo): Texture
  createShader(info: ShaderInfo): Shader
  createPipelineState(info: PipelineStateInfo): PipelineState
}

interface RenderPipeline {
  render(passes: Pass[]): void
}
```

### 4. cocos/2d - 2D 渲染

**功能职责**: 提供 2D 游戏开发所需的组件和系统。

**核心组件**:
```typescript
// 渲染组件
class Sprite           // 精灵
class Label            // 文本标签
class Canvas           // 画布
class Graphics         // 图形绘制
class Mask             // 遮罩
class RichText         // 富文本

// 渲染系统
class Batcher2D       // 2D 批处理器
class UIMeshBuffer     // UI 网格缓冲
```

### 5. cocos/3d - 3D 渲染

**功能职责**: 提供 3D 游戏开发所需的组件和系统。

**核心组件**:
```typescript
// 3D 对象
class MeshRenderer     // 网格渲染器
class SkinnedMeshRenderer // 蒙皮网格渲染
class Terrain         // 地形

// 光照
class DirectionalLight  // 方向光
class PointLight        // 点光源
class SpotLight         // 聚光灯

// 相机
class Camera
```

### 6. cocos/physics - 物理引擎

**功能职责**: 集成物理引擎，支持 2D/3D 物理模拟。

**支持的物理引擎**:
- **2D**: Box2D
- **3D**: Bullet, Cannon, PhysX, Ammo

**核心接口**:
```typescript
// 物理世界
class PhysicsWorld {
  // 射线检测
  raycast(ray: Ray, mask: number): PhysicsRaycastResult[]

  // 碰撞检测
  checkCollision(group1: number, group2: number): void
}

// 刚体
interface IRigidBody {
  applyForce(force: Vec3): void
  applyImpulse(impulse: Vec3): void
  setLinearVelocity(velocity: Vec3): void
}
```

### 7. cocos/animation - 动画系统

**功能职责**: 提供骨骼动画和关键帧动画系统。

**核心类**:
```typescript
// 骨骼动画
class AnimationComponent
class SkeletalAnimation

// 关键帧动画
class AnimationClip
class AnimationCurve

// 动画状态机
class Animator
class AnimatorController
```

### 8. cocos/ui - UI 系统

**功能职责**: 提供完整的 UI 组件库。

**核心组件**:
```typescript
class Button           // 按钮
class Label            // 文本
class Layout           // 布局容器
class ScrollView       // 滚动视图
class Slider           // 滑动条
class Toggle           // 开关
class ProgressBar      // 进度条
class EditBox          // 输入框
class Widget           // 对齐组件
```

### 9. cocos/rendering - 渲染管线

**功能职责**: 管理渲染管线的配置和执行。

**核心概念**:
```typescript
// 渲染管线
class RenderPipeline {
  addRenderPass(pass: RenderPass): void
  removeRenderPass(index: number): void
}

// 自定义管线
class CustomPipeline extends RenderPipeline {
  // 自定义渲染逻辑
}
```

## 平台抽象层 (PAL)

PAL (Platform Abstraction Layer) 设计用于分离平台相关代码，提供统一的接口。

```
┌─────────────────────────────────────────┐
│         Cocos Engine (业务逻辑)          │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│     PAL (Platform Abstraction Layer)    │
├─────────┬──────────┬──────────┬─────────┤
│   Web   │ Native   │ Minigame │ Node.js │
└─────────┴──────────┴──────────┴─────────┘
```

### PAL 模块

| 模块 | 功能 | 平台实现 |
|------|------|---------|
| `pal/audio` | 音频播放 | Web/Native/Minigame/Node.js |
| `pal/input` | 输入处理 | Web/Native/Minigame |
| `pal/screen-adapter` | 屏幕适配 | Web/Native/Minigame |
| `pal/system-info` | 系统信息 | Web/Native/Minigame |
| `pal/wasm` | WASM 支持 | Web/Native/Minigame |
| `pal/pacer` | 帧率控制 | 各平台统一 |

## 构建产物

### 输出目录结构

```
bin/
├── .declarations/      # 类型声明文件
│   ├── cc.d.ts
│   └── cc.editor.d.ts
│
├── adapter/            # 平台适配器
│   ├── native/         # 原生平台
│   ├── web/            # Web 平台
│   ├── minigame/       # 小游戏平台
│   │   ├── wechat/     # 微信小游戏
│   │   ├── alipay/     # 支付宝小游戏
│   │   └── ...
│   └── nodejs/         # Node.js 平台
│
└── dev/                # 开发构建产物
    └── cc-min/         # 最小化构建
        ├── base.js
        ├── 2d.js
        ├── 3d.js
        ├── physics-*.js
        └── assets/     # 资源文件
```

## 扩展阅读

- [模块详解](MODULES.md) - 了解各模块的详细功能
- [构建系统](BUILD_SYSTEM.md) - 了解构建流程和配置
