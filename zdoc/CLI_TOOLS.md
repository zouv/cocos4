# Cocos 4 CLI 工具详解

本文档详细介绍 Cocos 4 引擎的命令行工具（CLI），包括构建命令、原生打包工具、平台适配等功能。

## 目录

1. [构建命令](#构建命令)
2. [原生平台打包工具](#原生平台打包工具)
3. [平台适配器](#平台适配器)
4. [项目模板](#项目模板)
5. [平台抽象层 (PAL)](#平台抽象层-pal)
6. [典型工作流程](#典型工作流程)
7. [CLI 工具架构](#cli-工具架构)

---

## 构建命令

### 完整构建

```bash
npm run build
```

**功能**: 执行完整的产品级构建

**执行流程**:
```
npm run build
    ↓
├─ npm run build:min          # 最小化引擎构建
└─ npm run build:declaration  # TypeScript 声明文件生成
```

**输出目录**:
- `bin/dev/cc-min/` - 压缩后的引擎代码
- `bin/.declarations/` - TypeScript 类型定义文件

---

### 开发构建

```bash
npm run build:dev
```

**功能**: 快速构建用于开发的 H5 源码版本

**特点**:
- 构建速度更快
- 包含完整的源码映射（sourceMap）
- 便于调试和开发

---

### 最小化构建

```bash
npm run build:min
```

**功能**: 生成最小化的 H5 游戏包

**核心配置**:
```javascript
{
    engine: './',
    moduleFormat: 'system',      // 模块格式
    mode: 'BUILD',              // 构建模式
    platform: 'HTML5',          // 目标平台
    out: 'bin/dev/cc-min',      // 输出目录
    compress: true,              // 启用压缩
    sourceMap: true,             // 生成源码映射
    visualize: true              // 可视化分析支持
}
```

---

### CLI 版本构建

```bash
npm run build:cli
```

**功能**: 构建 Node.js 平台的 CLI 工具版本

**执行流程**:
```
npm run build:cli
    ↓
├─ npm run build:cli-min      # CLI 最小化构建
└─ npm run build:declaration  # TypeScript 声明文件生成
```

**目标平台**: `NODEJS`

**输出目录**: `bin/dev/cc-cli-min/`

**用途**:
- 服务端游戏逻辑运行
- 无头（Headless）服务器
- 命令行批处理工具
- CI/CD 自动化构建

---

### 适配器构建

```bash
npm run build:adapter
```

**功能**: 为不同平台构建适配器代码

**输出目录**: `bin/adapter/`

---

### 原生打包工具构建

```bash
npm run build:native-pack-tool
```

**功能**: 构建原生平台打包工具

**位置**: `scripts/native-pack-tool/`

---

## 原生平台打包工具

### 工具概述

原生打包工具用于将 Cocos Creator 构建生成的资源项目文件夹整合成标准的原生平台工程。

### 目录结构要求

```
build/
└── windows/
    ├── data/                          # 原始构建工程资源
    └── cocos.compile.config.json      # 原生工程配置信息
```

### 支持的命令

```bash
# 创建原生模板
npm run pack [projectBuildPath] create

# 生成 CMake 工程
npm run pack [projectBuildPath] generate

# 编译原生工程
npm run pack [projectBuildPath] make

# 运行编译后的应用
npm run pack [projectBuildPath] run

# 组合命令
npm run pack [projectBuildPath] create,generate,make,run
```

### 支持平台

| 平台 | 目标工程 | 说明 |
|------|---------|------|
| Windows | Visual Studio | `.sln` 解决方案 |
| macOS | Xcode | `.xcodeproj` 项目 |
| iOS | Xcode | `.xcodeproj` 项目 |
| Android | Gradle | Android Studio 项目 |
| Linux | CMake | Makefile 工程 |
| HarmonyOS | CMake | OpenHarmony 工程 |

### 新平台注册流程

1. **编写平台打包类**：参考现有平台实现，继承默认类或已有平台类

2. **注册平台**：在 `source/index.ts` 中注册新平台

```typescript
import { MyPlatformPackTool } from './platforms/my-platform';
nativePackToolMg.register('my-platform', new MyPlatformPackTool());
```

3. **实现核心方法**：确保 `create`、`generate`、`make`、`run` 逻辑正常

4. **测试验证**：通过命令行测试新平台功能

---

## 平台适配器

### 适配器概述

平台适配器为不同运行环境提供统一的 API 适配层，使得引擎代码能够在各种平台上运行。

### 输出结构

```
bin/adapter/
├── native/           # 原生平台适配器
├── web/              # Web 平台适配器
├── nodejs/           # Node.js 平台适配器
├── minigame/         # 小游戏平台适配器
│   ├── alipay/       # 支付宝小游戏
│   ├── bytedance/    # 抖音小游戏
│   ├── taobao/       # 淘宝小程序
│   ├── wechat/       # 微信小游戏
│   └── xiaomi/       # 小米游戏
└── runtime/          # 运行平台适配器
    ├── honor-mini-game/
    ├── huawei-quick-game/
    ├── migu-mini-game/
    ├── oppo-mini-game/
    └── vivo-mini-game/
```

### 适配器内容

每个平台适配器包含：
- `engine-adapter.js` - 引擎层适配
- `engine-adapter.min.js` - 压缩版本
- `web-adapter.js` - Web API 适配
- `web-adapter.min.js` - 压缩版本

---

## 项目模板

### 模板概述

项目模板提供各平台的基础项目结构，用于快速生成原生工程。

### 模板目录

```
templates/
├── ios/          # iOS Xcode 项目模板
├── mac/          # macOS Xcode 项目模板
├── windows/      # Windows Visual Studio 模板
├── android/     # Android Gradle 模板
├── ohos/        # HarmonyOS 模板
├── linux/       # Linux CMake 模板
├── qnx/         # QNX 平台模板
└── launcher/    # 启动器资源
```

### 模板内容

每个平台模板包含：
- **项目配置文件**：`.xcodeproj`、`.sln`、`CMakeLists.txt` 等
- **构建脚本**：构建配置和脚本
- **资源文件**：图标、配置文件等
- **入口代码**：`main.cpp`、`AppDelegate` 等

---

## 平台抽象层 (PAL)

### PAL 概述

PAL (Platform Abstraction Layer) 提供平台无关的接口定义，根据不同平台提供具体实现。

### PAL 模块

#### pal/audio - 音频抽象

```typescript
interface AudioPlayer {
    play(): void
    pause(): void
    stop(): void
    volume: number
    loop: boolean
    readonly state: AudioState
}
```

#### pal/system-info - 系统信息

```typescript
interface SystemInfo {
    readonly platform: Platform
    readonly os: OS
    readonly screenSize: Size
    supportWebGL: boolean
    supportWebGPU: boolean
    supportWasm: boolean
}
```

#### pal/input - 输入系统

```typescript
interface Input {
    setTouchCallback(callback: Function): void
    setMouseCallback(callback: Function): void
    setKeyboardCallback(callback: Function): void
    setGamepadCallback(callback: Function): void
}
```

#### pal/wasm - WASM 支持

```typescript
interface WASM {
    loadWasm(url: string): Promise<WebAssembly.Instance>
    loadWasmModule(moduleName: string, url: string): Promise<Function>
}
```

### PAL 实现结构

```
pal/
├── audio/        # 音频实现
│   ├── web/      # Web 浏览器实现
│   ├── native/   # 原生平台实现
│   ├── minigame/ # 小游戏平台实现
│   └── nodejs/   # Node.js 实现
├── input/       # 输入实现
├── env/         # 环境信息
├── pacer/       # 帧率控制
└── wasm/        # WASM 支持
```

---

## 典型工作流程

### H5 游戏开发流程

```bash
# 1. 安装依赖
npm install

# 2. 开发构建
npm run build:dev

# 3. 开发调试（使用浏览器或开发服务器）

# 4. 生产构建
npm run build

# 5. 部署到 Web 服务器
```

### 原生平台发布流程

```bash
# 1. 构建原生打包工具
npm run build:native-pack-tool

# 2. 创建原生模板
npm run pack build/windows create

# 3. 生成平台工程
npm run pack build/windows generate

# 4. 编译项目
npm run pack build/windows make

# 5. 运行测试
npm run pack build/windows run
```

### 小游戏发布流程

```bash
# 1. 构建平台适配器
npm run build:adapter

# 2. 在 Cocos Creator 中选择目标平台

# 3. 导出项目

# 4. 使用平台开发者工具预览和发布
```

---

## CLI 工具架构

### 架构图

```
┌─────────────────────────────────────────────────┐
│           npm scripts (命令入口)                  │
├─────────────────────────────────────────────────┤
│  build         → 完整构建                        │
│  build:dev     → 开发构建                        │
│  build:min     → 最小化构建                      │
│  build:cli     → Node.js CLI 构建                │
│  build:adapter → 平台适配器构建                   │
│  build:native-pack-tool → 原生打包工具构建        │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│         @cocos/ccbuild (统一构建引擎)             │
├─────────────────────────────────────────────────┤
│  • Rollup         → 模块打包                      │
│  • TypeScript     → 类型检查                      │
│  • Babel          → 代码转译                      │
│  • Terser         → 代码压缩                      │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│              输出产物 (bin/)                      │
├─────────────────────────────────────────────────┤
│  • cc-min/          → H5 游戏包                   │
│  • cc-cli-min/      → Node.js CLI 版本            │
│  • adapter/         → 平台适配器                  │
│  • .declarations/   → TypeScript 类型定义          │
└─────────────────────────────────────────────────┘
```

### 构建流程详解

1. **初始化阶段**：读取配置文件，确定构建参数
2. **模块图生成**：分析模块依赖关系
3. **代码转译**：TypeScript → JavaScript
4. **代码压缩**：Terser 压缩优化
5. **资源打包**：WASM、纹理等资源处理
6. **产物输出**：生成最终构建产物

### 配置文件

#### package.json - 脚本定义

```json
{
  "scripts": {
    "build": "npm run build:min && npm run build:declaration",
    "build:dev": "npm run build:debug-infos && node ./scripts/build-h5-source",
    "build:min": "npm run build:debug-infos && node ./scripts/build-h5-minified",
    "build:cli": "npm run build:cli-min && npm run build:declaration",
    "build:adapter": "node ./scripts/build-adapter.js",
    "build:native-pack-tool": "cd ./scripts/native-pack-tool && npm install && npm run build"
  }
}
```

#### cc.config.json - 特性配置

```json
{
  "features": {
    "base": { "modules": ["base"] },
    "gfx-webgl2": { "modules": ["gfx-webgl2"] },
    "3d": { 
      "modules": ["3d"],
      "overrideConstants": { "USE_3D": true }
    },
    "physics-physx": { "modules": ["physics-physx"] }
  }
}
```

---

## CLI 工具总结

### 核心能力

1. **多平台构建**：支持 H5、Node.js、原生平台等多种目标
2. **自动化打包**：原生平台一键打包和编译
3. **平台适配**：自动适配小游戏、原生、Web 等平台
4. **开发支持**：开发模式和生产模式分离

### 开发者价值

- **命令行驱动**：无需 IDE 即可完成引擎定制和构建
- **CI/CD 集成**：轻松集成到持续集成流程
- **自动化部署**：实现构建和部署自动化
- **跨平台开发**：一套代码多平台发布

### 扩展能力

- **新平台支持**：通过注册机制添加新平台
- **自定义构建**：灵活配置构建参数
- **插件系统**：支持 hooks 扩展打包逻辑

---

## 相关文档

- [项目架构](ARCHITECTURE.md) - 了解引擎整体架构
- [模块详解](MODULES.md) - 了解各功能模块
- [构建系统](BUILD_SYSTEM.md) - 了解构建系统详情
