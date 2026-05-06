# Cocos 4 构建系统

本文档详细介绍 Cocos 4 引擎的构建系统，包括构建流程、配置选项和自定义构建方法。

## 目录

1. [构建概述](#构建概述)
2. [构建命令](#构建命令)
3. [构建流程详解](#构建流程详解)
4. [构建配置](#构建配置)
5. [输出产物](#输出产物)
6. [自定义构建](#自定义构建)
7. [常见问题](#常见问题)

---

## 构建概述

Cocos 4 使用现代化的构建系统，基于以下技术栈：

- **Node.js**: 运行时环境
- **npm**: 包管理和脚本执行
- **Rollup**: JavaScript 模块打包
- **TypeScript**: 类型检查和编译
- **Babel**: JavaScript 代码转换
- **CMake**: C++ 原生代码构建

### 构建架构

```
┌─────────────────────────────────────────┐
│          npm scripts (入口)              │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│     @cocos/ccbuild (统一构建工具)         │
├─────────────┬─────────────┬──────────────┤
│   Rollup    │  TypeScript │   Babel     │
│  (打包)     │  (类型检查)  │  (转译)     │
└─────────────┴─────────────┴──────────────┘
                   ↓
        ┌─────────────────────┐
        │    构建产物输出      │
        │  bin/dev/cc-min/   │
        └─────────────────────┘
```

---

## 构建命令

### 常用命令

#### 完整构建

```bash
# 安装依赖（首次使用）
npm install

# 完整构建（生产级别）
npm run build
```

这个命令会执行：
1. `npm run build:min` - 构建最小化引擎
2. `npm run build:declaration` - 生成 TypeScript 声明文件

#### 开发构建

```bash
# 开发模式构建（更快）
npm run build:dev
```

#### 仅构建适配器

```bash
# 构建平台适配器
npm run build:adapter
```

#### 构建原生工具

```bash
# 构建原生打包工具
npm run build:native-pack-tool
```

#### 清理构建缓存

```bash
# 清理所有缓存
npm run clear

# 清理平台构建
npm run clear:platform
```

### 构建命令列表

| 命令 | 功能 | 输出 |
|------|------|------|
| `npm run build` | 完整构建 | 引擎 + 声明文件 |
| `npm run build:dev` | 开发构建 | H5 源码构建 |
| `npm run build:min` | 最小化构建 | 压缩的 H5 构建 |
| `npm run build:cli` | CLI 构建 | 命令行工具构建 |
| `npm run build:adapter` | 适配器构建 | 平台适配器 |
| `npm run build:native-pack-tool` | 原生打包工具 | 原生打包可执行文件 |
| `npm run build:debug-infos` | 调试信息构建 | DebugInfos.json |
| `npm run build:declaration` | 声明文件生成 | .d.ts 文件 |

---

## 构建流程详解

### 完整构建流程

```
npm run build
    ↓
[1] npm run build:min
    ↓
    ├─ npm run build:debug-infos
    │   └─→ 生成 DebugInfos.json
    │
    └─ node ./scripts/build-h5-minified
        ├─→ Rollup 打包引擎代码
        ├─→ Babel 转译
        ├─→ TypeScript 类型检查
        └─→ 输出到 bin/dev/cc-min/
    ↓
[2] npm run build:declaration
    ↓
    ├─ npm run build:const
    │   └─→ 生成 @types/consts.d.ts
    │
    └─ node ./scripts/build-declarations
        ├─→ TypeScript 声明文件打包
        └─→ 输出到 bin/.declarations/
```

### 构建脚本详解

#### build:debug-infos

**位置**: `scripts/build-debug-infos.js`

**功能**: 生成调试信息文件 `DebugInfos.json`，包含所有引擎内置错误代码和警告信息。

```javascript
// 简化流程
async function buildDebugInfos() {
    // 1. 扫描源代码中的调试信息宏
    // 2. 生成映射表
    // 3. 输出到 DebugInfos.json
}
```

#### build-h5-minified

**位置**: `scripts/build-h5-minified.js`

**功能**: 构建最小化的 H5 引擎包。

```javascript
// 核心配置
{
    engine: 'path/to/engine',
    moduleFormat: 'system',      // 模块格式
    mode: 'BUILD',              // 构建模式
    platform: 'HTML5',          // 目标平台
    out: 'bin/dev/cc-min',      // 输出目录
    compress: true,              // 压缩
    sourceMap: true,             // 源码映射
    visualize: true              // 可视化分析
}
```

#### build-declarations

**位置**: `scripts/build-declarations.js`

**功能**: 生成 TypeScript 声明文件。

```javascript
// 核心配置
{
    engine: 'path/to/engine',
    outDir: 'bin/.declarations',
    withIndex: true,            // 生成 index.d.ts
    withExports: false,         // 包含 exports
    withEditorExports: true     // 包含编辑器导出
}
```

---

## 构建配置

### 项目配置文件

#### package.json

**位置**: 根目录 `package.json`

```json
{
  "name": "cocos-creator",
  "version": "4.0.0-alpha.15",
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "build": "npm run build:min && npm run build:declaration",
    "build:dev": "npm run build:debug-infos && node ./scripts/build-h5-source",
    "build:min": "npm run build:debug-infos && node ./scripts/build-h5-minified"
  }
}
```

#### cc.config.json

**位置**: 根目录 `cc.config.json`

功能：定义引擎模块配置、特性开关、资源 UUID 等。

```json
{
  "features": {
    "base": {
      "modules": ["base"],
      "dependentAssets": ["uuid1", "uuid2"]
    },
    "gfx-webgl": {
      "modules": ["gfx-webgl"]
    },
    "3d": {
      "modules": ["3d"],
      "overrideConstants": {
        "USE_3D": true
      }
    },
    "2d": {
      "modules": ["2d", "sorting"]
    },
    "physics": {
      "modules": ["physics"]
    }
  }
}
```

#### tsconfig.json

**位置**: 根目录 `tsconfig.json`

TypeScript 编译配置。

```json
{
  "compilerOptions": {
    "target": "es6",
    "lib": ["es2015", "es2017", "dom"],
    "module": "commonjs",
    "strict": true,
    "noImplicitAny": false,
    "experimentalDecorators": true,
    "baseUrl": "./",
    "paths": {
      "cc.decorator": ["cocos/core/data/decorators/index.ts"]
    }
  }
}
```

### 引擎常量配置

#### predefine.ts

**位置**: 根目录 `predefine.ts`

定义全局宏和编译常量。

```typescript
// 构建模式
export const BUILD = true;
export const DEBUG = true;
export const DEV = false;

// 平台标识
export const HTML5 = true;
export const NATIVE = false;
export const WECHAT = false;

// 功能开关
export const USE_3D = true;
export const USE_UI_SKEW = true;
export const SUPPORT_JIT = true;

// JSB 相关
export const JSB = false;
```

---

## 输出产物

### 输出目录结构

```
bin/
│
├── .declarations/              # TypeScript 声明文件
│   ├── cc.d.ts                # 引擎主声明
│   └── cc.editor.d.ts         # 编辑器声明
│
├── adapter/                    # 平台适配器
│   │
│   ├── native/                # 原生平台适配
│   │   ├── engine-adapter.js
│   │   └── web-adapter.js
│   │
│   ├── web/                   # Web 平台适配
│   │
│   ├── nodejs/                # Node.js 适配
│   │
│   ├── minigame/              # 小游戏平台适配
│   │   ├── alipay/
│   │   ├── bytedance/
│   │   ├── taobao/
│   │   ├── wechat/
│   │   └── xiaomi/
│   │
│   └── runtime/               # 运行平台适配
│       ├── honor-mini-game/
│       ├── huawei-quick-game/
│       └── ...
│
└── dev/                       # 开发构建产物
    │
    └── cc-min/                # 最小化构建
        │
        ├── index.js           # 入口文件
        ├── base.js            # 基础模块
        ├── 2d.js              # 2D 模块
        ├── 3d.js              # 3D 模块
        ├── animation.js       # 动画模块
        ├── physics-*.js        # 物理模块
        ├── gfx-*.js           # 图形模块
        │
        ├── assets/            # 资源文件
        │   ├── *.wasm         # WebAssembly 模块
        │   ├── *.js.mem       # 内存文件
        │   └── *.bin          # 二进制资源
        │
        └── *.js.map           # 源码映射文件
```

### 构建产物说明

#### JavaScript 模块

每个 JavaScript 文件都是独立的 Rollup bundle，包含：
- 压缩后的引擎代码
- 内联资源（着色器、配置等）
- 源码映射（开发构建）

#### WebAssembly 模块

位于 `assets/` 目录：

| 文件 | 功能 |
|------|------|
| `box2d.release.asm.js` | Box2D ASM.js 版本 |
| `box2d.release.wasm.js` | Box2D WebAssembly 版本 |
| `bullet.release.asm.js` | Bullet ASM.js 版本 |
| `bullet.release.wasm.js` | Bullet WebAssembly 版本 |
| `physx.release.asm.js` | PhysX ASM.js 版本 |
| `physx.release.wasm.js` | PhysX WebAssembly 版本 |
| `spine.asm.js` | Spine ASM.js 版本 |
| `spine.wasm.js` | Spine WebAssembly 版本 |
| `glslang.wasm` | GLSL 编译器 WebGPU 版本 |
| `twgsl.wasm` | WGSL 转换器 WebGPU 版本 |
| `meshopt_decoder.wasm` | 网格优化解码器 |

---

## 自定义构建

### 按需加载模块

Cocos 4 支持按需构建，只包含你需要的模块。

#### 1. 创建自定义构建配置

```javascript
// custom-build.js
const { buildEngine } = require('@cocos/ccbuild');

async function buildCustomEngine() {
  await buildEngine({
    engine: './',
    moduleFormat: 'system',
    mode: 'BUILD',
    platform: 'HTML5',
    out: './bin/custom',
    features: [
      'base',
      'gfx-webgl2',
      '2d',
      'animation',
      'ui'
      // 添加你需要的模块
    ],
    compress: true,
    sourceMap: true
  });
}

buildCustomEngine();
```

#### 2. 运行自定义构建

```bash
node custom-build.js
```

### 修改构建常量

#### 全局宏

编辑 `predefine.ts`：

```typescript
// 开启/关闭功能
export const USE_3D = true;           // 启用 3D 功能
export const SUPPORT_JIT = true;       // 启用 JIT 编译
export const USE_WEBGPU = false;       // 禁用 WebGPU

// 物理引擎
export const LOAD_BULLET_MANUALLY = false;  // 自动加载 Bullet
```

#### 模块常量

在 `cc.config.json` 中设置：

```json
{
  "features": {
    "3d": {
      "overrideConstants": {
        "USE_3D": true,
        "USE_PBR_GI": true
      }
    }
  }
}
```

### 创建模块变体

#### 平台特定适配

在 `pal/` 目录创建新的平台适配：

```typescript
// pal/my-platform/my-feature.ts
export const myPlatformAdaptor = {
  // 平台特定实现
};
```

#### 注册到构建系统

```javascript
// 在 build 配置中指定平台
{
  platform: 'my-platform'
}
```

---

## 常见问题

### 1. 构建失败：找不到外部依赖

**问题**: 编译时报错找不到原生外部库。

**解决方案**:
```bash
# 更新原生外部依赖
npm run update:native-external
```

### 2. 路径过长错误 (Windows)

**问题**: Windows 系统路径超过 260 字符限制。

**解决方案**:

方法一：启用 Windows 长路径
```bash
git config --global core.longpaths true
```

方法二：移动项目到更短的路径
```bash
# 例如移动到 C:\cocos4\
```

### 3. TypeScript 类型错误

**问题**: 构建时出现 TypeScript 类型检查错误。

**解决方案**:

检查 `tsconfig.json` 配置：
```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "noImplicitAny": false
  }
}
```

### 4. 内存不足

**问题**: 大型构建导致内存不足。

**解决方案**:

调整 Node.js 内存限制：
```bash
node --max-old-space-size=4096 node_modules/.bin/rollup ...
```

或设置环境变量：
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
```

### 5. 缓存问题

**问题**: 修改代码后构建结果未更新。

**解决方案**:

清理缓存后重新构建：
```bash
npm run clear
npm run build
```

---

## 进阶用法

### 程序化构建

```javascript
// programmatic-build.js
const { buildEngine, buildModule } = require('@cocos/ccbuild');

// 构建不同配置的引擎
async function buildVariants() {
  // WebGL2 版本
  await buildEngine({
    engine: './',
    out: './bin/webgl2',
    features: ['base', 'gfx-webgl2', '2d', '3d']
  });

  // WebGPU 版本
  await buildEngine({
    engine: './',
    out: './bin/webgpu',
    features: ['base', 'gfx-webgpu', '2d', '3d']
  });
}

buildVariants();
```

### 多平台构建

```javascript
// multi-platform-build.js
const platforms = ['HTML5', 'Android', 'iOS', 'Windows', 'macOS'];

for (const platform of platforms) {
  await buildEngine({
    engine: './',
    platform: platform,
    out: `./bin/${platform.toLowerCase()}`
  });
}
```

### 增量构建

```javascript
// incremental-build.js
const { createWatcher } = require('@cocos/ccbuild');

// 监视文件变化，增量构建
const watcher = createWatcher({
  engine: './',
  out: './bin/dev',
  watch: ['cocos/**/*.ts', 'exports/**/*.ts']
});

watcher.start();
watcher.on('change', (changedFiles) => {
  console.log('Changed:', changedFiles);
  // 仅重新构建受影响的模块
});
```

---

## 相关文档

- [架构详解](ARCHITECTURE.md)
- [模块详解](MODULES.md)
- [@cocos/ccbuild 文档](https://www.npmjs.com/package/@cocos/ccbuild)
