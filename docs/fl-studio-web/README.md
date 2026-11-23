# 在线 FL Studio 复刻版 · 需求分析总览

> 目标：在浏览器中尽可能完整地还原 FL Studio 的核心工作流与插件生态，做一款"在线 FL Studio 类似版"，而不是官方 FL Studio Web 的简化版。
>
> 当前阶段：**只做需求分析与技术预研，不实现代码**。
>
> 资料归档：`references/raw/`（FL Studio 官方手册 26 份 + 原生插件手册 133 份 + 2024–2026 更新日志 + 官方 Web/定价/FAQ 页面 + 技术论文）。

---

## 1. 一句话结论

官方 FL Studio Web 目前是"面向初学者的简化版 Beta"：只有 Channel Rack、基础乐器/效果器、FL Cloud 采样库、教学流程，且**不支持从桌面版导入工程**，仅支持 Web → Desktop 单向打开。我们要做的是**功能密度接近桌面版的完整 Web DAW**，差异化空间明确存在。

但"完整还原"必须分层落地：桌面版有 **133 个原生插件**（54 Generator / 67 效果器 / 5 控制器 / 7 编辑器工具）、500 轨混音器、999 个 Pattern、512 步进、VST/AU、ASIO、视频同步、ReWire 等。浏览器无法 1:1 复刻的部分（VST/AU 原生插件、ASIO、ReWire）需要明确替代方案或列入非目标。

---

## 2. 已完成的资料工作

| 类别 | 数量 | 本地位置 | 说明 |
|---|---:|---|---|
| FL Studio 核心功能手册 | 26 份 | `references/raw/*.txt` | 通道机架、步进音序器、钢琴卷帘、播放列表、混音器、自动化、浏览器、导出、工程设置等 |
| 原生插件手册 | 133 份 | `references/raw/plugins/`（HTML）+ `plugin_descriptions.tsv` | 乐器、效果器、工具、编辑器；Buzz Generator Adapter 单页下载失败，其余齐全 |
| 版本与商业资料 | 10+ 份 | `references/raw/` | 2024/2025/2026 更新日志、定价、版本对比、官方 Web 落地页与 Beta FAQ |
| 技术预研 | 1 份论文 + 网络检索 | `references/raw/wam-studio-paper.pdf` 等 | WAM-Studio、Web Audio、AudioWorklet、WASM、协作、导出、采样库授权 |
| 本地技能资产 | 2 个 Skill | `../../music-skills/skills/` | `famistudio-compose`（音乐理论/编曲/格式）、`music-skill`（MIDI/WAV 读取与转换），可直接用于测试素材与编曲校验 |

> 原始资料归档在 `references/raw/`；主要来源为 Image-Line 官方在线手册与插件页面，更新日志/定价/FAQ 来自官方站点，技术预研含 WAM-Studio 论文。使用这些资料时请遵守其版权，仅作需求分析参考。

---

## 3. 文档导航

| 文档 | 内容 | 读者 |
|---|---|---|
| [`01-product-requirements.md`](01-product-requirements.md) | 产品目标、用户、范围、功能矩阵、非功能需求、验收标准 | 产品/全员 |
| [`02-feature-specification.md`](02-feature-specification.md) | 逐窗口/逐模块的详细功能规格（FR 编号） | 产品/开发/测试 |
| [`03-audio-and-technical-architecture.md`](03-audio-and-technical-architecture.md) | 技术栈、音频引擎、调度、渲染、存储、性能预算、浏览器兼容、安全 | 架构/开发 |
| [`04-instruments-catalog.md`](04-instruments-catalog.md) | 54 个插件 Generator + 通道类型等价映射、18 个引擎家族、100+ 乐器扩展、133 插件全量核对表、GM 128 音色映射表、引擎覆盖矩阵、采样包策略 | 音频/内容 |
| [`05-effects-catalog.md`](05-effects-catalog.md) | 93 个效果器清单（覆盖 67/67 原生效果器）、旗舰规格、效果链模板、优先级 | 音频/开发 |
| [`06-data-model-and-project-format.md`](06-data-model-and-project-format.md) | 工程实体 schema、`.flw` ZIP 格式、版本迁移、FLP 导入映射 | 开发 |
| [`07-roadmap-risks-qa-compliance.md`](07-roadmap-risks-qa-compliance.md) | 分阶段路线图、工作量、风险登记、QA、授权合规、开放问题、采样库授权清单、导出编码方案 | 项目管理/全员 |

---

## 4. 范围边界（先定边界，再谈"完整"）

### 4.1 必须做（P0，MVP 的底线）

- 走带/速度/拍号/摇摆、Song/Pattern 双模式
- Channel Rack + Step Sequencer（512 步、通道分组/静音/独奏/路由）
- Piano Roll（音符属性、Slide/Portamento、事件编辑器、Snap/Scale、量化、Ghost Notes）
- Playlist（Pattern/Audio/Automation Clip、500 轨、Arrangements、Performance Mode）
- Mixer（500 Insert + Current + Master、10 FX Slots、发送/侧链、PDC、EQ/相位/立体声/推子）
- Automation（Automation Clip、Event Automation、LFO/Envelope/Formula/Peak/XY 控制器）
- Browser（工程内容、采样、预设、搜索、试听、拖放）
- 工程新建/打开/保存/自动保存/崩溃恢复、撤销/重做
- MIDI 输入与 MIDI 文件导入导出
- 至少 8 个乐器、10 个效果器达到可用品质（见 `04`/`05` 的 M1 清单）
- WAV 16/24/32-bit 导出；MP3/FLAC 作为 P1

### 4.2 应该做（P1，完整度的主体）

- 音频录制、Edison 式音频编辑、Slicer/Slicex、Sampler/DirectWave 多区采样
- 54 个插件 Generator 等价能力 / 100+ 乐器定义、GM 128 音色 + 60+ 鼓组
- 93 个效果器定义（覆盖 67/67 原生效果器）
- Patcher 模块化、Vocoder、Convolver、Gross Beat、Maximus、NewTone/NewTime
- FLP 导入（尽力而为）、分轨导出、尾音处理、LUFS 表
- 钢琴卷帘脚本、Chord Progression、Riff Machine
- 主题/布局/快捷键自定义、可访问性

### 4.3 可以做（P2，差异化）

- 实时协作（Yjs CRDT）、云端工程、分享链接
- AI 助手（Gopher 式：生成 Piano Roll 脚本/工程建议）
- 分轨/人声分离（Stem Separation）
- 移动端/平板触控布局
- 第三方插件 SDK（WAM/WASM）、音色市场
- 视频同步、ZGameEditor 式可视化

### 4.4 明确非目标（至少在 1.0）

- 原生 VST2/VST3/AU 宿主（浏览器无法加载原生二进制；可做 WAM/WASM 插件协议）
- ASIO/低层音频驱动、ReWire、Buzz 适配器
- 与 FL Studio 像素级一致的 UI/图标/皮肤（商标与 trade dress 风险）
- 100% 保真的 FLP 双向读写（格式闭源、插件状态不透明；只做尽力导入）
- 官方 FL Cloud / Image-Line 账号体系集成

---

## 5. 关键决策摘要（ADR）

| 编号 | 决策 | 理由 | 备选 |
|---|---|---|---|
| ADR-01 | 音频核心采用 **单个 AudioWorklet DSP Graph**，而非成千上万个 Web Audio 原生节点 | 采样级调度、离线渲染一致性、可控 CPU、便于自动化与 PDC | Tone.js 节点图、Elementary Audio |
| ADR-02 | DSP 以 **TypeScript 为主 + Rust/WASM 处理重模块** | 快速迭代 + 重算法性能兼顾 | 全 Rust、全 TS |
| ADR-03 | UI 用 **React + Canvas2D**，时间线/钢琴卷帘自绘并虚拟化 | 500 轨/万级音符仍保持 60fps | 全 DOM、PixiJS/WebGL |
| ADR-04 | 状态用 **Zustand + Immer + Command Bus**，撤销基于命令逆操作 | 复杂编辑可预测、可协作迁移 | Redux、Yjs 直接驱动 |
| ADR-05 | 工程文件用 **ZIP + JSON + assets**（自有 `.flw`），FLP 仅尽力导入 | 可版本化、可迁移、可协作 | 纯 JSON、直接写 FLP |
| ADR-06 | 乐器策略：**54 个 Generator 等价 + GM 128 + 100+ 乐器定义 + 扩展世界乐器/预设库** | 满足"种类多"且可控 | 只做合成器、只做采样器 |
| ADR-07 | 品牌与素材 **完全独立**：不使用 FL 名称/图标/内置采样/预设 | 合规 | 直接复刻 |
| ADR-08 | 先 **本地优先（IndexedDB/OPFS）**，云同步后置 | 降低后端成本与隐私风险 | 云优先 |

---

## 6. 核心开放问题（需要产品拍板）

1. **产品定位**：面向 FL 老用户（重快捷键/工作流保真）还是面向新手（重引导/简化）？两者 UI 密度冲突，建议"默认专业布局 + 可选简化模式"。
2. **商业模式**：免费 + 云存储/音色包订阅？还是买断？影响云与采样授权成本。
3. **乐器深度**：优先"数量多、开箱即用"（GM/采样为主），还是"少数深度合成器"（Sytrus/Harmor 级）？建议 P0 数量、P1 深度。
4. **FLP 导入优先级**：是否必须支持老工程迁移？这决定是否要投入 dawparse/FLP 兼容专项。
5. **MP3 导出授权**：lamejs 为 LGPL，需法务确认动态链接/源码提供方式；或改用 WebCodecs AAC/Opus。方案见 [`07` 附录 B](07-roadmap-risks-qa-compliance.md)。
6. **采样库授权**：GeneralUser GS 允许用于软件项目；VSCO 2 CE 为 CC0；Salamander 为 CC BY 3.0；FluidR3/Sonatina 需逐项核验。上线前必须完成授权清单（见 [`07` 附录 A](07-roadmap-risks-qa-compliance.md)）。
7. **协作范围**：只做工程数据同步，还是包含音频试听/聊天？后者需要后端与带宽预算。
8. **移动端**：1.0 是否承诺平板可用？触控交互会显著增加设计成本。
9. **插件生态**：是否开放第三方 WASM 插件？涉及沙箱、签名、分成与安全审核。
10. **AI 功能**：是否内置 Gopher 式助手？涉及模型成本、隐私、离线降级。

---

## 7. 建议的下一步（本阶段之后）

1. 评审本套文档，冻结 P0 范围与验收标准。
2. 做 2 周技术 Spike：AudioWorklet 引擎原型（32 复音 + 3 轨 + 1 混响 + 离线导出），验证延迟/CPU/ Safari。
3. 做 1 周 UI Spike：500 轨 Playlist + 1 万音符 Piano Roll 的 Canvas 虚拟化帧率。
4. 按 [`07` 附录 A/B](07-roadmap-risks-qa-compliance.md) 逐项确认采样库授权与导出编码方案。
5. 进入 Phase 0/1（见 [`07-roadmap-risks-qa-compliance.md`](07-roadmap-risks-qa-compliance.md)）。

---

## 8. 合规声明

本项目是**独立实现的 FL Studio 工作流类似产品**，与 Image-Line 无关联。文档中出现的 FL Studio、Fruity、Sytrus、Harmor 等名称仅用于功能对照与兼容性说明，不作为产品名、插件名或商标使用。实现时使用原创名称、原创 UI、原创预设与已授权采样。