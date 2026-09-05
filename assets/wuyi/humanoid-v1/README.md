# Wuyi Humanoid V1

`wuyi-humanoid-v1` 是 ONEWU / Wuyi Runtime 的默认无一人形显化动画资产包。

> **边界说明**：这是产品层默认表现（default/fallback presentation），不是《残剑宇宙》正史中“无一”的固定本体形态。无一仍然可以在不同产品、世界和上下文中以其他人物、宠物、装备、召唤物、粒子、雾化、数据形态或其他特殊结构显化。

## Summary

当前资产规范包含：

- `idle`：6 帧
- `walk`：8 帧
- `run`：6 帧
- `sit`：5 帧
- `sleep`：4 帧
- `wave`：4 帧
- `expressions`：8 个常用表情

默认适用于：

- ONEWU Web
- Wuyi Desktop
- Wuyi Runtime
- Wuyi Game SDK / 游戏宿主

## Directory Structure

```text
assets/wuyi/humanoid-v1/
├─ source/
│  └─ motion-reference-sheet.png
├─ animations/
│  ├─ idle/
│  │  ├─ 01.png
│  │  └─ ...
│  ├─ walk/
│  ├─ run/
│  ├─ sit/
│  ├─ sleep/
│  └─ wave/
├─ expressions/
│  ├─ normal.png
│  ├─ smile.png
│  ├─ blink.png
│  ├─ think.png
│  ├─ happy.png
│  ├─ doubt.png
│  ├─ sleepy.png
│  └─ silent.png
├─ reference/
│  └─ full-body.png
├─ previews/
│  ├─ idle.gif
│  ├─ walk.gif
│  └─ run.gif
├─ manifest.json
└─ README.md
```

## Asset Rules

### 1. Image Format

所有 Runtime 帧统一：

```text
PNG
Transparent background
sRGB
Canvas: 512 x 512 px
```

不得把原始浅色/蓝色参考表背景保留进最终 Runtime 帧。

### 2. Frame Naming

统一使用两位数字，从 `01` 开始：

```text
01.png
02.png
03.png
...
```

文件顺序就是播放顺序。

不要混用：

```text
frame1.png
walk_01.png
001.png
1.png
```

### 3. Animation Names

基础动作名称固定为：

```text
idle
walk
run
sit
sleep
wave
```

后续扩展继续使用 `snake_case`：

```text
drink_coffee
coding
reading
thinking
greeting
cat_interact
manifest
dematerialize
particle_idle
```

同一动作不要再创建 `walking`、`walk-cycle` 等同义名称。

### 4. Expression Names

当前固定：

```text
normal
smile
blink
think
happy
doubt
sleepy
silent
```

## Canvas / Anchor Standard

动画帧不能只做“裁得差不多”。所有帧必须对齐到统一 Canvas 和明确的接地锚点，否则播放时会产生：

- 左右抖动
- 上下跳动
- 脚底漂移
- 状态切换闪位

### Standing Anchor

适用于：

```text
idle
walk
run
wave
```

规范：

```text
anchor.x = 0.50
anchor.y = 0.92
```

人物中心保持在 Canvas 水平中心附近，双脚/主支撑脚使用同一地面基准线。

### Seated Anchor

适用于：

```text
sit
```

规范：

```text
anchor.x = 0.50
anchor.y = 0.90
```

以坐姿身体接地位置作为稳定基准，不强制继续对齐站立脚底线。

### Lying Anchor

适用于：

```text
sleep
```

规范：

```text
anchor.x = 0.50
anchor.y = 0.86
```

以卧姿主体接地线作为基准。

> 精确锚点以 `manifest.json` 为 Source of Truth。若未来动作重新修帧，需要更新资产版本，不要让各宿主自行维护另一套偏移参数。

## Animation Defaults

| Animation | Frames | FPS | Loop | Anchor |
|---|---:|---:|---|---|
| `idle` | 6 | 6 | yes | standing |
| `walk` | 8 | 10 | yes | standing |
| `run` | 6 | 12 | yes | standing |
| `sit` | 5 | 8 | no | seated |
| `sleep` | 4 | 4 | yes | lying |
| `wave` | 4 | 8 | no | standing |

这些是 V1 默认播放参数。宿主可以根据渲染帧率做时间补偿，但不应改变动作语义和帧顺序。

## Runtime Contract

Runtime 不应该硬编码：

```text
wuyi.png
walk = 8
fps = 10
```

正确流程：

```text
asset id
  ↓
load manifest.json
  ↓
animation name
  ↓
path / frame count / fps / loop / anchor
  ↓
load frames
  ↓
render
```

例如宿主只需要请求：

```json
{
  "asset": "wuyi-humanoid-v1",
  "animation": "walk"
}
```

Runtime 根据 `manifest.json` 解析：

```text
animations/walk/01.png ... 08.png
fps = 10
loop = true
anchor = standing
```

## Product Boundary

### Wuyi Identity != Humanoid V1

`wuyi-humanoid-v1` 只是一个显化资产包：

```text
Wuyi Identity
      ↓
Manifestation
      ↓
Asset Resolver
      ↓
wuyi-humanoid-v1
      ↓
Animation Renderer
```

因此：

- 删除或替换本资产包，不等于删除“无一”；
- 新增 `humanoid-v2` 不需要重写 Wuyi Core；
- 游戏显化为武器/宠物时可以完全不加载本资产；
- Web / Desktop 可以把本包作为默认或离线 fallback；
- 不得把本资产反向定义为残剑宇宙正史本体。

## Source and Runtime Assets

`source/` 中保存原始动作参考表，仅用于制作、复核、重新切帧。

真正 Runtime 使用：

```text
animations/
expressions/
reference/
manifest.json
```

宿主不要直接裁切 `source/motion-reference-sheet.png`。

## Previews

`previews/` 用于资产验收，不参与正式动画加载。

建议至少提供：

```text
previews/idle.gif
previews/walk.gif
previews/run.gif
```

Preview 的目的：

- GitHub 页面直接检查动作；
- 快速发现帧抖动；
- 检查帧顺序；
- 检查透明背景；
- 不启动 Wuyi Runtime 也能验收。

## Validation Checklist

提交新的动作帧前检查：

- [ ] PNG 为透明背景
- [ ] Canvas 为 512 × 512
- [ ] 文件从 `01.png` 开始连续编号
- [ ] 无缺帧/重复帧
- [ ] 人物比例在同一动作内一致
- [ ] Anchor 与 `manifest.json` 对齐
- [ ] standing 动作共用同一脚底基准
- [ ] sit / sleep 使用自己的接地基准
- [ ] Preview 无明显左右抖动
- [ ] Preview 无明显上下跳动
- [ ] 首尾循环动作衔接自然
- [ ] 非循环动作最后一帧可以安全停留/切状态
- [ ] 更新帧数时同步更新 `manifest.json`

## Versioning

资产包采用语义化版本：

```text
1.0.0
```

建议：

- 修正单帧透明边缘、微小位置：PATCH
- 增加动作/表情且保持兼容：MINOR
- 修改 Canvas、Anchor 语义或现有动作契约：MAJOR

目录 `humanoid-v1` 代表第一代资源系列；系列内部版本以 `manifest.json.version` 为准。

## Current Roadmap

基础动作完成后，优先增加：

```text
drink_coffee
coding
reading
thinking
greeting
cat_interact
```

随后再增加：

```text
manifest
dematerialize
particle_idle
```

黑猫建议未来作为独立 Entity / Asset Pack 管理，再通过 `cat_interact` 组合动作关联，而不是永久焊死到 Humanoid V1 的每一个基础动作中。
