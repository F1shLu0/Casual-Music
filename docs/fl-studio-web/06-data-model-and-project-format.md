# 06 · 数据模型与工程格式

> 目标：定义**可版本化、可迁移、可协作、可校验**的工程数据模型，并给出 `.flw` 工程包格式。
> 原则：ID 稳定、引用不靠数组下标、音频资源内容寻址、所有结构变更可迁移。

---

## 1. 顶层模型

```
Project
├── schemaVersion
├── id / meta
├── settings（BPM、拍号、采样率、颜色、尾音策略）
├── tempoMap
├── assets[]            # 采样/IR/视频等二进制资源引用
├── patterns[]          # 每个 Pattern 保存各通道的音符/步进/事件
├── channels[]          # 通道机架（全局，跨 Pattern 共享）
├── mixer               # 500 Insert + Current + Master
├── playlist            # Arrangements + Tracks + Clips
├── automationClips[]   # 参数曲线（也可内联在 playlist）
├── controllers[]       # Envelope/LFO/Formula/XY/Peak 等调制源
├── presets[]           # 工程内嵌预设（可选）
├── markers[]           # 时间标记/循环/拍号
└── ui?                 # 可选：布局/缩放（默认不入工程）
```

### 1.1 设计要点

- **Channel 与 Pattern 解耦**：Pattern 只保存"每个通道的数据"，通道全局唯一。
- **Playlist 与 Mixer 解耦**：Clip 引用 Pattern/Asset/Automation，不直接引用 Mixer；声音路由由 Channel → Mixer 决定。
- **Asset 内容寻址**：`assetId = sha256(file)[:16]`，天然去重，跨工程可复用。
- **ID 稳定**：UUIDv7/NaNoid；所有引用用 ID，不用数组索引。
- **可迁移**：`schemaVersion` 单调递增；迁移函数链式执行。
- **可校验**：Zod schema + 语义校验（引用完整性、范围、环路）。

---

## 2. 实体定义（TypeScript 形态）

> 以下为**数据契约**，不是实现代码；字段名将作为 JSON 键。

### 2.1 Project / Meta / Settings

```ts
interface Project {
  schemaVersion: number;          // 当前 1
  id: string;                     // uuid
  meta: ProjectMeta;
  settings: ProjectSettings;
  tempoMap: TempoMap;
  assets: Asset[];
  patterns: Pattern[];
  channels: Channel[];
  mixer: Mixer;
  playlist: Playlist;
  automationClips: AutomationClip[];
  controllers: Controller[];
  presets?: EmbeddedPreset[];
  markers: Marker[];
  ui?: UiState;                   // 可选，默认不写入分享工程
}

interface ProjectMeta {
  title: string;
  author?: string;
  description?: string;
  createdAt: string;              // ISO 8601
  modifiedAt: string;
  appVersion: string;
  tags?: string[];
  license?: string;
}

interface ProjectSettings {
  sampleRate: number;             // 44100 | 48000 | 96000
  timeSignature: { numerator: number; denominator: number };
  globalSwing: number;            // 0..1
  swingMode: 'truncate' | 'allowOverlap';
  tailMode: 'leave' | 'wrap' | 'cut';
  color: string;                  // 工程强调色
  metronome: { enabled: boolean; volume: number; onlyRecording: boolean };
  countInBars: 0 | 1 | 2 | 4;
  loop: { enabled: boolean; startTick: number; endTick: number };
  gridSnap: SnapMode;
  ppq: 3840;                      // 固定常量
}

type SnapMode = 'main' | 'bar' | 'beat' | 'step' | 'line' | 'none';
```

### 2.2 TempoMap / Marker

```ts
interface TempoMap {
  points: TempoPoint[];           // 至少一个，tick 升序
}

interface TempoPoint {
  tick: number;
  bpm: number;                    // 10..522，支持 3 位小数
  timeSignature?: { numerator: number; denominator: number };
}

interface Marker {
  id: string;
  tick: number;
  name: string;
  kind: 'time' | 'loopStart' | 'loopEnd' | 'songEnd' | 'punchIn' | 'punchOut';
  color?: string;
  timeSignature?: { numerator: number; denominator: number };
}
```

**规则**

- TempoMap 第一个点必须在 tick 0；重复 tick 需合并。
- BPM 自动化通过 TempoMap 点实现；导出必须按 tick 精确换算。
- Marker 的 tick 必须落在合法范围；loopStart < loopEnd。

### 2.3 Asset

```ts
interface Asset {
  id: string;                     // sha256 前 16 位
  name: string;
  kind: 'sample' | 'ir' | 'video' | 'midi' | 'other';
  mime: string;
  size: number;
  durationSec?: number;
  sampleRate?: number;
  channels?: number;
  bitDepth?: number;
  bpm?: number;
  key?: string;                   // 如 "C4"
  tags?: string[];
  license: AssetLicense;
  source?: string;                // 来源 URL/包名
  blobRef: string;                // zip 内路径或 OPFS key
  peaksRef?: string;              // 波形峰值缓存
  missing?: boolean;              // 打开时缺失
}

interface AssetLicense {
  type: 'cc0' | 'cc-by' | 'commercial' | 'user' | 'unknown';
  name?: string;
  url?: string;
  attribution?: string;
}
```

**规则**

- `assetId` 由内容哈希生成；同内容不同文件名视为同一 Asset。
- 缺失资源保留 Asset 记录并置 `missing=true`，不阻塞加载。
- 用户录音默认 `license.type='user'`，仅本地使用。

### 2.4 Channel

```ts
interface Channel {
  id: string;
  kind: 'instrument' | 'sampler' | 'audio' | 'automation' | 'layer' | 'controller';
  name: string;
  color: string;
  engineId: string;               // 如 'triple-osc' / 'sampler' / 'fpc'
  presetId?: string;
  params: Record<string, ParamValue>;
  mixerTrackId: string;
  gain: number;                   // dB
  pan: number;                    // -1..1
  pitch: number;                  // 半音
  mute: boolean;
  solo: boolean;
  swingMix: number;               // 0..1
  group?: string;
  displayFilter?: string;
  sampleId?: string;              // sampler/audio
  layer?: LayerConfig;
  controller?: ControllerConfig;
  enabled: boolean;
  notes?: string;
}

interface LayerConfig {
  children: string[];             // channel ids
  soloChildId?: string;
  crossfade: boolean;
  mode: 'layer' | 'split';
}

interface ControllerConfig {
  type: 'envelope' | 'lfo' | 'formula' | 'peak' | 'xy' | 'keyboard';
  params: Record<string, number | string>;
}
```

**规则**

- `mixerTrackId` 必须存在；删除 Mixer Track 时需重定向或阻止。
- Layer 的 children 不得成环；不得引用自身。
- Channel 删除前检查 Pattern/Playlist/Automation 引用。

### 2.5 Pattern 与音符/步进

```ts
interface Pattern {
  id: string;
  name: string;
  color: string;
  lengthTicks: number;            // 1..(512 steps × stepTicks)
  timeSignature?: { numerator: number; denominator: number };
  swing?: number;                 // 覆盖全局
  channelData: Record<string, ChannelPatternData>;
  notes?: string;
}

interface ChannelPatternData {
  channelId: string;
  notes?: Note[];                 // Piano Roll 数据
  steps?: Step[];                 // Step Sequencer 数据
  events?: EventAutomation[];     // Event Automation（控制器事件）
}

interface Note {
  id: string;
  startTick: number;              // 相对 Pattern 起点
  lengthTicks: number;            // >0
  pitch: number;                  // MIDI 0..127
  velocity: number;               // 0..1
  release?: number;               // 0..1
  pan?: number;                   // -1..1
  pitchFine?: number;             // 半音微调
  modX?: number;                  // 0..1（Cutoff 等）
  modY?: number;                  // 0..1（Resonance 等）
  slide?: boolean;
  portamento?: boolean;
  delayTicks?: number;            // 音符延迟
  mute?: boolean;
  group?: number;                 // 音符分组
  colorGroup?: number;            // MIDI 通道/颜色组 0..15
  articulation?: string;          // 如 'legato' / 'staccato'
}

interface Step {
  index: number;                  // 0..511
  velocity: number;               // 0..1
  kind?: 'normal' | 'trigger' | 'gate';
  mute?: boolean;
}

interface EventAutomation {
  id: string;
  paramId: string;                // 控制器参数
  tick: number;
  value: number;
  curve?: number;                 // 张力
}
```

**存储优化**

- 小 Pattern（<1,000 音符）用对象数组，便于调试。
- 大 Pattern 用**列式数组**：`notes: { start: number[], len: number[], pitch: number[], vel: number[] }`，减少 JSON 体积。
- 运行时统一转换为 TypedArray（Int32/Float32）供音频线程使用；保存时再序列化。

### 2.6 Mixer

```ts
interface Mixer {
  tracks: MixerTrack[];           // 最多 500
  currentTrackId: string;
  masterTrackId: string;          // 固定 'master'
}

interface MixerTrack {
  id: string;
  index: number;                  // 0..499；Master 固定 -1
  name: string;
  color: string;
  fader: number;                  // 0..1（映射 dB）
  pan: number;                    // -1..1
  mute: boolean;
  solo: boolean;
  arm: boolean;
  input?: { deviceId: string; channel: number };
  eq: { low: number; mid: number; high: number }; // dB
  phaseInvert: { left: boolean; right: boolean };
  swapChannels: boolean;
  stereoSeparation: number;       // 0..1
  trackDelaySamples: number;      // 手动 PDC
  sends: Send[];
  slots: FxSlot[];                // 10
  pluginState?: Record<string, unknown>;
}

interface Send {
  toTrackId: string;
  level: number;                  // 0..1
  enabled: boolean;
  preFader: boolean;
}

interface FxSlot {
  id: string;
  effectId?: string;              // 为空表示空槽
  enabled: boolean;
  wet: number;                    // 0..1
  gainDb: number;
  params: Record<string, ParamValue>;
  presetId?: string;
  oversampling?: 1 | 2 | 4;
}
```

**规则**

- `sends` 目标必须存在；禁止自环；全局检测环路。
- 删除轨道：若被发送引用，需重接或阻止；Master 不可删除。
- 未使用轨（无 slot、无发送、无通道路由）不实例化 DSP。

### 2.7 Playlist / Arrangement / Clip

```ts
interface Playlist {
  arrangements: Arrangement[];
  activeArrangementId: string;
  tracks: PlaylistTrack[];        // 500，跨 Arrangement 共享轨道属性
}

interface Arrangement {
  id: string;
  name: string;
  clips: Clip[];
  markers?: string[];             // Marker ids
  createdAt: string;
}

interface PlaylistTrack {
  id: string;
  index: number;
  name: string;
  color: string;
  mute: boolean;
  solo: boolean;
  locked: boolean;
  height: number;
  trackMode?: { channelId: string }; // 1:1 绑定
}

interface Clip {
  id: string;
  trackId: string;
  type: 'pattern' | 'audio' | 'automation';
  startTick: number;
  lengthTicks: number;
  offsetTicks: number;            // 源内偏移
  sourceId: string;               // patternId | assetId | automationClipId
  gainDb: number;
  fadeInTicks: number;
  fadeOutTicks: number;
  mute: boolean;
  color?: string;
  name?: string;
  loop: boolean;
  timeStretch?: { ratio: number; mode: 'offline' | 'realtime' };
  pitchSemitones?: number;
  reverse?: boolean;
  sliceMarkers?: number[];        // 采样内切片点
  automationId?: string;          // 若 type=automation
}
```

**规则**

- Clip 的 `trackId` 必须存在；`sourceId` 必须存在或标记缺失。
- `startTick >= 0`，`lengthTicks > 0`，`offsetTicks >= 0`。
- 同一轨道允许重叠；选择焦点按"最上层 + 最后编辑"规则。
- 删除 Arrangement 不得删除共享 Channel/Mixer/Pattern。

### 2.8 AutomationClip

```ts
interface AutomationClip {
  id: string;
  name: string;
  color: string;
  target: ParamTarget;
  mode: 'curve' | 'step' | 'slide';
  points: AutomationPoint[];
  initValue: number;              // 删除后恢复值
  lengthTicks: number;
}

interface AutomationPoint {
  tick: number;
  value: number;                  // 0..1 归一化
  tension?: number;               // -1..1
  type?: 'linear' | 'hold' | 'smooth' | 'sine' | 'exp';
}

interface ParamTarget {
  kind: 'channel' | 'mixer' | 'fx' | 'master' | 'tempo' | 'clip';
  channelId?: string;
  mixerTrackId?: string;
  slotId?: string;
  paramId: string;
}
```

**规则**

- `points` 按 tick 升序；同 tick 只允许一个点（后写覆盖）。
- 目标参数必须存在且可自动化；参数删除时 Automation 标记失效并提示。
- 删除 AutomationClip 时参数回到 `initValue`。

### 2.9 Controller 与参数链接

```ts
interface Controller {
  id: string;
  type: 'envelope' | 'lfo' | 'formula' | 'peak' | 'xy' | 'keyboard' | 'macro';
  params: Record<string, ParamValue>;
  outputs: ControllerOutput[];
}

interface ControllerOutput {
  id: string;
  name: string;
  value: number;
}

interface ParamLink {
  id: string;
  source: { controllerId: string; outputId: string } | { midiCC: number };
  target: ParamTarget;
  mapping: { min: number; max: number; curve: 'linear' | 'log' | 'exp' };
  enabled: boolean;
}
```

### 2.10 ParamValue

```ts
type ParamValue = number | boolean | string;
```

- 数值统一归一化到 0..1 存储，展示时按 `ParamDef` 映射单位。
- 枚举/开关用 string/boolean；避免存浮点布尔。

---

## 3. 工程文件格式 `.flw`

### 3.1 容器

`.flw` = **ZIP**（deflate），目录结构：

```
project.flw
├── manifest.json          # 格式版本、文件清单、哈希、许可证摘要
├── project.json           # 上述 Project（不含二进制）
├── assets/
│   ├── <assetId>.wav      # 原始采样（或压缩缓存）
│   ├── <assetId>.flac
│   └── <assetId>.ir
├── peaks/
│   └── <assetId>.json     # 波形峰值金字塔（可选，可重建）
├── previews/
│   └── thumbnail.png      # 工程缩略图
└── licenses/
    └── assets.json        # 采样授权明细（可公开）
```

### 3.2 manifest.json

```json
{
  "format": "openbeat-project",
  "schemaVersion": 1,
  "appVersion": "0.1.0",
  "createdAt": "2026-01-01T00:00:00Z",
  "projectId": "…",
  "files": [
    { "path": "project.json", "sha256": "…", "bytes": 12345 },
    { "path": "assets/ab12.wav", "sha256": "…", "bytes": 44100 }
  ],
  "assets": [
    { "id": "ab12", "path": "assets/ab12.wav", "license": "cc0", "source": "…" }
  ]
}
```

### 3.3 打包规则

- 保存时**默认打包所有被引用采样**；用户可选择"仅引用外部文件"（不推荐，易丢失）。
- 采样保持原格式；若需转换（如 MP3→WAV），写入转换后的副本并记录 `originalFormat`。
- 大工程可流式写入 ZIP（分块），避免一次性内存峰值。
- 导出分轨时另建 ZIP：`stems/<trackName>.wav` + `manifest.json`。

### 3.4 打开与校验顺序

1. 检查 ZIP 大小/条目数/压缩比（防 ZIP bomb）。
2. 读取 `manifest.json`，校验 `format` 与 `schemaVersion`。
3. 校验 `project.json` schema（Zod）。
4. 校验引用完整性：channel→mixer、clip→source、automation→target。
5. 校验路由无环、Layer 无环、数值范围。
6. 校验资源哈希；缺失/损坏则标记 `missing`。
7. 迁移到当前版本。
8. 构建运行时模型（TypedArray/索引）。

**失败策略**

- schema 失败：不覆盖原文件，提示错误码 + 可导出诊断。
- 资源缺失：允许打开，静音占位。
- 迁移失败：保留原文件副本，提示升级应用。

---

## 4. 版本迁移

- `schemaVersion` 为整数；每次破坏性变更 +1。
- 迁移函数：`migrate[v] : (project) => project`，按序执行。
- 迁移必须幂等、可测试（提供各版本 fixture）。
- 未知更高版本：只读打开或拒绝，禁止静默降级写回。
- 字段废弃：先保留读取兼容，再在下一大版本移除。

**迁移示例**

| 版本 | 变更 | 迁移 |
|---|---|---|
| 1 → 2 | Note 增加 `colorGroup` | 默认 0 |
| 2 → 3 | Mixer sends 增加 `preFader` | 默认 false |
| 3 → 4 | Automation 增加 `mode` | 默认 'curve' |

---

## 5. FLP 导入映射（尽力）

| FLP 实体 | 本项目 | 处理 |
|---|---|---|
| Channel（Generator/Sampler） | Channel | 按插件名匹配最接近引擎；未知插件 → 占位采样器 + 警告 |
| Pattern | Pattern | 保留名称/颜色/长度；音符转 Note |
| Playlist Item | Clip | Pattern/Audio/Automation 分别映射 |
| Mixer Track | MixerTrack | 推子/pan/发送/插入效果尽力映射 |
| Automation Clip | AutomationClip | 曲线点转换；目标参数按名称匹配 |
| Plugin State | `pluginState` | 原样保存（不解析），用于诊断 |
| Sample | Asset | 从 FLP 内嵌或外部路径提取；缺失标记 |
| Time Marker | Marker | 名称/tick 映射 |
| Arrangement | Arrangement | 多 Arrangement 保留 |

**限制**

- 不保证 100% 兼容；未知插件/效果器以"旁通占位"呈现。
- FLP 的某些内部曲线/包络无法精确还原，需提示。
- 导入前备份原文件；导入结果生成"导入报告"（成功/降级/失败清单）。

---

## 6. 撤销、协作与序列化

### 6.1 撤销

- 命令层维护逆操作；不把撤销栈写入工程文件。
- 结构变更（增删实体）使用"删除墓碑 + 恢复快照"，避免 ID 复用。
- 参数拖动合并：`coalesceKey = entityId + paramId`，300ms 内合并。

### 6.2 协作（P2）

- Yjs 文档与上述实体一一对应：
  - `channels: Y.Map<Channel>`
  - `patterns: Y.Map<Pattern>`
  - `clips: Y.Map<Clip>`
  - `mixer.tracks: Y.Map<MixerTrack>`
- 删除用 tombstone（`deleted: true`），避免并发复活。
- 数组顺序用 `Y.Array<string>` 存 ID，不存对象。
- Awareness 只同步 presence，不写入文档。

### 6.3 运行时模型 vs 持久化模型

| 场景 | 形态 |
|---|---|
| 编辑 | 归一化对象 + ID 索引（Map） |
| 音频线程 | TypedArray + 紧凑事件队列 + 只读快照 |
| 保存 | JSON（大列表列式数组）+ ZIP |
| 协作 | Yjs Map/Array |
| 撤销 | 命令 + 补丁 |

---

## 7. 校验规则清单

| 规则 | 处理 |
|---|---|
| ID 唯一 | 重复则拒绝加载 |
| 引用存在 | 缺失则标记并降级 |
| 数值范围 | 超界夹紧并警告 |
| tick 非负、升序 | 修正或拒绝 |
| 路由无环 | 拒绝连接 |
| Layer 无环 | 拒绝 |
| Clip 长度 > 0 | 修正为 1 tick |
| 采样率合法 | 否则重采样 |
| ZIP 条目/大小限制 | 拒绝 |
| JSON 深度/大小限制 | 拒绝 |
| 未知字段 | 保留（前向兼容） |
| 未知枚举值 | 降级为默认并警告 |

---

## 8. 容量与性能

- 单工程音符软上限 1,000,000；超过提示拆分工程。
- 单 Pattern 音符软上限 100,000。
- 采样数软上限 5,000；资产总大小软上限 2GB。
- `project.json` 超过 50MB 时启用列式数组 + gzip。
- 加载时并行解码采样（并发 4–8），优先解码当前视图所需。
- 索引：`channelById`、`patternById`、`clipByTrack`、`noteByTickBucket`。

---

## 9. 示例（最小工程）

```json
{
  "schemaVersion": 1,
  "id": "prj_01",
  "meta": { "title": "Demo", "createdAt": "2026-01-01T00:00:00Z", "modifiedAt": "2026-01-01T00:00:00Z", "appVersion": "0.1.0" },
  "settings": {
    "sampleRate": 48000,
    "timeSignature": { "numerator": 4, "denominator": 4 },
    "globalSwing": 0,
    "swingMode": "truncate",
    "tailMode": "leave",
    "color": "#4f8cff",
    "metronome": { "enabled": false, "volume": 0.5, "onlyRecording": false },
    "countInBars": 0,
    "loop": { "enabled": false, "startTick": 0, "endTick": 0 },
    "gridSnap": "beat",
    "ppq": 3840
  },
  "tempoMap": { "points": [{ "tick": 0, "bpm": 120 }] },
  "assets": [],
  "patterns": [{
    "id": "pat_01", "name": "Pattern 1", "color": "#4f8cff", "lengthTicks": 15360,
    "channelData": {
      "ch_01": { "steps": [{ "index": 0, "velocity": 1 }, { "index": 4, "velocity": 1 }] }
    }
  }],
  "channels": [{
    "id": "ch_01", "kind": "instrument", "name": "Kick", "color": "#ff6b6b",
    "engineId": "drum-synth", "params": { "decay": 0.3, "pitch": 0.2 },
    "mixerTrackId": "mt_01", "gain": 0, "pan": 0, "pitch": 0,
    "mute": false, "solo": false, "swingMix": 0, "enabled": true
  }],
  "mixer": {
    "tracks": [{
      "id": "mt_01", "index": 0, "name": "Kick", "color": "#ff6b6b",
      "fader": 0.8, "pan": 0, "mute": false, "solo": false, "arm": false,
      "eq": { "low": 0, "mid": 0, "high": 0 },
      "phaseInvert": { "left": false, "right": false },
      "swapChannels": false, "stereoSeparation": 1, "trackDelaySamples": 0,
      "sends": [], "slots": []
    }],
    "currentTrackId": "mt_01",
    "masterTrackId": "master"
  },
  "playlist": {
    "arrangements": [{ "id": "arr_01", "name": "Arrangement 1", "clips": [], "createdAt": "2026-01-01T00:00:00Z" }],
    "activeArrangementId": "arr_01",
    "tracks": [{ "id": "plt_01", "index": 0, "name": "Track 1", "color": "#4f8cff", "mute": false, "solo": false, "locked": false, "height": 80 }]
  },
  "automationClips": [],
  "controllers": [],
  "markers": []
}
```

---

## 10. 与需求文档的对应

- `01-product-requirements.md` 的 AC-07（持久化/恢复）依赖本文的 schema 与自动保存。
- `02-feature-specification.md` 的 FR-PRJ/FR-HIS 依赖本文的迁移与撤销模型。
- `03-audio-and-technical-architecture.md` 的音频快照/协作依赖本文的运行时/持久化分离。
- `04-instruments-catalog.md` / `05-effects-catalog.md` 的参数 schema 遵循本文 `ParamDef` 约定。