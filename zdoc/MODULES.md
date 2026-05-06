# Cocos 4 模块详解

本文档详细介绍 Cocos 4 引擎中各个核心模块的功能和使用方法。

## 目录

1. [2D 模块](#2d-模块)
2. [3D 模块](#3d-模块)
3. [渲染模块](#渲染模块)
4. [物理模块](#物理模块)
5. [动画模块](#动画模块)
6. [UI 模块](#ui-模块)
7. [音频模块](#音频模块)
8. [场景图模块](#场景图模块)
9. [资源管理模块](#资源管理模块)
10. [平台抽象层](#平台抽象层pal)

---

## 2D 模块

路径: `cocos/2d/`

### 概述

2D 模块提供完整的 2D 游戏开发功能，包括精灵渲染、文本显示、UI 布局等。

### 核心组件

#### Sprite (精灵)

```typescript
// 2D 游戏中最重要的渲染组件
class Sprite extends UITransform {
  spriteFrame: SpriteFrame | null  // 精灵帧
  type: SpriteType                   // 精灵类型
  sizeMode: SizeMode                 // 尺寸模式
  fillType: FillType                 // 填充类型（用于进度条等）
  fillCenter: Vec2                    // 填充中心
  fillStart: number                  // 填充起始位置
  fillRange: number                  // 填充范围
  isTrimmedMode: boolean             // 是否使用原始尺寸
}

// SpriteType 枚举值
enum SpriteType {
  SIMPLE,     // 简单矩形精灵
  SLICED,     // 九宫格精灵
  TILED,      // 平铺精灵
  FILLED      // 填充精灵
}
```

#### Label (文本标签)

```typescript
// 2D 文本显示组件
class Label extends UITransform {
  string: string | I18nString           // 显示文本
  fontSize: number                       // 字体大小
  lineHeight: number                      // 行高
  fontFamily: string                      // 字体家族
  font: TTFFont | BitmapFont | null      // 字体资源
  horizontalAlign: HorizontalTextAlignment // 水平对齐
  verticalAlign: VerticalTextAlignment    // 垂直对齐
  overflow: LabelOverflow                 // 文本溢出处理
  enableWrap: boolean                    // 是否启用自动换行
}

// LabelOverflow 溢出处理方式
enum LabelOverflow {
  CLAMP,    // 截断
  SHRINK,   // 缩小字体适应
  RESIZE_HEIGHT // 自动调整高度
}
```

#### Graphics (图形绘制)

```typescript
// 2D 图形绘制组件
class Graphics extends UITransform {
  // 绘制操作
  moveTo(x: number, y: number): void
  lineTo(x: number, y: number): void
  bezierCurveTo(c1x: number, c1y: number, c2x: number, c2y: number, x: number, y: number): void
  quadraticCurveTo(cx: number, cy: number, x: number, y: number): void
  arc(cx: number, cy: number, r: number, startAngle: number, endAngle: number): void
  circle(x: number, y: number, r: number): void
  rect(x: number, y: number, w: number, h: number): void
  roundRect(x: number, y: number, w: number, h: number, r: number): void

  // 样式设置
  fillColor(color: Color): void
  strokeColor(color: Color): void
  lineWidth(width: number): void

  // 填充和描边
  fill(): void
  stroke(): void
  clear(): void
}
```

#### Mask (遮罩)

```typescript
// 遮罩组件，用于创建裁剪区域
class Mask extends UITransform {
  type: MaskType                // 遮罩类型
  inverted: boolean              // 是否反向
  alphaThreshold: number         // Alpha 阈值
  spriteFrame: SpriteFrame | null // 遮罩图形

  // MaskType 枚举值
  // RECT - 矩形遮罩
  // ELLIPSE - 椭圆遮罩
  // IMAGE_STENCIL - 图像遮罩
}
```

### 2D 渲染系统

#### Batcher2D (2D 批处理器)

```typescript
// 负责 2D 渲染的批处理优化
class Batcher2d {
  // 添加渲染数据
  addDrawInfo(drawInfo: RenderDrawInfo): void

  // 提交渲染命令
  uploadBuffers(): void

  // 渲染
  render(): void
}
```

### 2D 实用工具

#### TextUtils (文本工具)

```typescript
// 文本处理工具函数
namespace TextUtils {
  function safeCStr(str: string): number[]
  function strToUtf8(str: string): Uint8Array
  function getWordWidth(font: string, size: number, str: string): number
}
```

---

## 3D 模块

路径: `cocos/3d/`

### 概述

3D 模块提供完整的 3D 游戏开发功能，包括 3D 渲染、光照系统、模型加载等。

### 核心组件

#### MeshRenderer (网格渲染器)

```typescript
// 3D 网格渲染组件
class MeshRenderer extends Component {
  mesh: Mesh | null                    // 网格资源
  material: Material | null             // 材质
  materials: Material[]                 // 材质数组（用于多子网格）
  lightmapSettings: LightmapSettings    // 光照图设置

  // 启用/禁用
  enable: boolean

  // 接收阴影
  receiveShadow: boolean

  // 投射阴影
  castShadow: boolean
}
```

#### SkinnedMeshRenderer (蒙皮网格渲染器)

```typescript
// 蒙皮网格渲染器，用于骨骼动画
class SkinnedMeshRenderer extends MeshRenderer {
  skeleton: Skeleton | null    // 骨骼资源
  skinningRoot: Node | null     // 蒙皮根节点

  // 动画权重
  blendWeight: number
}
```

#### Camera (摄像机)

```typescript
// 3D 摄像机组件
class Camera extends Component {
  // 投影类型
  projectionType: ProjectionType

  // 视野角度
  fov: number
  fovAxis: number

  // 裁剪平面
  near: number
  far: number

  // 视口
  viewport: Rect

  // 清理标志
  clearFlags: ClearFlags

  // 背景颜色
  backgroundColor: Color

  // 渲染顺序
  priority: number

  // 适配方式
  aperture: number
  iso: number
  shutter: number
}

// 投影类型
enum ProjectionType {
  PERSPECTIVE,  // 透视投影
  ORTHOGRAPHIC  // 正交投影
}
```

### 光照系统

#### DirectionalLight (方向光)

```typescript
// 方向光，类似太阳光
class DirectionalLight extends Light {
  // 光照强度
  illuminance: number

  // 阴影配置
  shadowEnabled: boolean
  shadowPackingScheme: ShadowType
  shadowMapSize: number
  shadow Bias: number

  // 阴影级联
  shadowCascadeMode: ShadowCascadeMode
  numShadowCascades: number
}
```

#### PointLight (点光源)

```typescript
// 点光源，向所有方向发光
class PointLight extends Light {
  // 光照强度
  illuminance: number

  // 衰减配置
  importand: number        // 影响范围
  cosHalfThreshold: number // 衰减角度

  // 射程
  range: number
}
```

#### SpotLight (聚光灯)

```typescript
// 聚光灯，锥形光束
class SpotLight extends Light {
  // 光照强度
  illuminance: number

  // 锥形角度
  angle: number            // 半角
  innerAngle: number       // 内角
  outerAngle: number       // 外角

  // 射程
  range: number

  // 阴影
  shadowEnabled: boolean
}
```

### 3D 资产

#### Mesh (网格)

```typescript
// 3D 网格数据
class Mesh extends Asset {
  // 顶点数据
  getPositions(): Float32Array
  getNormals(): Float32Array
  getTangents(): Float32Array
  getUVs(layer: number): Float32Array
  getColors(): Float32Array

  // 索引数据
  getIndices(): Uint16Array | Uint32Array

  // 子网格
  getSubMesh(idx: number): RenderingSubMesh | null
  setSubMesh(idx: number, submesh: RenderingSubMesh): void

  // 绑定骨骼动画数据
  setBoneAnimationData(data: BoneAnimationData): void
}
```

#### Skeleton (骨骼)

```typescript
// 骨骼数据结构
class Skeleton extends Asset {
  // 骨骼数据
  get bones(): Bone[]
  get rootBone(): Bone | null
  get joints(): Joint[]

  // 骨骼索引
  getBoneIndex(boneName: string): number
  getBoneByIndex(index: number): Bone | null
}
```

---

## 渲染模块

路径: `cocos/rendering/` 和 `cocos/gfx/`

### 图形抽象层 (GFX)

GFX 层提供统一的图形 API 抽象，支持多种渲染后端。

#### Device (图形设备)

```typescript
// 图形设备接口
interface Device {
  // 能力查询
  getCapabilities(): DeviceCapabilities

  // 缓冲区
  createBuffer(info: BufferInfo): Buffer
  createBufferV2(info: BufferInfo, data: ArrayBuffer): Buffer

  // 纹理
  createTexture(info: TextureInfo): Texture
  createImageTexture(info: ImageTextureInfo): Texture

  // 着色器
  createShader(info: ShaderInfo): Shader
  createProgram(info: ProgramInfo): Program

  // 管线状态
  createPipelineState(info: PipelineStateInfo): PipelineState
  createRenderPass(info: RenderPassInfo): RenderPass

  // 描述符集
  createDescriptorSet(info: DescriptorSetInfo): DescriptorSet

  // 命令提交
  acquire(): void
  present(): void
}

// 设备能力
interface DeviceCapabilities {
  maxVertexAttributes: number
  maxVertexUniformVectors: number
  maxFragmentUniformVectors: number
  maxTextureUnits: number
  maxColorRenderTargets: number
  supportSRGB: boolean
  supportCompressedTextures: CompressedTextureType[]
}
```

#### Buffer (缓冲区)

```typescript
// GPU 缓冲区
interface Buffer {
  // 缓冲类型
  get type(): BufferType

  // 内存使用
  get memory(): number

  // 更新数据
  update(data: ArrayBuffer): void
}

// BufferType 枚举
enum BufferType {
  VERTEX,
  INDEX,
  UNIFORM,
  STORAGE,
  INDIRECT
}
```

#### Texture (纹理)

```typescript
// GPU 纹理
interface Texture {
  get type(): TextureType
  get format(): Format
  get width(): number
  get height(): number
  get depth(): number
  get mipLevel(): number
  get arrayLayer(): number
  get samples(): number

  // 更新纹理
  resize(width: number, height: number): void
}

// TextureType 枚举
enum TextureType {
  TEX2D,
  TEX2D_ARRAY,
  TEX3D,
  CUBE,
  CUBE_ARRAY
}
```

#### Shader (着色器)

```typescript
// 着色器程序
interface Shader {
  get name(): string
  get attributes(): Attribute[]
  get uniforms(): Uniform[]
}

// 着色器信息
interface ShaderInfo {
  name: string
  glsl4?: {
    vert: string
    frag: string
  }
  glsl3?: {
    vert: string
    frag: string
  }
  glsl1?: {
    vert: string
    frag: string
  }
  defines?: MacroInfo[]
}
```

### 渲染管线

#### RenderPipeline (渲染管线)

```typescript
// 渲染管线基类
class RenderPipeline {
  // 渲染 passes
  protected _passes: RenderPass[]

  // 添加渲染 Pass
  addRenderPass(pass: RenderPass): void

  // 移除渲染 Pass
  removeRenderPass(index: number): void

  // 更新渲染 Pass
  updatePass(index: number, pass: RenderPass): void

  // 渲染
  render(cameras: Camera[]): void
}

// 内置渲染 Pass 类型
enum PassType {
  UIPass,
  ForwardPass,
  DeferredPass,
  ShadowPass,
  ReflectionProbePass,
  PostProcessPass
}
```

---

## 物理模块

路径: `cocos/physics/`

### 概述

Cocos 4 集成了多个物理引擎，支持 2D 和 3D 物理模拟。

### 3D 物理引擎支持

| 引擎 | 说明 | 特点 |
|------|------|------|
| Bullet | 开源 3D 物理引擎 | 功能全面 |
| Cannon | 轻量级 3D 物理 | 性能优秀 |
| PhysX | NVIDIA 物理引擎 | 工业级品质 |
| Ammo | Bullet 的 WebAssembly 版本 | 跨平台兼容 |

### 2D 物理引擎

| 引擎 | 说明 |
|------|------|
| Box2D | 经典 2D 物理引擎 |
| Box2D-WASM | Box2D 的 WebAssembly 版本 |

### 核心接口

#### PhysicsWorld (物理世界)

```typescript
// 物理世界管理器
interface PhysicsWorld {
  // 场景碰撞配置
  enableCollisionMatrix(): void
  setCollisionMatrix(group: number, mask: number): void

  // 射线检测
  raycast(ray: Ray, mask: number, maxDistance: number): PhysicsRaycastResult
  raycastClosest(ray: Ray, mask: number, maxDistance: number): PhysicsRaycastResult

  // 碰撞事件
  on(collisionEvent: CollisionEventType, callback: Function): void

  // 步进模拟
  step(dt: number): void
}
```

#### IRigidBody (刚体接口)

```typescript
// 刚体接口
interface IRigidBody {
  // 质量
  mass: number
  invMass: number

  // 线性运动
  linearVelocity: Vec3
  angularVelocity: Vec3
  applyForce(force: Vec3, relativePoint?: Vec3): void
  applyImpulse(impulse: Vec3, relativePoint?: Vec3): void

  // 旋转
  applyTorque(torque: Vec3): void
  applyTorqueImpulse(torque: Vec3): void

  // 阻尼
  linearDamping: number
  angularDamping: number

  // 类型
  type: RigidBodyType
}

enum RigidBodyType {
  DYNAMIC,    // 动态刚体
  STATIC,     // 静态刚体
  KINEMATIC   // 运动学刚体
}
```

#### ICollider (碰撞器接口)

```typescript
// 碰撞器接口
interface ICollider {
  // 碰撞器形状
  shape: ColliderShape

  // 物理属性
  material: PhysicsMaterial
  attachedRigidBody: IRigidBody | null

  // 碰撞组
  group: number
  mask: number

  // 事件回调
  onCollisionEnter: Function | null
  onCollisionStay: Function | null
  onCollisionExit: Function | null
}
```

### 碰撞形状

```typescript
// 盒型碰撞器
class BoxCollider extends Component {
  shape: BoxShape
  getSize(): Vec3
  setSize(value: Vec3): void
}

// 球形碰撞器
class SphereCollider extends Component {
  shape: SphereShape
  getRadius(): number
  setRadius(value: number): void
}

// 胶囊碰撞器
class CapsuleCollider extends Component {
  shape: CapsuleShape
  getRadius(): number
  setRadius(value: number): void
  getHeight(): number
  setHeight(value: number): void
}

// 圆柱碰撞器
class CylinderCollider extends Component {
  shape: CylinderShape
  getRadius(): number
  setRadius(value: number): void
  getHeight(): number
  setHeight(value: number): void
}

// 锥形碰撞器
class ConeCollider extends Component {
  shape: ConeShape
  getRadius(): number
  setRadius(value: number): void
  getHeight(): number
  setHeight(value: number): void
}

// 网格碰撞器
class MeshCollider extends Component {
  shape: MeshShape
  getMesh(): Mesh | null
  setMesh(mesh: Mesh): void
  getConvex(): boolean
  setConvex(convex: boolean): void
}
```

---

## 动画模块

路径: `cocos/animation/`

### 概述

动画模块提供骨骼动画和关键帧动画系统。

### 核心类

#### Animation (动画组件)

```typescript
// 动画组件
class Animation extends Component {
  // 动画剪辑
  clips: AnimationClip[]
  defaultClip: AnimationClip | null

  // 播放控制
  play(name?: string): void
  stop(name?: string): void
  pause(name?: string): void
  resume(name?: string): void

  // 播放模式
  playOnLoad: boolean

  // 混合
  getBlendFactor(): number
  setBlendFactor(factor: number): void

  // 事件
  on(AnimEventType, callback: Function): void
}
```

#### AnimationClip (动画剪辑)

```typescript
// 动画剪辑数据
class AnimationClip extends Asset {
  // 剪辑信息
  name: string
  duration: number              // 持续时间（秒）
  speed: number                 // 播放速度
  sample: number                // 采样率

  // 曲线数据
  keys: KeyframeValues[]        // 关键帧数据
  curves: AnimationCurve[]       // 动画曲线

  // 事件
  events: AnimationEvent[]        // 动画事件

  // 创建曲线
  createCurve(target: string, property: string): AnimationCurve
}
```

#### AnimationCurve (动画曲线)

```typescript
// 动画曲线
class AnimationCurve {
  // 曲线类型
  path: string                  // 属性路径

  // 关键帧
  keyFrames: Keyframe[]

  // 插值模式
  interpolateFn: InterpolateFn

  // 获值
  evaluate(time: number): any
}
```

### 骨骼动画

#### SkeletalAnimation (骨骼动画)

```typescript
// 骨骼动画组件
class SkeletalAnimation extends Animation {
  // 骨骼资源
  skeleton: Skeleton | null

  // 动画控制器
  controller: AnimatorController | null

  // 混合模式
  getBlendMode(): BlendMode
  setBlendMode(mode: BlendMode): void
}

// 混合模式
enum BlendMode {
  INHERIT,
  ADDITIVE,
  OVERRIDE
}
```

---

## UI 模块

路径: `cocos/ui/`

### 概述

UI 模块提供完整的 2D UI 组件库，包括基础组件、容器组件和输入组件。

### 基础组件

#### Button (按钮)

```typescript
class Button extends UIComponent {
  // 状态
  clickEvents: EventHandler[]

  // 外观
  normalColor: Color
  pressedColor: Color
  hoverColor: Color
  disabledColor: Color

  // 精灵状态
  normalSprite: SpriteFrame | null
  pressedSprite: SpriteFrame | null
  hoverSprite: SpriteFrame | null
  disabledSprite: SpriteFrame | null

  // 过渡效果
  transition: ButtonTransition

  // 交互
  interactable: boolean
  enableAutoGrayEffect: boolean
}

enum ButtonTransition {
  NONE,
  COLOR,
  SPRITE,
  SCALE
}
```

#### Label (标签)

```typescript
class Label extends UIComponent {
  // 文本内容
  string: string
  horizontalAlign: HorizontalTextAlignment
  verticalAlign: VerticalTextAlignment

  // 字体
  fontSize: number
  lineHeight: number
  fontFamily: string
  font: TTF/font | null

  // 溢出处理
  overflow: LabelOverflow
  enableWrap: boolean

  // 颜色
  color: Color
  cacheMode: CacheMode
}
```

#### ProgressBar (进度条)

```typescript
class ProgressBar extends UIComponent {
  // 当前进度 [0, 1]
  progress: number

  // 模式
  mode: Mode

  // 填充
  barSprite: Sprite | null
  fillCenter: Vec2
  fillStart: number
  fillRange: number

  // 背景
  totalLength: number
  trackSprite: Sprite | null
}

enum Mode {
  PERCENT,    // 百分比模式
  VALUE       // 数值模式
}
```

#### Slider (滑动条)

```typescript
class Slider extends UIComponent {
  // 当前值 [0, 1]
  progress: number

  // 方向
  direction: Direction

  // 精灵
  handle: Sprite | null
  backgroundSprite: Sprite | null
  fillSprite: Sprite | null

  // 事件
  slideEvents: EventHandler[]
}

enum Direction {
  HORIZONTAL,
  VERTICAL
}
```

### 容器组件

#### Layout (布局容器)

```typescript
class Layout extends UIComponent {
  // 布局类型
  type: LayoutType

  // 容器尺寸
  containerSize: Size

  // 子节点适配
  resizeMode: ResizeMode
  cellSize: Size
  spacing: Vec2

  // 对齐
  horizontalAlign: HorizontalTextAlignment
  verticalAlign: VerticalTextAlignment

  // 边界
  padding: number
  paddingLeft: number
  paddingRight: number
  paddingTop: number
  paddingBottom: number
}

enum LayoutType {
  NONE,
  HORIZONTAL,
  VERTICAL,
  GRID
}

enum ResizeMode {
  NONE,
  CONTAINER,
  CHILDREN
}
```

#### ScrollView (滚动视图)

```typescript
class ScrollView extends UIComponent {
  // 内容
  content: Node | null
  viewport: Node | null

  // 滚动方向
  horizontalScrollBar: Scrollbar | null
  verticalScrollBar: Scrollbar | null

  // 滚动行为
  horizontal: boolean
  vertical: boolean
  invert: boolean

  // 滚动条
  scrollEvents: EventHandler[]
  autoScrollToBottom: boolean

  // 弹性
  elasticity: number
  brake: number
  bounceDuration: number
}
```

#### Widget (对齐组件)

```typescript
class Widget extends UIComponent {
  // 对齐目标
  target: Node | null

  // 边距
  left: number
  right: number
  top: number
  bottom: number

  // 对齐模式
  alignMode: AlignMode

  // 水平/垂直对齐
  isAlignLeft: boolean
  isAlignRight: boolean
  isAlignTop: boolean
  isAlignBottom: boolean
  isAlignHorizontalCenter: boolean
  isAlignVerticalCenter: boolean

  // 水平/垂直偏移
  horizontalCenter: number
  verticalCenter: number

  // 百分比模式
  isAbsoluteLeft: boolean
  isAbsoluteRight: boolean
  isAbsoluteTop: boolean
  isAbsoluteBottom: boolean
}
```

---

## 音频模块

路径: `cocos/audio/`

### AudioSource (音频源)

```typescript
class AudioSource extends Component {
  // 音频剪辑
  clip: AudioClip | null

  // 播放控制
  play(): void
  pause(): void
  stop(): void

  // 状态
  readonly playing: boolean

  // 音量
  volume: number

  // 循环
  loop: boolean

  // 播放时间
  currentTime: number
  duration: number

  // 播放速度
  playbackRate: number
}
```

---

## 场景图模块

路径: `cocos/scene-graph/`

### Node (节点)

```typescript
class Node extends CCObject {
  // 名称
  name: string
  uuid: string

  // 变换属性
  position: Vec3
  rotation: Quat
  scale: Vec3
  eulerAngles: Vec3

  // 世界变换
  worldPosition: Vec3
  worldRotation: Quat
  worldScale: Vec3
  worldMatrix: Mat4

  // 层级关系
  parent: Node | null
  children: Node[]

  // 激活状态
  active: boolean
  activeInHierarchy: boolean

  // 层级
  siblingIndex: number
  zIndex: number

  // 生命周期
  onLoad(): void
  start(): void
  update(dt: number): void
  lateUpdate(dt: number): void
  onDestroy(): void

  // 方法
  addChild(child: Node): void
  removeChild(child: Node): void
  setParent(parent: Node): void
  getSiblingIndex(): number
  setSiblingIndex(index: number): void
}
```

### Scene (场景)

```typescript
class Scene extends Node {
  // 场景信息
  name: string
  autoReleaseAssets: boolean

  // 激活
  activate(): void

  // 资源
  assets: Map<string, Asset>

  // 获取根节点
  readonly root: Node | null
}
```

---

## 资源管理模块

路径: `cocos/asset/`

### 核心类

#### Asset (资源基类)

```typescript
class Asset extends CCObject {
  // 资源信息
  name: string
  uuid: string
  url: string

  // 引用计数
  addRef(): void
  decRef(): void
  getRef(): number

  // 序列化
  serialize(): unknown
  deserialize(data: unknown): void
}
```

#### Mesh (网格资源)

```typescript
class Mesh extends Asset {
  // 顶点数据
  getPositions(): Float32Array
  getNormals(): Float32Array
  getTangents(): Float32Array
  getUVs(layer: number): Float32Array
  getColors(): Float32Array

  // 索引数据
  getIndices(): Uint16Array | Uint32Array
  getIndexCount(): number

  // 渲染数据
  getRenderingMesh(): RenderingMesh | null
  getSubMesh(idx: number): RenderingSubMesh | null
}
```

#### Texture2D (2D 纹理)

```typescript
class Texture2D extends Asset {
  // 纹理信息
  width: number
  height: number
  format: PixelFormat
  mipmapLevel: number

  // GL 句柄
  getGFXTexture(): Texture | null
  getGFXSamplerState(): SamplerInfo

  // 更新
  update(): void
  resize(width: number, height: number): void
}
```

---

## 平台抽象层 (PAL)

路径: `pal/`

### 概述

PAL (Platform Abstraction Layer) 提供平台无关的接口定义，根据不同平台提供具体实现。

### 模块列表

#### pal/audio - 音频

```typescript
// 音频播放器接口
interface AudioPlayer {
  // 播放控制
  play(): void
  pause(): void
  stop(): void

  // 状态
  readonly state: AudioState
  readonly duration: number
  readonly currentTime: number

  // 音量
  volume: number
  loop: boolean

  // 事件
  onStart?: () => void
  onPause?: () => void
  onStop?: () => void
  onEnd?: () => void
  onError?: (error: Error) => void
}

enum AudioState {
  INITIALIZED,
  PLAYING,
  PAUSED,
  STOPPED
}
```

#### pal/system-info - 系统信息

```typescript
// 系统信息接口
interface SystemInfo {
  // 平台
  readonly platform: Platform

  // 操作系统
  readonly os: OS

  // 浏览器
  readonly browserType: BrowserType
  readonly browserVersion: string

  // 屏幕
  readonly screenSize: Size
  readonly windowSize: Size
  readonly devicePixelRatio: number

  // 特性支持
  supportBitwiseOp: boolean
  supportWebGL: boolean
  supportWebGL2: boolean
  supportWebGPU: boolean
  supportWasm: boolean
  supportAudio: boolean

  // 获取 GPU 信息
  getGPUCapability(): GPUCapability
}
```

#### pal/input - 输入系统

```typescript
// 输入系统接口
interface Input {
  // 输入源
  get inputSources(): InputSource[]

  // 触摸
  setTouchCallback(callback: (touch: Touch, event: Event) => void): void

  // 鼠标
  setMouseCallback(callback: (mouse: MouseInput, event: Event) => void): void

  // 键盘
  setKeyboardCallback(callback: (keyboard: KeyboardInput, event: Event) => void): void

  // 游戏手柄
  setGamepadCallback(callback: (gamepad: GamepadInput, event: Event) => void): void
}

// 输入源类型
enum InputSourceType {
  KEYBOARD,
  MOUSE,
  TOUCH,
  GAMEPAD
}
```

#### pal/wasm - WASM 支持

```typescript
// WASM 模块加载器
interface WASM {
  // 加载 WASM 二进制
  loadWasm(url: string, importObject?: object): Promise<WebAssembly.Instance>

  // 加载 WASM 模块
  loadWasmModule(moduleName: string, url: string, exportName: string, importObject?: object): Promise<Function>

  // 内存
  getMemory(): WebAssembly.Memory | null
}
```

---

## 总结

本文档详细介绍了 Cocos 4 引擎的各个核心模块。每个模块都经过精心设计，采用分层和模块化架构，便于扩展和维护。

更多关于构建系统的信息，请参考 [构建系统文档](BUILD_SYSTEM.md)。
