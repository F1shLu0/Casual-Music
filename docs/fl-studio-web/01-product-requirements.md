# 01 · 产品需求文档（PRD）

> 产品代号：**OpenBeat Studio（暂定）**
> 版本：v0.1（需求分析稿，未进入实现）
> 关联文档：`02-feature-specification.md`、`03-audio-and-technical-architecture.md`

---

## 1. 背景

FL Studio 是 Image-Line 的桌面 DAW，以 Channel Rack + Pattern + Playlist 的非线性工作流、业内公认最强的 Piano Roll、丰富的原生乐器/效果器著称。官方在 2025–2026 年推出了 **FL Studio Web Beta**，但定位是"面向初学者的简化版"：

- 需要申请 waitlist，访问受限；
- 仅包含"essential workflows and tools, simplified"；
- 支持 Web → Desktop 单向打开工程，**不支持 Desktop → Web**；
- 依赖 FL Cloud 的 2M+ 采样库；
- 官方说明"best experienced on a computer"、"works best in Google Chrome"。

这意味着：**"完整版 FL Studio 工作流"在 Web 端仍是空白**。本项目的目标就是填补这个空白，做一个功能密度接近桌面版、乐器与效果器尽可能全面的在线 DAW。

---

## 2. 产品目标

### 2.1 总目标

在浏览器中提供一套**可长期创作完整歌曲**的 DAW，而不是"网页节拍器"：

1. 工作流与 FL Studio 高度同构：Channel Rack 为中心、Pattern 与 Playlist 解耦、Piano Roll 强大、Mixer 灵活路由。
2. 乐器"尽可能全面、种类多"：覆盖合成器、采样器、鼓机、管弦、世界乐器、GM 全音色。
3. 工程可保存、可迁移、可导出、可恢复；不锁死用户。
4. 在主流笔记本上做到可用延迟与稳定播放；离线渲染快于实时。

### 2.2 分目标（可度量）

| 编号 | 目标 | 度量 |
|---|---|---|
| G-01 | 工作流同构 | P0 功能覆盖 FL 桌面版核心窗口的 90% 以上操作语义 |
| G-02 | 乐器广度 | 1.0 提供 ≥45 个乐器引擎/等价插件；≥128 GM 音色；≥20 个鼓组 |
| G-03 | 效果器广度 | 1.0 提供 ≥50 个效果器，覆盖 EQ/动态/延迟/混响/调制/失真/滤波/工具 |
| G-04 | 可用性 | 中端笔记本（i5/16GB/集成显卡）64 复音、16 轨、48kHz 稳定播放 |
| G-05 | 响应 | UI 在 500 轨 + 10,000 音符下保持 55–60fps |
| G-06 | 导出 | 5 分钟工程离线渲染 ≤ 2 分钟（含混响尾音） |
| G-07 | 可靠性 | 崩溃后自动恢复最近 30 秒内的编辑；工程保存成功率 ≥99.9% |
| G-08 | 兼容性 | Chrome/Edge/Firefox/Safari 17+ 可完成"新建→编曲→导出"全流程 |

### 2.3 非目标（1.0 不做）

- 原生 VST/AU 宿主、ASIO、ReWire、Buzz；
- 与 FL Studio 像素级 UI/皮肤/图标一致；
- 官方 FL Cloud、Image-Line 账号、FLP 100% 双向兼容；
- 桌面级视频剪辑；只做视频同步参考（P2）。

---

## 3. 用户与场景

### 3.1 用户画像

| 画像 | 特征 | 核心诉求 | 关键功能 |
|---|---|---|---|
| P1 FL 老用户 | 熟悉 FL 快捷键与工作流，有旧工程 | 不想装电脑也能继续做；工程能带走 | 快捷键、Piano Roll、FLP 导入、Mixer 路由、VST 替代 |
| P2 卧室制作人 | 16–30 岁，Chromebook/轻薄本，预算有限 | 开箱即用、音色多、能导出 | 预设库、Step Sequencer、Loop、模板、MP3 导出 |
| P3 初学者 | 第一次做音乐 | 引导、即时发声、少概念 | 教学工程、工具提示、简化模式、Chord Stamp |
| P4 教师/课堂 | 机房/平板，无法装软件 | 零安装、统一分发、可控 | 分享链接、只读演示、课堂模板 |
| P5 协作团队 | 远程合作 | 同一工程实时编辑/评论 | 协作房间、版本历史、分轨导出 |

### 3.2 核心用户故事

1. 作为 FL 老用户，我希望网页版的 Channel Rack/Piano Roll/Playlist 操作逻辑与桌面版一致，这样我不需要重新学习。
2. 作为初学者，我希望 3 分钟内做出 4 小节鼓点并导出，这样我能获得成就感。
3. 作为制作人，我希望有足够多的合成器、鼓组、管弦与世界乐器，不用到处找音源。
4. 作为协作方，我希望两个人能同时编辑同一工程而不互相覆盖。
5. 作为教师，我希望学生打开链接就能试听和编辑，不需要安装几百 MB 的软件。

---

## 4. 竞品与基准

| 产品 | 形态 | 优势 | 不足 | 对本项目的启示 |
|---|---|---|---|---|
| FL Studio Desktop | 原生 DAW | 工作流独特、Piano Roll 强、插件多 | 不能浏览器直接用；正版付费 | 功能基准与交互语义来源 |
| FL Studio Web (官方 Beta) | 浏览器简化 DAW | 官方品牌、FL Cloud、教学 | 功能简化、waitlist、Desktop→Web 不支持 | 证明 Web DAW 可行；差异化做"完整版" |
| openDAW | 开源 Web DAW | 现代引擎、开源 | 生态与插件少 | 架构参考 |
| WAM-Studio | Web Audio 插件生态研究 | WAM 插件协议 | 产品化不足 | 插件 SDK 参考 |
| Soundtrap/BandLab | 在线 DAW | 协作、易用 | 偏录音/流行，缺 FL 式 Pattern 工作流 | 协作与引导参考 |
| Ableton Live (桌面) | 原生 DAW | Session View、效果器强 | 非 Web | 不做对标，避免混淆 |

**结论**：不与官方 FL Studio Web 正面比拼"简化入门"，而是以"**浏览器里的完整 FL 工作流 + 最全乐器库**"为定位。

---

## 5. 功能范围与优先级

优先级定义：

- **P0**：MVP 必须，缺失则不能称为 FL-like DAW；
- **P1**：完整度主体，1.0 目标；
- **P2**：差异化/增强，1.x；
- **Out**：明确不做或长期不做。

### 5.1 功能矩阵

| 模块 | 能力 | FL Desktop | 官方 FL Web | 本项目 |
|---|---|---|---|---|
| 平台 | 浏览器零安装 | ✗ | ✓ | ✓ P0 |
| 平台 | 离线可用（PWA） | ✓ | 部分 | ✓ P1 |
| 平台 | 移动/平板 | App | 可打开但体验差 | 平板布局 P2 |
| 走带 | Song/Pattern 模式 | ✓ | 简化 | ✓ P0 |
| 走带 | 速度/拍号/摇摆/半速/Tap | ✓ | 部分 | ✓ P0 |
| 通道机架 | Channel Rack | ✓ | ✓ | ✓ P0 |
| 通道机架 | Step Sequencer 512 步 | ✓ | ✓（简化） | ✓ P0 |
| 通道机架 | 通道分组/过滤/排序/克隆 | ✓ | 部分 | ✓ P1 |
| 通道机架 | Layer/层叠通道 | ✓ | ? | ✓ P1 |
| 钢琴卷帘 | 音符编辑/属性/事件编辑器 | ✓ | 简化 | ✓ P0 |
| 钢琴卷帘 | Slide/Portamento | ✓ | ? | ✓ P0 |
| 钢琴卷帘 | Ghost Notes/音符分组/量化 | ✓ | ? | ✓ P1 |
| 钢琴卷帘 | Chord Stamp/Scale Snap | ✓ | ? | ✓ P1 |
| 钢琴卷帘 | Riff Machine/脚本 | ✓ | ✗ | ✓ P2 |
| 播放列表 | Pattern/Audio/Automation Clip | ✓ | 简化 | ✓ P0 |
| 播放列表 | 多 Arrangement | ✓ | ✗ | ✓ P1 |
| 播放列表 | Performance Mode | ✓ | ✗ | ✓ P1 |
| 播放列表 | 500 轨/剪辑增益/淡入淡出/切片 | ✓ | 部分 | ✓ P0/P1 |
| 混音器 | 500 Insert + Current + Master | ✓ | 简化 | ✓ P0（逻辑上限，UI 虚拟化） |
| 混音器 | 10 FX Slots/发送/侧链/PDC | ✓ | 部分 | ✓ P0/P1 |
| 混音器 | 分轨导出 | ✓ | ? | ✓ P1 |
| 自动化 | Automation Clip/曲线/Step/Slide | ✓ | 简化 | ✓ P0 |
| 自动化 | Event Automation/控制器/LFO | ✓ | ? | ✓ P1 |
| 自动化 | 参数链接/Macro/Formula | ✓ | ? | ✓ P1 |
| 浏览器 | 工程内容/采样/预设/搜索/试听 | ✓ | ✓（FL Cloud） | ✓ P0 |
| 浏览器 | 标签/收藏/相似音色/缺失重链 | ✓ | 部分 | ✓ P1 |
| 乐器 | 原生合成器/采样器/鼓机 | 54 Generator | 少量 | 54 项等价 P1 |
| 乐器 | GM 全音色/管弦/世界乐器 | 需第三方 | FL Cloud 采样 | ✓ 扩展库 P1 |
| 效果器 | 原生效果器 | 67 | 少量 | 67 项等价 + 扩展共 93 P1 |
| 录音 | 音频输入/MIDI 录制/接管 | ✓ | ✗ | ✓ P1 |
| 编辑 | Edison/NewTone/NewTime/Slicer | ✓ | 部分（Slicer 2） | ✓ P1/P2 |
| 插件 | VST/AU | ✓ | ✗ | ✗（WAM/WASM P2） |
| 工程 | 保存/自动保存/撤销/版本 | ✓ | ✓ | ✓ P0 |
| 工程 | FLP 导入 | ✓ | ✗ | 尽力 P1 |
| 工程 | 云备份 | FL Cloud | ✓ | P2 |
| 协作 | 实时协作 | ✗ | ✗ | P2 |
| 导出 | WAV/MP3/FLAC/OGG/M4A/MIDI | ✓ | 导出 | ✓ P0/P1 |
| AI | Gopher 助手/脚本生成 | ✓（2026） | ? | P2 |
| 教学 | 交互式课程 | ✗ | ✓ | P1 |

### 5.2 版本节奏建议

- **MVP（v0.1）**：P0 的 70%，可做 4–8 小节 beat 并导出 WAV。
- **v0.5**：P0 全量 + P1 的音频录制、Piano Roll 高级工具、20+ 乐器、25+ 效果器。
- **v1.0**：P1 全量（54 个 Generator 等价、93 个效果器、FLP 导入、分轨导出、GM 128）。
- **v1.x**：P2（协作、云、AI、移动、插件 SDK）。

---

## 6. 非功能需求（NFR）

| 编号 | 类别 | 要求 | 验证方式 |
|---|---|---|---|
| NFR-01 | 延迟 | 48kHz、128 帧缓冲下，往返延迟目标 ≤ 30ms；256 帧 ≤ 45ms | 实测 RTL、loopback |
| NFR-02 | 稳定性 | 30 分钟连续播放无 dropout；UI 线程阻塞不影响音频 | soak test |
| NFR-03 | CPU | 64 复音 + 16 轨 + 8 效果器，中端 i5 不超过单核 60% | 性能基准工程 |
| NFR-04 | 内存 | 1 小时工程 + 200 个采样，常驻内存 ≤ 1.5GB；采样按需流式 | 内存快照 |
| NFR-05 | UI | 500 轨/10k 音符/1000 剪辑下 55–60fps | Chrome DevTools |
| NFR-06 | 启动 | 冷启动到可播放 ≤ 3s（不含采样解码） | 计时 |
| NFR-07 | 工程加载 | 50MB 工程 ≤ 3s；采样并行解码 | 计时 |
| NFR-08 | 导出 | 离线渲染 ≥ 2× 实时；可取消；进度可见 | 基准 |
| NFR-09 | 兼容 | Chrome/Edge 120+、Firefox 120+、Safari 17+；AudioWorklet 必须 | 浏览器矩阵 |
| NFR-10 | 可访问性 | 键盘可完成核心流程；对比度 WCAG AA；可缩放 200% | axe + 人工 |
| NFR-11 | 数据安全 | 工程文件校验、大小上限、解析超时、无 eval | 模糊测试 |
| NFR-12 | 隐私 | 默认本地存储；不上传音频/工程除非用户显式开启云 | 代码审计 |
| NFR-13 | 可维护性 | 模块边界清晰；DSP 单测覆盖 ≥70%；黄金音频回归 | CI |
| NFR-14 | 国际化 | 中/英双语，文案可扩展；RTL 预留 | 抽查 |
| NFR-15 | 可恢复 | IndexedDB 自动保存 ≤30s；崩溃恢复提示 | 故障注入 |

---

## 7. 成功指标

| 指标 | 目标（上线 3 个月） |
|---|---|
| 首次发声时间（TTFS） | 中位数 < 60 秒 |
| 首个 8 小节工程完成率 | ≥ 40% |
| 7 日留存 | ≥ 25% |
| 导出成功率 | ≥ 99% |
| 音频故障率（dropout/崩溃） | < 0.5% 会话 |
| 平均会话时长 | ≥ 20 分钟 |
| NPS | ≥ 30 |

---

## 8. 验收标准（Epic 级）

### AC-01 新建与发声

- Given 用户首次打开应用，When 点击"新建工程"并选择一个鼓组通道，Then 3 次点击内可触发第一次发声，且无需授权麦克风。
- Given 用户没有音频输出设备，When 开始播放，Then 显示可操作的错误提示并提供重试，不崩溃。

### AC-02 步进音序

- Given 一个 16 步 Pattern，When 在 Kick 通道点亮 1/5/9/13 步，Then 播放时严格按 4/4 网格触发，速度改变后仍对齐。
- When 将 Pattern 长度改为 32 步，Then 网格扩展且已有步进数据不丢失。

### AC-03 钢琴卷帘

- Given 一个 MIDI 键盘，When 录制 8 小节旋律，Then 音符按当前 Snap 与量化设置落位，可撤销。
- When 给两个相邻音符添加 Portamento 标记，Then 播放时音高连续滑变，且不影响其他音符。
- When 拖动一个音符的 Velocity，Then 仅该音符力度变化，事件编辑器同步显示。

### AC-04 播放列表

- Given 两个 Pattern Clip 与一个 Audio Clip，When 拖动到不同轨道并重叠，Then 音频按轨道叠加播放，剪辑可独立静音/移动/裁剪。
- When 切换到第二个 Arrangement，Then 第一个 Arrangement 的剪辑保持不变，Channel Rack/Mixer 共享。

### AC-05 混音器

- Given 通道 A 路由到 Insert 1、通道 B 路由到 Insert 2，When 将 Insert 1 Send 到 Insert 2，Then Insert 2 同时收到 A 的发送信号与 B 的直连信号，推子独立可调。
- When 在 Master 插入限制器并把音量推高，Then Master 峰值不超过 0dBFS，且不产生明显失真。

### AC-06 自动化

- Given 一个 Automation Clip 链接到滤波器 Cutoff，When 在 4 小节内画一条从 20Hz 到 20kHz 的曲线，Then 播放时参数按曲线变化，导出音频包含同样变化。
- When 使用 Step 模式绘制自动化，Then 每个网格步生成一个控制点，曲线呈阶梯状。

### AC-07 工程持久化

- Given 用户编辑后 30 秒内关闭浏览器，When 重新打开，Then 提示恢复自动保存版本，恢复后音符/剪辑/混音状态一致。
- When 保存工程并刷新页面后重新打开，Then 所有引用采样可播放，缺失采样有明确重链入口。

### AC-08 导出

- Given 一个 3 分钟工程，When 选择 WAV 24-bit/48kHz 导出，Then 生成文件可在第三方播放器播放，时长与尾音符合设置。
- When 选择分轨导出，Then 每个 Mixer Track 生成独立文件，且文件名与轨道名对应。
- When 导出过程中取消，Then 任务终止、临时文件清理、UI 恢复可用。

### AC-09 MIDI

- Given 一个标准 MIDI 文件，When 导入，Then 按轨道/通道映射为 Pattern 与乐器通道，速度与拍号正确。
- When 导出 MIDI，Then 主流 DAW 可识别音符、速度与轨道名。

### AC-10 性能

- Given 500 轨 Playlist + 10,000 音符，When 缩放/滚动/拖拽，Then 帧率 ≥55fps，无明显卡顿。
- Given 64 复音持续演奏 10 分钟，When 观察音频，Then 无 dropout、无爆音、CPU 稳定。

### AC-11 兼容性

- Given Chrome、Edge、Firefox、Safari 最新稳定版，When 执行"新建→编曲→导出"脚本，Then 全流程通过；Safari 需用户手势后恢复 AudioContext。

### AC-12 合规

- Given 发布包，When 审计，Then 不含 FL Studio 商标、官方图标、官方采样/预设；所有第三方采样有授权记录；MP3 编码方案符合许可证。

---

## 9. 假设、依赖与约束

### 假设

- 用户设备支持 Web Audio AudioWorklet（2026 年主流浏览器均支持）。
- 大部分用户使用 Chrome/Edge；Safari 作为兼容目标而非性能基准。
- 1.0 不要求原生插件，用户可接受内置等价乐器/效果器。
- 采样库以 CC0/CC BY/明确商用许可为主，避免版权风险。

### 依赖

- 采样库授权（GeneralUser GS、VSCO 2 CE、Salamander、FluidR3/Sonatina 核验；清单见 `07-roadmap-risks-qa-compliance.md` 附录 A）。
- MP3/FLAC 编码库许可证（lamejs LGPL、libflacjs 核验）。
- 浏览器能力：AudioWorklet、OfflineAudioContext、Web MIDI、IndexedDB/OPFS、WebCodecs（可选）。
- 设计与法务：品牌命名、UI 原创性审查。

### 约束

- 浏览器单线程 UI 与音频线程隔离；不能依赖原生插件。
- 移动端内存/发热限制；大采样必须流式。
- 无法保证 FLP 100% 兼容；需向用户明确"尽力导入"。

---

## 10. 术语表

| 术语 | 含义 |
|---|---|
| Channel Rack | 通道机架，承载乐器/采样器/自动化通道 |
| Pattern | 一个或多个通道的音符/步进/自动化数据集合，最多 999 个 |
| Playlist | 编排 Pattern/Audio/Automation Clip 的时间线，最多 500 轨 |
| Piano Roll | 钢琴卷帘，音符与事件编辑 |
| Automation Clip | 播放列表中的参数曲线剪辑 |
| Insert Track | 混音器插入轨，最多 500 条 |
| PDC | Plugin Delay Compensation，插件延迟补偿 |
| Slide / Portamento | 滑音音符 / 音高过渡 |
| Arrangement | 播放列表的一个编排版本 |
| GM | General MIDI，128 旋律音色 + 鼓组 |
| SF2/SFZ | SoundFont / 采样映射格式 |
| PPQ | 每四分音符的 tick 数，本方案 3840 |