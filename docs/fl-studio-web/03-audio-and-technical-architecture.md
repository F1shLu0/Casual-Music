# 03 · 技术架构与音频引擎

> 目标：在浏览器中实现**采样级精确、可离线渲染、可扩展**的 DAW 音频引擎，并支撑 500 轨/万级音符的 UI。
> 本文是技术方案与约束，不包含实现代码。

---

## 1. 约束与设计原则

### 1.1 硬约束

- 浏览器只有 Web Audio API / AudioWorklet 可用；不能加载原生 VST/AU。
- 音频线程（AudioWorklet）不能阻塞、不能分配大对象、不能做文件 I/O。
- 主线程负责 UI；音频线程必须与 UI 解耦，UI 卡顿不能导致 dropout。
- 移动端内存/发热受限；大采样必须流式/按需解码。
- 浏览器标签页后台会被节流；需要处理 suspend/resume 与时钟漂移。

### 1.2 设计原则

1. **单一 DSP Graph**：整个工程在一个 AudioWorklet 内渲染，而不是为每个插件创建 Web Audio 节点。这样调度、自动化、PDC、离线渲染完全一致。
2. **数据与音频分离**：工程状态是纯数据；音频线程只读快照 + 参数事件，不直接持有 UI 对象。
3. **无锁音频线程**：参数通过环形缓冲区/原子操作传递；音频线程不做 JSON、不做 DOM、不 GC 密集分配。
4. **可离线**：实时与离线共用同一套 DSP 与调度代码，导出结果可复现。
5. **渐进增强**：SharedArrayBuffer/多线程/WASM SIMD 有则用，无则降级。
6. **可测试**：DSP 纯函数化，黄金音频回归；时间数学属性测试。

---

## 2. 技术栈选型

| 层 | 选型 | 理由 | 备选/备注 |
|---|---|---|---|
| 语言 | TypeScript（strict） | 类型安全、生态、团队熟悉 | Rust 仅用于重 DSP |
| UI | React 19 + Vite | 组件化、快速迭代 | Svelte/Solid 更轻但生态弱 |
| 样式 | CSS Modules + design tokens | 可控、无运行时 | Tailwind 可选 |
| 时间线渲染 | Canvas 2D（分层 + 虚拟化） | 500 轨/万音符稳定 | PixiJS/WebGL 用于极大数据 |
| 状态 | Zustand + Immer | 细粒度订阅、简单 | Redux Toolkit 较重 |
| 撤销 | Command Bus + 逆操作/补丁 | 可预测、可协作迁移 | 全量快照内存大 |
| 校验 | Zod | 运行时 schema 校验 | 手写校验 |
| 音频 | Web Audio + AudioWorklet | 浏览器标准、低延迟 | Tone.js 仅用于原型 |
| 重 DSP | Rust + wasm-bindgen | 加法/卷积/频谱/建模性能 | AssemblyScript |
| 解码 | Web Audio decodeAudioData + WASM 解码器 | 兼容广 | WebCodecs 可选 |
| 编码 | WAV（自写）、MP3（lamejs/WASM）、FLAC（libflacjs）、Opus/AAC（WebCodecs） | 覆盖主流 | 需许可证核验 |
| MIDI | Web MIDI + @tonejs/midi | 设备与文件 | Web MIDI 兼容性有限 |
| 存储 | IndexedDB + OPFS + Cache API | 本地优先、大文件 | 云端后置 |
| 协作 | Yjs + y-indexeddb + WebSocket provider | CRDT、离线合并 | 自研 OT 成本高 |
| 测试 | Vitest + Playwright + 黄金音频 | 单元/端到端/音频回归 | Jest 较慢 |
| 监控 | 自建轻量 telemetry（可选） | 音频故障定位 | Sentry 可选 |

### 2.1 为什么不用 Tone.js 作为核心

Tone.js 适合快速原型与常规节点图，但：

- 每个音符/插件对应多个 Web Audio 节点，500 轨/64 复音时节点数与 GC 压力大；
- 自动化与 PDC 难以做到采样级精确；
- 离线渲染与实时渲染的调度路径不完全一致；
- 难以统一实现"工程快照 → 音频线程"的数据流。

**结论**：核心引擎自研；Tone.js 仅用于早期 Spike/对照测试。

### 2.2 为什么不用 Elementary Audio 作为核心

Elementary 的声明式 DSP 很适合合成器，但其渲染器与节点模型仍偏向"每个插件一个图"，且生态/调试工具较新。可作为个别插件的参考实现，不作为全局引擎。

---

## 3. 系统总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Main Thread (UI)                           │
│  React Shell ── Zustand Store ── Command Bus ── Selectors            │
│      │              │                │                              │
│      ├── Canvas Views (Playlist / PianoRoll / Mixer meters)          │
│      ├── Browser / Presets / Settings                               │
│      └── Project I/O Worker ── IndexedDB / OPFS / File System        │
│                              │                                      │
│                    postMessage (events, snapshots)                   │
│                              ▼                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    AudioWorklet (Audio Thread)                 │  │
│  │  EngineProcessor                                              │  │
│  │   ├── Transport / TempoMap / Scheduler                        │  │
│  │   ├── Track Graph (500 inserts, sends, PDC)                   │  │
│  │   ├── Instrument Voices (synth/sampler/drum)                  │  │
│  │   ├── FX Chain (10 slots/track)                               │  │
│  │   ├── Automation / Parameter Ring Buffers                     │  │
│  │   └── Metering / Glitch Detection                             │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│                     AudioDestinationNode                            │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.1 关键数据流

1. **编辑**：UI 产生 Command → 更新 Store → 生成增量事件（增删改）→ 通过 MessagePort 发送到音频线程。
2. **播放**：音频线程按 TempoMap 计算未来 100–200ms 的事件窗口，触发 voice/FX 参数。
3. **参数**：高频参数（推子、自动化）走 SAB 环形缓冲；低频/结构变更走 MessagePort。
4. **快照**：打开工程/撤销跳转时，发送完整工程快照（结构化克隆），音频线程重建图。
5. **导出**：主线程创建 OfflineAudioContext，加载同一 Worklet，发送同一快照与渲染范围。

---

## 4. 音频引擎设计

### 4.1 EngineProcessor

- 每个 AudioContext 一个 `EngineProcessor`（`AudioWorkletProcessor`）。
- 内部维护：
  - `TransportState`（播放状态、位置、循环、速度、半速）
  - `TempoMap`（速度/拍号分段，支持自动化）
  - `TrackGraph`（Mixer 路由、发送、PDC）
  - `VoiceManager`（每乐器 voice pool）
  - `FxRegistry`（DSP 实例）
  - `EventQueue`（未来事件）
  - `ParamRing`（参数更新）
- `process()` 以 128 帧为量子；内部按事件时间切分，保证采样级精度。

### 4.2 调度器

- **Lookahead**：默认 150ms（可 50–300ms 配置）。
- 事件类型：NoteOn/NoteOff、Automation、Clip 开始/结束、Tempo 变更、Loop 跳转、FX 参数。
- 事件时间用 `double` 秒 + tick 双表示；跨 tempo 段时按 tick 推进。
- 循环跳转：在 block 内切分，丢弃已越过的事件，重建未来窗口。
- 播放头：`position = f(context.currentTime)`，UI 通过 `requestAnimationFrame` 读取音频线程发布的原子位置。

### 4.3 参数系统

- 参数定义：`ParamDef { id, min, max, default, curve, unit, automatable, modTargets }`。
- 更新通道：
  - **SAB 环形缓冲**（若 crossOriginIsolated）：每帧批量写入 `(paramId, value, rampFrames)`，音频线程无锁读取。
  - **MessagePort 回退**：每 128 帧批量消息；延迟略高但可用。
- 平滑：一阶低通/线性 ramp；增益/pan 必须无 zipper noise。
- 自动化：事件列表在音频线程按时间插值；支持 step/linear/curve（tension）。

### 4.4 Voice 管理

- 每个乐器预分配 voice pool（默认 64，按引擎可调）。
- Voice 状态：attack/sustain/release/idle；release tail 不占用新 voice 配额。
-  stealing 策略：最旧、最安静、最低优先级；可配置。
- 单音/连奏：Mono、Legato、Portamento time。
- 每个 voice 的 DSP 必须是预分配、无 `new`、无闭包分配。

### 4.5 采样播放

- 解码：优先 `decodeAudioData`（主线程异步）或 Worker + WASM 解码器；结果写入 `Float32Array`。
- 重采样：项目采样率与文件不同时，使用高质量 sinc/窗口重采样（离线预处理并缓存）。
- 循环：采样点循环、交叉淡入淡出、release 段。
- 切片：slice markers → 触发映射；支持 Slicer/Slicex 语义。
- 流式：大文件（>50MB）按块读取 OPFS，预取前后若干秒；避免整文件驻留。
- 缓存：按内容哈希去重；LRU 上限（默认 1GB，可配）。

### 4.6 混音与路由

- Track 节点：10 个 FX slot + 集成 EQ/相位/立体声/推子/pan。
- 路由：有向无环图；发送可带电平；侧链输出单独总线。
- PDC：
  - 每个 FX 报告 latency（采样）；
  - 计算每条路径最大 latency；
  - 在短路径插入 delay line 对齐；
  - 手动 Track Delay 叠加。
- Master 限制器/表；Insert 内部浮点不削波。
- 未使用轨：不实例化 DSP，不参与图遍历。

### 4.7 离线渲染

- `OfflineAudioContext(numberOfChannels, length, sampleRate)`。
- 加载同一 Worklet 模块，发送工程快照与范围（Song/Pattern/Selection）。
- 分块渲染（例如每 30 秒一个 OfflineAudioContext）以控制内存；块间保留 FX/voice 尾音状态（需要引擎支持状态导出/导入，或用单次长上下文 + 流式写盘）。
- 尾音模式：Leave/Wrap/Cut 由渲染长度与混音回写实现。
- 进度：按已渲染帧数上报；取消通过 `suspend`/终止 Worker 实现。

### 4.8 时钟、延迟与故障处理

- 使用 `AudioContext.currentTime` 作为唯一时钟；不使用 `Date.now()` 驱动音频。
- 记录 `baseLatency`/`outputLatency` 用于录音对齐与显示。
- 检测 dropout：连续 block 超时/欠载计数；UI 提示并建议增大缓冲。
- 设备变更：监听 `devicechange`，暂停 → 重建 AudioContext → 恢复位置（尽力）。
- 页面隐藏：允许后台播放（若浏览器允许）；否则暂停并记录位置。

---

## 5. DSP 模块与 WASM 策略

### 5.1 TypeScript DSP 原语

- 振荡器：Sine、Saw、Square、Triangle、Pulse、Noise、Band-limited wavetable。
- 滤波器：One-pole、Biquad、TPT SVF（LP/HP/BP/Notch/Peak）、Ladder、Comb。
- 动态：Envelope（AD/ADSR/多段）、Compressor、Limiter、Gate、Transient。
- 调制：LFO、Chorus/Flanger/Phaser、Ring Mod。
- 延迟：单抽头/多抽头、Ping-Pong、Diffusion、All-pass。
- 混响：Schroeder、FDN、Convolution（分区 FFT）。
- 失真：WaveShaper、Bitcrusher、Oversampling。
- 分析：FFT、Spectrum、LUFS、Peak/RMS。
- 空间：Pan、Stereo width、Mid/Side、Haas。

### 5.2 Rust/WASM 适用模块

| 模块 | 原因 |
|---|---|
| 加法/重合成（Harmor 类） | 大量正弦/FFT，JS 性能不足 |
| 卷积混响 | 分区 FFT + SIMD |
| 频谱处理/变声 | STFT、相位处理 |
| 物理建模（弦/鼓） | 高采样率递归运算 |
| 分轨/人声分离 | 模型推理（ONNX/WASM） |
| 高精度重采样 | sinc 卷积 |

### 5.3 WASM 约束

- 模块按需加载（动态 import + fetch），避免首屏过大。
- 内存上限与实例隔离；崩溃不影响主引擎（每个插件独立实例）。
- 若启用多线程：需要 COOP/COEP 响应头 + SharedArrayBuffer；否则单线程降级。
- SIMD 可选；检测 `WebAssembly.validate` 与 feature detection。

---

## 6. UI 与渲染架构

### 6.1 分层

- **Shell**：React，负责布局、面板、菜单、对话框、设置。
- **Canvas 视图**：Playlist、Piano Roll、Mixer 表、波形、自动化曲线。
- **Overlay**：选区、拖拽预览、工具提示用独立 Canvas/DOM，避免重绘全图。
- **Meters**：Canvas + rAF，绕过 React 状态，避免高频重渲染。

### 6.2 虚拟化

- Playlist：只绘制可视轨道/时间范围；轨道高度可变。
- Piano Roll：只绘制可视音高/时间范围；音符空间索引（按 tick 分桶）。
- 波形：多级 mipmap（峰值金字塔），按缩放选择层级；OffscreenCanvas 在 Worker 生成。
- 命中测试：空间索引 + 二分查找，避免遍历全部对象。

### 6.3 交互与性能

- 拖拽使用 pointer events + requestAnimationFrame 合并更新。
- 编辑命令批量提交（一次拖拽结束只发一次结构变更）。
- 高频参数（推子/自动化）直接走音频参数通道，UI 只做视觉反馈。
- 大选择集使用位图/区间表示，避免逐对象 setState。

### 6.4 主题与布局

- Design tokens（颜色、间距、字体、轨道高度）。
- FL 风格默认深色；可切换浅色/高对比度。
- 窗口布局序列化为 JSON，支持 Workspace。

---

## 7. 状态、命令与撤销

### 7.1 Store 分片

- `projectSlice`：工程实体（patterns、channels、mixer、playlist）。
- `transportSlice`：播放状态、位置、循环（部分镜像音频线程）。
- `uiSlice`：窗口、选择、缩放、工具、对话框。
- `historySlice`：撤销栈、事务。
- `assetSlice`：采样、预设、缓存状态。

### 7.2 Command Bus

- 每个编辑操作是 `Command { do, undo, label, coalesceKey }`。
- 拖拽：`beginTransaction` → 连续更新 → `commit` 合并为一条历史。
- 音频同步：Command 提交后生成 `AudioDelta`（增删改），批量发送。
- 结构变更（增删通道/轨道/插件）触发音频图重建；参数变更只发参数事件。

### 7.3 撤销策略

- 逆操作优先；复杂操作保存受影响对象的前后补丁。
- 历史上限 1,000 步；内存超限时丢弃最旧。
- 协作模式（P2）：本地撤销只回滚自己的操作（Yjs UndoManager）。

---

## 8. 存储与文件

### 8.1 IndexedDB Schema

| Store | Key | 内容 |
|---|---|---|
| `projects` | projectId | 工程元数据、缩略图、更新时间 |
| `snapshots` | snapshotId | 自动保存/手动版本（压缩 JSON） |
| `assets` | assetHash | 采样元数据 + Blob（小文件） |
| `presets` | presetId | 乐器/效果器预设 |
| `settings` | key | 用户设置、快捷键、布局 |
| `samplesIndex` | assetHash | 标签、BPM、Key、时长、授权 |

### 8.2 OPFS

- 大采样（>10MB）、导出临时文件、分轨渲染块。
- 通过 `navigator.storage.getDirectory()`；Worker 中同步 API 可用（`createSyncAccessHandle`）。

### 8.3 配额与清理

- `navigator.storage.estimate()` 显示用量；接近配额时提示。
- LRU 清理采样缓存；用户工程与用户预设不自动删除。
- 导出临时文件在成功/取消后清理。

### 8.4 文件系统

- Chrome/Edge：File System Access API 直接保存/打开。
- 其他：下载 Blob + `<input type=file>` 打开。
- 拖放导入：DataTransfer → File → 解析。

---

## 9. MIDI 与外部设备

- Web MIDI API：设备枚举、Note/CC/Pitchbend/Aftertouch、时间戳。
- 权限：首次使用时请求；被拒则降级为屏幕键盘 + MIDI 文件导入。
- MIDI Learn：CC → 参数映射；映射随工程保存。
- MIDI 输出（P2）：发送到硬件/外部软件（Web MIDI output）。
- MIDI 文件：@tonejs/midi 解析/生成；GM 通道映射；速度/拍号。
- 时钟同步：P2 支持 MIDI Clock（浏览器限制较多）。

---

## 10. 导出管线

```
OfflineAudioContext
   ├── 加载 EngineProcessor
   ├── 发送工程快照 + 渲染范围
   ├── 逐块渲染（30s/块）
   ├── 写入 AudioBuffer / OPFS 临时 WAV
   └── 编码：WAV / MP3 / FLAC / Opus / AAC
```

- WAV：自写 PCM 编码（16/24/32f），支持大文件分块写。
- MP3：lamejs（LGPL 核验）或 WASM LAME；采样率限制 32/44.1/48kHz。
- FLAC：libflacjs（许可证核验）；16/24-bit。
- Opus/AAC：WebCodecs AudioEncoder；容器封装（WebM/MP4）需自写 muxer 或库。
- 分轨：对每个 Mixer Track 单独渲染（复用同一快照，独奏该轨）。
- 校验：解码回读前 1 秒，检查时长/峰值/声道。

---

## 11. 协作架构（P2）

- 文档：Yjs `Y.Doc`，每个实体为 `Y.Map`/`Y.Array`；ID 稳定。
- 同步：WebSocket provider（自建/第三方）；离线用 y-indexeddb。
- Presence：Awareness 广播光标、选择、播放头。
- 音频：各端本地渲染；播放头同步尽力而为（网络抖动下不强制）。
- 冲突：对象级合并；结构删除需" tombstone "避免复活。
- 权限：服务端校验；只读客户端不写 doc。
- 安全：房间 token、端到端可选加密（P3）。

---

## 12. 性能预算

| 项目 | 预算 | 说明 |
|---|---|---|
| 单 voice（合成器） | ≤ 8 µs/128 帧 | 中端 CPU，64 voice ≈ 0.5ms/block |
| 单 voice（采样） | ≤ 3 µs/128 帧 | 含重采样 |
| 单 FX slot | ≤ 15 µs/128 帧 | 复杂 FX 可更高 |
| 16 轨 × 4 FX | ≤ 1.5 ms/block | 48kHz/128 帧 block = 2.67ms |
| UI 帧 | ≤ 16.7ms（60fps） | 500 轨/10k 音符 |
| 工程加载 | ≤ 3s/50MB | 采样并行解码 |
| 导出 | ≥ 2× 实时 | 离线 |
| 内存 | ≤ 1.5GB | 1 小时工程/200 采样 |
| 首屏 JS | ≤ 300KB gzip | 重型模块懒加载 |

> 音频线程每个 128 帧 block 只有 2.67ms（48kHz）；必须留 50% 余量。

---

## 13. 浏览器兼容与降级

| 能力 | Chrome/Edge | Firefox | Safari | 降级 |
|---|---|---|---|---|
| AudioWorklet | 66+ | 76+ | 14.1+ | 不支持则阻止进入 |
| OfflineAudioContext + Worklet | ✓ | ✓ | ✓ | 导出降级为实时录制（不推荐） |
| Web MIDI | ✓ | 部分/需权限 | 部分 | 屏幕键盘 + MIDI 文件 |
| SharedArrayBuffer | 需 COOP/COEP | 需 COOP/COEP | 需 COOP/COEP | MessagePort 回退 |
| WebCodecs AudioEncoder | ✓ | 部分 | 部分 | WAV/MP3 回退 |
| File System Access | ✓ | ✗ | ✗ | 下载/上传 |
| OPFS | ✓ | ✓ | 较新版本 | IndexedDB Blob |
| WebAssembly SIMD | ✓ | ✓ | ✓ | 标量回退 |
| 后台播放 | 允许（可有提示） | 允许 | 限制多 | 暂停并恢复 |

**Safari 专项**

- AudioContext 必须在用户手势后 `resume()`。
- 采样率可能与设备不一致；统一重采样到项目采样率。
- `decodeAudioData` 对某些 MP3/FLAC 兼容性差异；准备 WASM 解码回退。
- 长时间后台可能被挂起；恢复时重新校准时钟。

---

## 14. 安全与合规

- 工程文件解析：大小上限、ZIP bomb 防护、路径穿越防护、超时、schema 校验。
- 禁止 `eval`/`new Function`；Piano Roll 脚本使用受控解释器（AST 白名单）或 Web Worker 沙箱。
- WASM 插件：无网络/文件系统权限；内存/CPU 配额；签名校验。
- 内容安全策略（CSP）：限制脚本/连接源；COOP/COEP 仅在生产需要 SAB 时开启。
- 用户数据：默认本地；云同步需显式开启；导出诊断包前脱敏。
- 供应链：锁定依赖版本、SBOM、许可证扫描。

---

## 15. 可观测性与调试

- 音频故障计数：dropout、超时 block、voice 溢出、FX NaN。
- 性能面板：每轨 CPU、voice 数、FX 耗时、内存、采样缓存命中率。
- 开发者模式：显示音频图、事件队列、参数环缓冲水位。
- 错误边界：插件崩溃隔离；工程解析失败可导出诊断 JSON。
- 可选遥测：匿名性能/错误，默认关闭或明确同意。

---

## 16. 测试策略

| 层级 | 工具 | 内容 |
|---|---|---|
| 单元 | Vitest | 时间换算、TempoMap、路由、PDC、DSP 数学、schema |
| 属性测试 | fast-check | tick↔秒、循环边界、量化、区间运算 |
| 黄金音频 | 自建 harness | 固定工程渲染，比较 RMS/频谱/峰值/THD，容差阈值 |
| 集成 | Playwright | 新建→编辑→保存→重载→导出 |
| 性能 | 自建基准 | 64 voice/16 轨/500 轨 UI/10k 音符 |
| 兼容 | BrowserStack/本地 | Chrome/Edge/Firefox/Safari |
| 安全 | fuzz | 工程 ZIP/JSON/MIDI/SF2 解析 |
| 可访问性 | axe | 核心流程 |

**黄金音频示例**

- 1kHz 正弦 → 增益 → 导出，检查频率/幅度。
- 脉冲 → 混响，检查 RT60 与能量衰减。
- 锯齿 → 低通扫频，检查混叠/截止。
- 侧链 → 压缩，检查增益包络。

---

## 17. 架构决策记录（ADR）

| 编号 | 决策 | 状态 |
|---|---|---|
| ADR-01 | 单 AudioWorklet DSP Graph | 已定 |
| ADR-02 | TS DSP + Rust/WASM 重模块 | 已定 |
| ADR-03 | Canvas2D 自绘 + 虚拟化 | 已定 |
| ADR-04 | Zustand + Command Bus 撤销 | 已定 |
| ADR-05 | ZIP + JSON 工程格式 | 已定 |
| ADR-06 | 本地优先存储，云后置 | 已定 |
| ADR-07 | Yjs 协作（P2） | 待验证 |
| ADR-08 | 不宿主 VST/AU | 已定 |
| ADR-09 | MP3 编码方案待法务确认 | 待定 |
| ADR-10 | SharedArrayBuffer 可选 | 已定 |

---

## 18. 技术风险与缓解

| 风险 | 影响 | 缓解 |
|---|---|---|
| 音频线程 CPU 超预算 | dropout | voice 上限、FX 简化模式、WASM、性能面板 |
| Safari 兼容差异 | 用户流失 | 专项测试、重采样、手势恢复、解码回退 |
| 大采样内存 | 崩溃 | 流式、LRU、配额提示 |
| 离线渲染内存 | 导出失败 | 分块渲染、OPFS 临时文件 |
| FLP 解析复杂 | 导入失败 | 尽力导入、明确提示、保留原文件 |
| 协作冲突 | 数据损坏 | 对象级 CRDT、tombstone、版本回滚 |
| 许可证问题 | 下架风险 | 授权清单、法务评审、替代素材 |
| 范围蔓延 | 延期 | P0/P1/P2 冻结、阶段验收 |