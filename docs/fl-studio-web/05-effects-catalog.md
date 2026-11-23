# 05 · 效果器目录（Effects Catalog）

> 目标：**覆盖 FL Studio 全部原生效果器的等价能力（67/67）**，并扩展到 **93 个效果器定义**（含 10 个"旗舰"级）。
> 每个效果器必须支持：参数自动化、预设、湿/干比、旁通、延迟上报（PDC）、立体声/侧链（适用时）。

---

## 1. 效果器框架

### 1.1 插槽模型

- 每 Mixer Track **10 个 FX Slot**；Slot 可插入任意效果器。
- 每个 Slot：旁通、湿/干、增益、延迟（自动上报）、独奏（调试）。
- FX 链可保存为预设；整轨可保存为"Track State"。
- Patcher 可作为"容器 Slot"，内部再嵌套任意节点图。

### 1.2 处理约定

| 项 | 约定 |
|---|---|
| 采样率 | 跟随工程；内部可按需 2×/4× 过采样 |
| 缓冲 | 128 帧量子；效果器必须支持任意块长（含 1 帧） |
| 延迟 | 必须上报 latency（采样），供 PDC 对齐 |
| 侧链 | 通过独立侧链总线输入，不占用主输入 |
| 参数 | 全部可自动化；参数变化必须平滑 |
| 状态 | 支持 `serialize/deserialize`，用于撤销/导出/协作 |
| CPU | 标注 Light/Standard/Heavy；Heavy 效果器在低配设备可降级 |

### 1.3 效果器通用参数

- Input Gain、Output Gain、Mix（Dry/Wet）、Bypass。
- 立体声模式：Stereo / Mid-Side / L-only / R-only / Mono。
- 可选： oversampling、quality（Draft/Good/Best）。

---

## 2. 效果器总表（93 项）

### 2.1 EQ 与滤波（EQ & Filter）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Parametric EQ 8 | Fruity Parametric EQ 2 | 8 段、频点/Q/增益、频谱显示、相位模式 | Light | P0 |
| Parametric EQ 6 | Fruity Parametric EQ | 6 段、简易版 | Light | P1 |
| 7-Band EQ | Fruity 7 Band EQ | 7 段固定频段、增益 | Light | P1 |
| Free Filter | Fruity Free Filter | 低通/高通/带通、截止/共振 | Light | P1 |
| Fast LP | Fruity Fast LP | 极简低通、低 CPU | Light | P1 |
| Multi-Mode Filter | Fruity Filter | LP/HP/BP/Notch、双滤波 | Light | P1 |
| Love Philter | Fruity Love Philter | 多段滤波、包络/LFO、每段独立 | Standard | P1 |
| Frequency Splitter | Frequency Splitter | 分频段独立处理/路由 | Light | P1 |
| EQUO | EQUO | 动态 EQ/多段塑形 | Standard | P1 |
| Emphasis | Emphasis | 高频/存在感塑形 | Light | P2 |
| Emphasizer | Emphasizer | 频段强调/动态 | Light | P2 |
| Low Lifter | Low Lifter | 低频提升/谐波 | Light | P2 |
| Bass Boost | Fruity Bass Boost | 低频增强/谐波激励 | Light | P2 |
| Spectral Shaper | （新增） | 频谱曲线塑形、共振峰 | Heavy | P2 |
| Dynamic EQ | （新增） | 频段随电平动态变化 | Standard | P2 |

### 2.2 动态处理（Dynamics）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Compressor | Fruity Compressor | Threshold/Ratio/Attack/Release/Knee、侧链 | Light | P0 |
| Multiband Compressor | Fruity Multiband Compressor | 3–5 段、每段独立、可视化 | Standard | P1 |
| Limiter | Fruity Limiter | 砖墙限制、噪声门、压缩模式、LUFS | Light | P0 |
| Maximizer | Maximus | 多段压缩/限制/扩展、曲线编辑 | Heavy | P1 |
| Soft Clipper | Fruity Soft Clipper | 软削波、阈值/硬度 | Light | P1 |
| Transient Processor | Transient Processor | Attack/Sustain 分离、独立增益 | Light | P1 |
| Transient Splitter | Transmitter | 瞬态/持续分离、独立路由 | Standard | P1 |
| Squeeze | Fruity Squeeze | 位深/降采样/压缩混合 | Light | P1 |
| Soundgoodizer | Soundgoodizer | 一键式多段增强（A/B/C/D） | Light | P1 |
| Gate | （新增） | 噪声门、侧链、hold/hysteresis | Light | P1 |
| Expander | （新增） | 向下扩展、门限 | Light | P2 |
| De-esser | （新增） | 齿音抑制 | Light | P1 |
| Upward Compressor | （新增） | 向上压缩/弱音增强 | Standard | P2 |

### 2.3 延迟与时间（Delay & Time）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Delay | Fruity Delay | 单抽头、反馈、滤波、ping-pong | Light | P0 |
| Delay 2 | Fruity Delay 2 | 双抽头、调制、扩散 | Light | P1 |
| Delay 3 | Fruity Delay 3 | 多抽头、时间/音高、滤波 | Standard | P1 |
| Delay Bank | Fruity Delay Bank | 多段延迟、每段独立 | Standard | P1 |
| Multiband Delay | Multiband Delay | 分频段延迟 | Standard | P1 |
| Time Warp | Gross Beat | 时间/音高曲线、scratch、stutter | Heavy | P1 |
| Pitch Shifter | Pitch Shifter | 移调、formant、延迟 | Standard | P1 |
| Frequency Shifter | Frequency Shifter | 频率搬移、环形调制 | Light | P2 |
| Echo/Reverb | （新增） | 延迟+混响混合单元 | Light | P2 |

### 2.4 混响（Reverb）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Reverb | Fruity Reeverb | 算法混响、房间/阻尼/湿干 | Light | P0 |
| Reverb 2 | Fruity Reeverb 2 | 更高质量、早期反射、调制 | Standard | P0 |
| Convolver | Fruity Convolver | 卷积、IR 加载、分区 FFT、EQ | Heavy | P1 |
| LuxeVerb | LuxeVerb | 高品质算法混响、调制/扩散 | Heavy | P1 |
| Plate Reverb | （新增） | 板式混响、金属感 | Standard | P2 |
| Spring Reverb | （新增） | 弹簧混响、吉他风格 | Standard | P2 |
| Shimmer Reverb | （新增） | 八度移调反馈混响 | Heavy | P2 |

### 2.5 调制与空间（Modulation & Spatial）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Chorus | Fruity Chorus | 多声部合唱、深度/速率 | Light | P0 |
| Vintage Chorus | Vintage Chorus | 复古 BBD 风格 | Light | P1 |
| Hyper Chorus | Hyper Chorus | 大规模调制、维度 | Standard | P1 |
| Flanger | Fruity Flanger | 镶边、反馈、立体声 | Light | P0 |
| Flangus | Fruity Flangus | 镶边+合唱混合 | Light | P1 |
| Phaser | Fruity Phaser | 多级全通、反馈 | Light | P0 |
| Vintage Phaser | Vintage Phaser | 复古相位 | Light | P1 |
| Stereo Enhancer | Fruity Stereo Enhancer | 宽度/相位/立体声增强 | Light | P1 |
| Stereo Shaper | Fruity Stereo Shaper | 左右声道塑形/延迟 | Light | P1 |
| Spreader | Spreader | 立体声扩散 | Light | P1 |
| Center | Fruity Center | 中置/声像处理 | Light | P2 |
| PanOMatic | Fruity PanOMatic | 自动声像/LFO | Light | P2 |
| Vibrator | Fruity Vibrator | 多 LFO 调制（复刻为软件 LFO） | Light | P2 |
| Vocoder | Fruity Vocoder | 通道声码器、频带数 | Standard | P1 |
| Vocodex | Vocodex | 高级声码器、共振峰/调制 | Heavy | P1 |
| Pitcher | Pitcher | 实时移调/和声/自动调 | Standard | P1 |
| Harmonizer | （新增） | 和声生成、音阶约束 | Standard | P2 |

### 2.6 失真与饱和（Distortion & Saturation）

| 本项目效果器 | FL 等价 | 核心参数/能力 | CPU | 优先级 |
|---|---|---|---|---|
| Distructor | Distructor | 多段失真/滤波/调制组合 | Standard | P1 |
| Blood Overdrive | Fruity Blood Overdrive | 过载、低频驱动 | Light | P1 |
| Fast Dist | Fruity Fast Dist | 极简失真、低 CPU | Light | P1 |
| Hardcore | Hardcore | 吉他效果链（失真/箱体/延迟/压缩） | Standard | P1 |
| WaveShaper | Fruity WaveShaper | 自定义波形映射、包络 | Light | P1 |
| Soft Clipper | Fruity Soft Clipper | 软削波（同动态组） | Light | P1 |
| Bitcrusher | Fruity Squeeze（位深部分） | 位深/降采样 | Light | P1 |
| Tape Saturator | （新增） | 磁带饱和/抖动/哇音 | Light | P2 |
| Tube Saturator | （新增） | 电子管饱和、偏置 | Light | P2 |
| Multi-band Distortion | （新增） | 分频段失真 | Standard | P2 |

### 2.7 工具、分析与视觉（Utility & Visual）

| 本项目 | FL 等价 | 说明 | 优先级 |
|---|---|---|---|
| Send | Fruity Send | 发送/路由辅助、增益 | P0 |
| Balance | Fruity Balance | 增益/声像/静音 | P0 |
| Mute 2 | Fruity Mute 2 | 平滑静音/淡入淡出 | P1 |
| Phase Inverter | Fruity Phase Inverter | 相位反转 | P1 |
| Tuner | Tuner | 音准表 | P2 |
| Spectrum Analyzer | Fruity Spectroman | 频谱显示 | P1 |
| Oscilloscope | Wave Candy | 波形/频谱/矢量示波器 | P1 |
| dB Meter | Fruity dB Meter | 峰值/RMS 表 | P1 |
| Big Clock | Fruity Big Clock | 时间/小节显示 | P2 |
| Transporter | Transporter | 走带/定位辅助 | P2 |
| NoteBook | Fruity NoteBook | 工程内文本笔记 | P2 |
| HTML NoteBook | Fruity HTML NoteBook | 富文本笔记（沙箱） | P2 |
| Patcher | Patcher | 模块化容器 | P1 |
| Effector | Effector | 多效果 XY 表演控制器 | P1 |
| Scratcher | Fruity Scratcher | 黑胶刮擦模拟 | P2 |
| Video Player | Fruity Video Player | 视频参考 | P2 |
| Visualizer | ZGameEditor Visualizer | 音频可视化（可选导出） | P2 |
| Audio Editor | Edison | 录音/波形编辑（独立工具） | P1 |
| Pitch Editor | NewTone | 音高校正 | P2 |
| Time Editor | NewTime | 时间伸缩 | P2 |
| EQ Tool | Audio Editor Equalizer | 编辑器内 EQ | P2 |
| Reverb Tool | Audio Editor Convolution | 编辑器内卷积混响 | P2 |

**统计**：EQ/滤波 15、动态 13、延迟/时间 9、混响 7、调制/空间 17、失真/饱和 10、工具/分析/视觉 22，合计 **93 项**（P0 11 / P1 50 / P2 32）。

> 其中 67 项与 FL 原生效果器一一对应（覆盖全部原生效果器，核对表见 `04-instruments-catalog.md` 附录 A），其余为等价体系新增（频谱塑形、动态 EQ、双耳渲染等）。满足"≥50 个可用效果器、≥10 个旗舰级"目标。

---

## 3. 旗舰效果器规格（重点实现）

### 3.1 Parametric EQ 8（P0）

- 8 段：Bell/Shelf/LowCut/HighCut/Notch/BandPass/AllPass。
- 每段：频率（20Hz–20kHz 对数）、Q（0.1–10）、增益（-24–+24dB）、立体声/中侧。
- 显示：实时频谱、EQ 曲线、节点拖拽、A/B 对比、预设。
- DSP：级联 TPT SVF/双二阶，浮点稳定；频率自动化平滑。
- 延迟：0 采样（最小相位）；可选线性相位模式（高延迟，P2）。

### 3.2 Limiter（P0）

- 砖墙限制：Threshold、Ceiling、Release、Knee、Lookahead。
- 噪声门：Threshold、Ratio、Attack、Hold、Release。
- 模式：Limiter / Compressor / Gate。
- Master 默认链路：EQ → Compressor → Limiter。
- 输出：峰值/RMS/LUFS 表；削波计数。

### 3.3 Reverb 2（P0）

- 算法：FDN/早期反射 + 阻尼/扩散/调制。
- 参数：Room Size、Decay、Damping、Diffusion、Pre-delay、Early/Late、Width、Mix。
- 支持侧链？不需要；支持自动化。
- 立体声：True Stereo；CPU Standard。

### 3.4 Convolver（P1）

- 分区 FFT 卷积（uniform partition，FFT 大小可调）。
- IR 加载：WAV/AIFF；单/双声道；长度上限（如 30s）。
- 参数：IR 选择、增益、预延迟、干湿、EQ、立体声宽度。
- IR 库：自生成/CC0；不使用 FL 官方 IR。

### 3.5 Maximus（P1）

- 3 段（Low/Mid/High）压缩/限制/扩展，曲线可拖拽。
- 每段：输入/输出增益、Attack/Release、Ratio、Knee。
- 侧链、Master 限制、LUFS 目标。
- 可视化：输入/输出曲线、频谱、增益衰减。

### 3.6 Gross Beat 类 Time Warp（P1）

- 时间曲线 + 音量曲线，网格 1/4–1/32。
- 模式：Stutter、Scratch、Reverse、Gate、Pitch。
- 实时表演：可映射到控制器；可录制为自动化。
- DSP：环形缓冲 + 变速/变调（相位声码器或 WSOLA）。

### 3.7 Vocodex 类声码器（P1）

- 频带数（8–64）、载波/调制输入、共振峰、包络跟随。
- 侧链输入作为调制源；载波可用内置噪声/合成器。
- 立体声/单声道模式；CPU Heavy。

### 3.8 Effector（P1）

- XY 面板：4–8 个效果模块（滤波、延迟、失真、混响、门限等）。
- 每个模块可开关/量控；XY 可自动化/录制。
- 用于现场表演与快速音色变化。

---

## 4. 效果链与预设

### 4.1 默认 Master 链

```
Input → Parametric EQ 8 → Compressor → Limiter → Output
```

### 4.2 推荐链路模板

| 场景 | 链路 |
|---|---|
| 鼓组总线 | EQ → Compressor → Transient → Soft Clipper → Reverb(轻) |
| 贝斯 | EQ → Saturator → Compressor → Limiter |
| 吉他 | EQ → Distortion → Cabinet → Delay → Reverb |
| 人声 | De-esser → EQ → Compressor → Vocoder(可选) → Reverb |
| Pad | EQ → Chorus → Delay → Reverb → Stereo Enhancer |
| 母带 | EQ → Multiband Comp → Maximizer → Limiter → Meter |

### 4.3 预设要求

- 每个效果器 ≥8 个工厂预设（旗舰 ≥20）。
- 预设命名不使用 FL 官方名称；按用途（Vocal/Bass/Drum/Master）。
- 预设可保存参数 + 侧链设置 + 过采样设置。

---

## 5. 实现优先级

### M1（MVP）

1. Parametric EQ 8
2. Compressor
3. Limiter
4. Reverb 2
5. Delay
6. Chorus / Flanger / Phaser（基础三件套）
7. Send / Balance
8. Spectrum Analyzer / dB Meter

### M2（v0.5）

- Multiband Compressor、Maximus、Soft Clipper、Transient Processor、Gate、De-esser。
- Delay 2/3、Multiband Delay、Convolver、LuxeVerb。
- Vintage Chorus、Hyper Chorus、Flangus、Vintage Phaser、Stereo Enhancer、Spreader。
- Distructor、Blood Overdrive、Fast Dist、Hardcore、WaveShaper、Bitcrusher。
- Vocoder、Pitcher、Time Warp。
- Patcher、Effector、Tuner、Oscilloscope。

### M3（v1.0）

- Love Philter、EQUO、Frequency Splitter、Dynamic EQ。
- Delay Bank、Pitch Shifter、Frequency Shifter。
- Plate/Spring/Shimmer Reverb。
- Vocodex、Harmonizer、Tape/Tube Saturator、Multi-band Distortion。
- 全部工具/视觉类。

### M4（v1.x）

- 线性相位 EQ、频谱塑形、AI 辅助母带、空间音频（Ambisonics/HRTF）。

---

## 6. 质量与合规

- **IR/预设不得来自 FL Studio 安装目录**；使用自生成 IR 或 CC0 IR。
- 效果器名称使用自有命名（如 "Studio EQ 8" 而非 "Fruity Parametric EQ 2"）。
- 所有 DSP 需通过稳定性测试：极值参数、NaN/Inf、采样率切换、块长 1、长时间运行。
- 每个效果器提供 CPU 基准与"低配降级"策略（如降低过采样、简化扩散）。
- 侧链/延迟上报必须正确，否则 PDC 会导致相位问题。