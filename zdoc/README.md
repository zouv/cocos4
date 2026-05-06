# Cocos 4 项目文档

欢迎使用 Cocos 4 游戏引擎项目文档。本文档旨在帮助开发者全面了解 Cocos 4 的项目结构、架构设计和模块功能。

## 📚 文档目录

### 核心文档
- **[项目架构总览](ARCHITECTURE.md)** - 详细介绍 Cocos 4 的整体架构设计
- **[模块详解](MODULES.md)** - 各核心模块的功能说明
- **[构建系统](BUILD_SYSTEM.md)** - 构建流程和工具链说明

### 快速导航

#### 项目主体结构
```
cocos4/
├── cocos/              # TypeScript 核心引擎
├── native/             # C++ 原生实现
├── pal/                # 平台抽象层
├── exports/            # 模块导出配置
├── editor/             # 编辑器相关
├── scripts/            # 构建脚本
├── templates/           # 项目模板
├── tests/              # 单元测试
├── vendor/             # 第三方库
├── @types/             # TypeScript 类型定义
└── docs/               # 官方文档
```

## 🎯 Cocos 4 简介

COCOS 4 是 Cocos Creator 引擎的全新版本，采用**双语言架构**：
- **C++**: 保证运行时高性能
- **TypeScript**: 保证开发效率

### 核心特性
- ✅ 跨平台支持（Windows、macOS、iOS、Android、Web）
- ✅ 现代图形 API（Vulkan、Metal、WebGPU、WebGL）
- ✅ 可定制渲染管线
- ✅ PBR 材质系统
- ✅ 物理引擎集成（Box2D、Bullet、PhysX）
- ✅ 骨骼动画支持（Spine、DragonBones）
- ✅ 完善的 2D/3D 功能

## 🚀 快速开始

### 环境要求
- Node.js >= 18.0.0
- npm >= 9.0.0

### 安装和构建
```bash
# 安装依赖
npm install

# 构建引擎
npm run build

# 开发模式构建
npm run build:dev

# 运行测试
npm run test
```

## 📖 内容索引

### 1. 架构设计
- [整体架构](ARCHITECTURE.md#整体架构)
- [双语言架构原理](ARCHITECTURE.md#双语言架构原理)
- [模块依赖关系](ARCHITECTURE.md#模块依赖关系)

### 2. 核心模块
- [2D 模块](MODULES.md#2d-模块)
- [3D 模块](MODULES.md#3d-模块)
- [渲染模块](MODULES.md#渲染模块)
- [物理模块](MODULES.md#物理模块)
- [动画模块](MODULES.md#动画模块)
- [UI 模块](MODULES.md#ui-模块)
- [音频模块](MODULES.md#音频模块)

### 3. 平台抽象层
- [PAL 设计理念](MODULES.md#平台抽象层pal)
- [平台适配机制](MODULES.md#平台适配机制)

### 4. 构建系统
- [构建流程](BUILD_SYSTEM.md#构建流程)
- [输出产物](BUILD_SYSTEM.md#输出产物)
- [自定义构建](BUILD_SYSTEM.md#自定义构建)

## 🔗 相关资源

- [官方文档](https://docs.cocos.com/)
- [GitHub 仓库](https://github.com/cocos/cocos-engine)
- [社区论坛](https://discuss.cocos2d-x.org/)
- [API 参考](https://docs.cocos.com/creator/api/en/)

## 📝 文档贡献

如果您发现文档中有任何错误或需要补充的内容，欢迎提交 Issue 或 Pull Request。

---

*最后更新: 2026-05-06*
