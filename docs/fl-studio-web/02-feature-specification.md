# 02 · 详细功能规格（Feature Specification）

> 本文按"窗口/模块"拆解功能，使用 FR 编号便于追踪与验收。优先级：**P0** 必须有、**P1** 1.0 目标、**P2** 后续增强。
> 交互语义以 FL Studio 桌面版手册为准；无法在浏览器实现的部分给出替代方案。

---

## 0. 全局约定

### 0.1 时间与网格

- 内部时间单位：**PPQ = 3840 tick/四分音符**，兼容三连音、五连音、七连音与高倍缩放。
- 显示单位：小节:拍:tick，可切换为秒/采样点。
- 速度范围：**10–522 BPM**，支持 1/1000 BPM 微调；半速播放开关；Tap Tempo。
- 拍号：工程级默认拍号 + 时间线/Pattern 级拍号标记，支持多拍号与多节奏（polyrhythm）。
- 摇摆：Global Swing + 每通道 Swing Mix（0–100%），两者相乘生效；可设置是否截断摆动后的重叠音符。

### 0.2 通用编辑语义

- 所有编辑必须可撤销；连续拖拽合并为一条撤销记录。
- 吸附（Snap）：Main/Bar/Beat/Step/Line/None 等；按住 Alt 临时禁用吸附。
- 右键删除、左键绘制、Ctrl 框选、Shift 增减选择，与 FL 保持一致。
- 所有参数可"右键 → Create automation clip / Link to controller"（原生参数）。
- 所有窗口共享选择上下文与播放头；窗口可停靠、浮动、分离到第二显示器。

### 0.3 窗口与布局

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-SYS-01 | 默认布局还原 FL 主窗口：顶部工具栏 + Browser 左侧 + 中央 Playlist/Piano Roll + 底部 Channel Rack/Mixer，可切换 | P0 |
| FR-SYS-02 | 窗口可拖动、缩放、最大化、停靠、浮动；布局可保存为 Workspace | P1 |
| FR-SYS-03 | 多显示器：支持将窗口弹出到独立浏览器窗口并保持状态同步 | P2 |
| FR-SYS-04 | 主题：深色/浅色/自定义强调色；支持导入主题包（JSON） | P1 |
| FR-SYS-05 | 全局缩放 80%–200%，文字与控件独立缩放 | P1 |
| FR-SYS-06 | 响应式：宽度 <1280px 自动收起 Browser；<900px 进入平板布局 | P2 |
| FR-SYS-07 | 崩溃/刷新后恢复窗口布局与打开的面板 | P1 |

---

## 1. 工程与设置

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-PRJ-01 | 新建工程：模板（空白/Beat/EDM/管弦/教学），默认 4 轨、120 BPM、4/4 | P0 |
| FR-PRJ-02 | 打开工程：本地文件（`.flw`）、最近工程、示例工程、云工程（P2） | P0 |
| FR-PRJ-03 | 保存/另存为：ZIP 工程包；支持 File System Access API 直接写盘，否则下载 | P0 |
| FR-PRJ-04 | 自动保存：每 30 秒或每 20 次编辑写入 IndexedDB 快照，保留最近 10 份 | P0 |
| FR-PRJ-05 | 崩溃恢复：启动时检测未正常关闭会话，提示恢复 | P0 |
| FR-PRJ-06 | 工程设置：标题、作者、备注、BPM、拍号、采样率、混响尾音、颜色 | P0 |
| FR-PRJ-07 | 版本历史：本地快照列表，可预览/回滚/另存 | P1 |
| FR-PRJ-08 | 导入：FLP（尽力）、MIDI、音频、SF2/SFZ、预设包 | P1 |
| FR-PRJ-09 | 导出工程：`.flw`、分轨 ZIP、MIDI、音频 | P0/P1 |
| FR-PRJ-10 | 工程统计：轨道数、音符数、采样占用、CPU 估算 | P1 |
| FR-PRJ-11 | 缺失采样：打开时列出缺失文件，支持逐个/批量重链、跳过、静音占位 | P0 |
| FR-PRJ-12 | 只读/演示模式：分享链接可设为只读，隐藏编辑工具 | P2 |

**边界情况**

- 工程引用外部采样时，保存必须把采样打包进 ZIP（除非用户选择"仅引用"）。
- 同名工程覆盖前需确认；自动保存不得覆盖用户手动保存版本。
- 采样缺失时不得阻塞工程打开；以静音占位并标记。

---

## 2. 走带与播放控制（Transport）

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-TRN-01 | Play/Pause/Stop；右键 Stop 回到上次起始位置或歌曲起点（可设置） | P0 |
| FR-TRN-02 | Song / Pattern 双模式；Pattern 模式循环当前 Pattern | P0 |
| FR-TRN-03 | BPM 调整（键入/拖拽/滚轮）、Tap Tempo、半速、微调 | P0 |
| FR-TRN-04 | 拍号设置与时间线拍号标记；Pattern 独立拍号 | P1 |
| FR-TRN-05 | 循环：选区循环、Loop 标记、Pattern 循环、循环跳转 | P0 |
| FR-TRN-06 | 播放头拖动/点击定位；播放中 scrub（可选） | P0 |
| FR-TRN-07 | 节拍器：音量/音色/仅录制时；Count-in 1/2/4 小节 | P1 |
| FR-TRN-08 | Punch In/Out 录制范围 | P1 |
| FR-TRN-09 | 时间标记（Time Marker）：命名、跳转、设置 Pattern 长度/拍号 | P1 |
| FR-TRN-10 | 播放速度/音高不变（仅音频剪辑 time-stretch 时） | P1 |
| FR-TRN-11 | 后台标签页：音频继续（允许时）；恢复时校准播放头 | P1 |
| FR-TRN-12 | 音频设备中断/切换：自动恢复并提示，不丢工程状态 | P0 |

**交互细节**

- 空格 = Play/Pause；停止按钮右键行为可配置。
- 播放头位置以 AudioContext 时钟为准，UI 以 rAF 插值，避免抖动。
- 循环边界必须采样级精确，不允许 UI 层截断。

---

## 3. Channel Rack 与 Step Sequencer

### 3.1 通道机架

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-CR-01 | 通道列表：每通道显示静音、Pan、Volume、Mixer 路由、通道按钮、选择器、活动指示 | P0 |
| FR-CR-02 | 通道类型：Generator（乐器）、Sampler、Audio Clip、Automation Clip、Layer、Controller | P0 |
| FR-CR-03 | 添加通道：插件选择器、Browser 拖放、收藏夹、克隆、从模板 | P0 |
| FR-CR-04 | 通道按钮打开插件 UI；选择器用于现场 MIDI 控制；两者可独立 | P0 |
| FR-CR-05 | 通道设置：名称、颜色、Mixer 路由、增益、Pan、Pitch、Declick、采样起止/循环、ADSR（依引擎） | P0 |
| FR-CR-06 | 分组与过滤：Group、Display Filter、按颜色/名称/类型/路由排序 | P1 |
| FR-CR-07 | 批量操作：静音/独奏/克隆/删除/上下移动/转调/设置 Swing Mix | P1 |
| FR-CR-08 | Layer 通道：多层叠加、独奏层、交叉淡入淡出 | P1 |
| FR-CR-09 | 通道压缩视图（Zip）与自动 Zip 空通道 | P2 |
| FR-CR-10 | 通道高度/按钮宽度/名称对齐可调 | P1 |
| FR-CR-11 | 路由快捷键：Ctrl+L 分配到空闲 Mixer Track，Ctrl+Shift+L 顺序分配 | P1 |
| FR-CR-12 | 通道峰值/活动指示；CPU 占用提示 | P2 |

**上限与规则**

- 通道数上限 500（与 FL 对齐；UI 虚拟化）。
- 每个 Pattern 共享同一组通道；Pattern 不是"每乐器一个"。
- 删除被 Playlist/自动化引用的通道前必须警告并给出替代方案。

### 3.2 Step Sequencer

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-SS-01 | 每通道步进网格；步数 1–512，Pattern 长度可设 | P0 |
| FR-SS-02 | 步进开/关、力度（点击后拖拽/右键）、步进类型（普通/触发/门限，视引擎） | P0 |
| FR-SS-03 | 网格缩放：16/32/64 步视图；滚动与缩放不改变数据 | P0 |
| FR-SS-04 | Global Swing 与 Channel Swing Mix；Swing 截断选项 | P0 |
| FR-SS-05 | Step 数据与 Piano Roll 数据可共存；切换显示模式不丢数据 | P0 |
| FR-SS-06 | 步进复制/粘贴/填充/清除/随机化（受控） | P1 |
| FR-SS-07 | 步进标签/音符名称（鼓组映射） | P1 |
| FR-SS-08 | 步进编辑的键盘操作（方向键移动、空格切换） | P1 |

**边界情况**

- 改变 Pattern 长度时，超出新长度的步进数据保留但灰显，恢复长度后重现。
- 摆动导致音符重叠时，按通道设置截断或允许重叠；不得产生未定义行为。

---

## 4. Piano Roll

### 4.1 音符编辑

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-PR-01 | 音符绘制/删除/选择/移动/缩放；默认长度 = 上次触碰音符长度 | P0 |
| FR-PR-02 | 工具集：Draw、Paint、Paint Drum、Delete、Mute、Slice、Select、Zoom、Play、Interpolate | P0 |
| FR-PR-03 | 吸附：Main/Bar/Beat/Step/Line/None；Alt 临时禁用 | P0 |
| FR-PR-04 | 目标通道切换；Ghost Notes 显示其他通道音符；可编辑 Ghost | P1 |
| FR-PR-05 | 音符属性：Velocity、Pan、Release Velocity、Pitch、Fine Pitch、Mod X（Cutoff）、Mod Y（Resonance）、Slide、Portamento、Delay、Key/Articulation | P0 |
| FR-PR-06 | Slide/Portamento 语义：Slide 音符不发声、作用于重叠音符；Porta 作用于音符过渡 | P0 |
| FR-PR-07 | 事件编辑器：音符属性与 Event Automation 曲线；可插值 | P0 |
| FR-PR-08 | 音符分组：Group、Color Group（对应 MIDI 通道）、编辑联动 | P1 |
| FR-PR-09 | 量化：强度、步长、仅选中；Humanize（时间/力度） | P1 |
| FR-PR-10 | 复制/粘贴/重复（Ctrl+B）/镜像/反转/转调/时间偏移 | P0 |
| FR-PR-11 | 切片/胶合（Ctrl+G）/从左侧缩放（可选） | P1 |
| FR-PR-12 | 时间线选择 + 按范围选择音符；框选/魔术套索 | P1 |
| FR-PR-13 | 预览键盘；点击发声；拖拽选择音高范围 | P0 |
| FR-PR-14 | 背景网格、拍号线、时间标记、节拍器对齐显示 | P1 |
| FR-PR-15 | 缩放：水平/垂直独立；快捷键与滚轮；缩放中心跟随鼠标 | P0 |
| FR-PR-16 | 撤销/重做覆盖所有音符操作；拖拽合并 | P0 |

### 4.2 作曲辅助

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-PR-20 | Snap to Scale：根音、音阶、编辑/录入吸附、阴影显示 | P1 |
| FR-PR-21 | Chord Stamp：和弦类型、自动和弦（上/下）、旋律/音阶和弦、打击乐模板 | P1 |
| FR-PR-22 | Chord Progression Tool：生成/替换/声部引导/节奏化 | P2 |
| FR-PR-23 | Riff Machine：模式生成、力度/时间变化、可预览 | P2 |
| FR-PR-24 | Piano Roll 脚本：JS 脚本 API（读取选区、修改音符、参数面板） | P2 |
| FR-PR-25 | 乐谱导出（MusicXML/MIDI） | P2 |

**边界情况**

- 同一通道同一时间允许多个重叠音符（和弦），但 Slide 音符语义需按颜色组隔离。
- 音符数量上限建议 100,000/Pattern；超过时提示拆分。
- 编辑时不得触发音频线程分配（避免 GC 抖动）。

---

## 5. Playlist

### 5.1 轨道与剪辑

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-PL-01 | 500 条 Clip Track；轨道可命名/着色/静音/独奏/锁定/高度调整 | P0 |
| FR-PL-02 | Clip 类型：Pattern Clip、Audio Clip、Automation Clip；可重叠/分层 | P0 |
| FR-PL-03 | 轨道模式：普通（不绑定乐器）与 Track Mode（乐器/音频轨道，1:1 绑定 Mixer） | P1 |
| FR-PL-04 | 绘制/涂刷/删除/静音/切片/滑移/选择/缩放/播放选中 | P0 |
| FR-PL-05 | 剪辑属性：起始、长度、偏移、增益、淡入/淡出、颜色、名称、静音 | P0 |
| FR-PL-06 | Audio Clip：Pitch、Time Stretch、Reverse、Slice、Loop、Tempo Sync、声道处理 | P1 |
| FR-PL-07 | Pattern Clip：选择 Pattern、长度、内部偏移、循环、裁剪音符 | P0 |
| FR-PL-08 | 剪辑分组、吸附、对齐、复制/粘贴、多重选择 | P0 |
| FR-PL-09 | 多 Arrangement：克隆/重命名/删除/合并/新增；共享 Channel Rack/Mixer | P1 |
| FR-PL-10 | Performance Mode：将剪辑映射到键盘/打击垫触发 | P1 |
| FR-PL-11 | 时间标记、Loop 标记、歌曲结尾、节拍线 | P1 |
| FR-PL-12 | 轨道头拖入乐器创建 Track Mode 链接 | P1 |
| FR-PL-13 | 剪辑渲染/合并（Consolidate）、提取 Stems（P2） | P1/P2 |
| FR-PL-14 | 剪辑搜索/过滤/跳转；大工程虚拟化 | P0 |
| FR-PL-15 | 轨道/剪辑颜色规则与主题一致；色盲友好模式 | P1 |

### 5.2 编辑规则

- Playlist 轨道**不绑定** Mixer；声音路由由 Channel → Mixer 决定。Track Mode 才建立 1:1 关系。
- 同一轨道可放任意 Clip 类型；允许重叠，但需明确的"焦点剪辑"选择规则。
- 切片必须保持源素材偏移；滑移不改变剪辑起止。
- 拖动剪辑跨轨道不改变其内部数据，仅改变轨道归属。

**边界情况**

- 删除 Arrangement 不得删除仍被其他 Arrangement 引用的通道/效果器。
- 500 轨上限下，自动化剪辑不应挤占普通轨道（FL 论坛已有争议；本方案允许自动化子轨）。
- 时间线缩放至极小/极大时仍可精确选择。

---

## 6. Mixer

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-MX-01 | 500 Insert Track + Current + Master；逻辑上限固定，UI 虚拟化 | P0 |
| FR-MX-02 | 每轨 10 个 FX Slot；支持插入/替换/重排/旁通/湿干比 | P0 |
| FR-MX-03 | 每轨集成控制：3 段 EQ、相位反转、左右交换、Stereo Separation、Pan、Fader、Mute/Solo、Arm | P0 |
| FR-MX-04 | 路由：任意 Insert → 任意 Insert/Master；发送开关 + 发送电平；侧链 | P0 |
| FR-MX-05 | PDC：自动延迟补偿 + 手动 Track Delay（采样） | P1 |
| FR-MX-06 | 峰值/RMS/LUFS 表；Master 削波指示；可选波形表 | P1 |
| FR-MX-07 | 轨道命名/着色/重排/复制状态（含 FX 链） | P1 |
| FR-MX-08 | FX 链预设、整轨预设、Patcher 化 | P1 |
| FR-MX-09 | 录音武装与输入选择（音频接口）；监听 | P1 |
| FR-MX-10 | 分轨渲染与独奏试听 | P1 |
| FR-MX-11 | 多轨选择联动推子；Ctrl+Shift 选择 | P1 |
| FR-MX-12 | 混音器布局：Compact → Extra Large，按尺寸显示不同控件 | P1 |
| FR-MX-13 | 路由环路检测与禁止；删除轨道前检查引用 | P0 |
| FR-MX-14 | 500 轨性能：未使用轨不分配 DSP 资源 | P0 |

**边界情况**

- 发送到自身的路由必须被拒绝；允许 A→B→C，不允许 C→A。
- 删除有发送/被发送的轨道时，自动重接或明确警告。
- Master 是唯一允许削波的轨（与 FL 语义一致），Insert 轨可超过 0dB 内部浮点。

---

## 7. 自动化与控制器

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-AUT-01 | Automation Clip：控制点、张力、单/双/非对称曲线、Step 模式、Slide 模式 | P0 |
| FR-AUT-02 | 从任意原生参数右键创建 Automation Clip；初始化值记录与恢复 | P0 |
| FR-AUT-03 | Event Automation：随 Pattern 存储的控制器事件；事件编辑器编辑；可转 Automation Clip | P1 |
| FR-AUT-04 | 控制器插件：Envelope、LFO、Formula、Peak、XY、Keyboard、Control Surface | P1 |
| FR-AUT-05 | 参数链接：MIDI CC、OSC（P2）、内部控制器、Macro | P1 |
| FR-AUT-06 | 自动化录制：Touch/Latch/Write 模式；覆盖/合并策略 | P1 |
| FR-AUT-07 | 自动化平滑/量化/简化控制点；曲线复制/粘贴 | P1 |
| FR-AUT-08 | 时间线自动化（BPM/拍号）与音频剪辑音量/Pan 自动化 | P1 |
| FR-AUT-09 | 自动化预览与实时编辑不触发音频中断 | P0 |

**边界情况**

- 删除 Automation Clip 后参数回到初始化值，不得停留在最后值。
- 自动化与手动拖动冲突时，以"最后写入"策略并给出视觉提示。
- 参数范围映射必须支持对数/指数曲线（频率、增益）。

---

## 8. Browser 与内容管理

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-BR-01 | 标签页：Current Project、Plugin Presets、Samples、Packs、Favorites、Cloud（P2） | P0 |
| FR-BR-02 | 树形/列表浏览；搜索（名称/标签/BPM/Key/类型）；排序 | P0 |
| FR-BR-03 | 采样试听：波形、播放/停止、循环、BPM/Key 同步、音量 | P0 |
| FR-BR-04 | 拖放到 Channel Rack/Playlist/Mixer/Piano Roll 的语义正确 | P0 |
| FR-BR-05 | 预设管理：保存/重命名/删除/导入/导出；按插件分组 | P1 |
| FR-BR-06 | 收藏、最近使用、相似音色推荐（本地特征或云端） | P1 |
| FR-BR-07 | 采样元数据：BPM、Key、时长、采样率、声道、标签、授权 | P1 |
| FR-BR-08 | 缺失采样重链；采样替换保持剪辑参数 | P1 |
| FR-BR-09 | 工程内容树：Patterns、Channels、Mixer、Automations、Samples | P1 |
| FR-BR-10 | 采样导入：拖入自动解码、去重（哈希）、格式转换缓存 | P1 |
| FR-BR-11 | 大库性能：10,000+ 条目虚拟列表 + 增量索引（IndexedDB） | P1 |

---

## 9. 乐器与插件框架

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-INS-01 | 统一乐器定义：参数 schema、预设、UI 生成、自动化暴露、复音策略 | P0 |
| FR-INS-02 | 引擎类型：减法、FM、加法、采样、颗粒、物理建模、鼓合成、切片、SoundFont、模块化 | P1 |
| FR-INS-03 | 参数模型：ID、名称、范围、单位、曲线、默认值、可自动化、Mod 目标 | P0 |
| FR-INS-04 | 预设：工厂预设 + 用户预设 + 收藏；预设可包含采样引用 | P1 |
| FR-INS-05 | 多输出乐器（鼓机/分层）路由到多个 Mixer Track | P1 |
| FR-INS-06 | 复音/单音/连奏/Mono Legato；Voice Stealing 策略 | P0 |
| FR-INS-07 | 乐器 UI：通用 Schema UI + 复杂插件的自定义 Canvas UI | P1 |
| FR-INS-08 | 插件沙箱：仅 WASM/受控 JS，无网络/文件系统直通；CPU/内存配额 | P1 |
| FR-INS-09 | 插件注册表：内置 + 用户安装包（签名/校验） | P2 |
| FR-INS-10 | 第三方插件协议：WAM 兼容或自定义 WASM ABI | P2 |
| FR-INS-11 | VST/AU：不宿主；提供"渲染为采样/音频"的替代工作流 | Out |

---

## 10. 录音与音频编辑

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-REC-01 | 音频输入：设备枚举、采样率/声道、权限请求、电平表 | P1 |
| FR-REC-02 | 录音武装：Mixer Track Arm、输入选择、监听开关、反馈保护 | P1 |
| FR-REC-03 | 录音到 Playlist Audio Clip；Take 管理；命名规则 | P1 |
| FR-REC-04 | 延迟补偿：输入/输出延迟测量与自动对齐 | P1 |
| FR-REC-05 | MIDI 录制：通道选择、量化、 overdub、接管 | P0 |
| FR-REC-06 | Edison 式音频编辑器：波形编辑、裁剪、归一化、淡入淡出、EQ、卷积混响 | P1 |
| FR-REC-07 | Slicer/Slicex：自动切片、切片标记、触发、导出切片 | P1 |
| FR-REC-08 | NewTone/NewTime：音高校正、时间伸缩（WSOLA/PSOLA） | P2 |
| FR-REC-09 | Stem Separation：人声/鼓/贝斯/其他（模型推理，WASM/服务端） | P2 |
| FR-REC-10 | 录音文件格式：WAV 16/24/32f；可选 FLAC | P1 |

**边界情况**

- 麦克风权限被拒：功能降级但工程可继续编辑。
- 录音过程中切换设备：停止并提示，避免半截文件。
- 录制长度上限按可用内存/磁盘配额动态提示。

---

## 11. 导出与渲染

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-EXP-01 | 渲染范围：Song / Pattern / 时间线选区 | P0 |
| FR-EXP-02 | 尾音：Leave remainder / Wrap remainder / Cut remainder | P1 |
| FR-EXP-03 | 格式：WAV 16/24/32f、MP3、OGG、FLAC、M4A（按浏览器能力） | P0/P1 |
| FR-EXP-04 | 采样率：44.1/48/96 kHz；声道：Stereo/Mono(L+R)/Mono L/Mono R | P1 |
| FR-EXP-05 | 分轨导出：每 Mixer Track 一文件，命名规则可配 | P1 |
| FR-EXP-06 | 离线渲染使用与实时相同的 DSP 图，保证一致性 | P0 |
| FR-EXP-07 | 进度/剩余时间/取消/失败重试；后台继续（可选） | P0 |
| FR-EXP-08 | 导出预设：平台（YouTube/Spotify/播客）、码率、归一化 | P1 |
| FR-EXP-09 | MIDI 导出：Pattern/通道/速度/拍号 | P1 |
| FR-EXP-10 | 工程导出：`.flw`、采样包、分轨 ZIP | P1 |
| FR-EXP-11 | 导出后校验：时长、峰值、文件可解码 | P0 |
| FR-EXP-12 | 批量队列：多个格式/多个工程 | P2 |

**边界情况**

- MP3 在非 44.1/48kHz 采样率下需重采样或禁用并提示。
- 导出中修改工程：使用导出开始时的快照，不读取后续编辑。
- 大工程内存不足：分块渲染并写入临时文件（OPFS）。

---

## 12. 撤销、剪贴板与历史

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-HIS-01 | 全局撤销/重做（Ctrl+Z / Ctrl+Shift+Z），按窗口聚焦优先 | P0 |
| FR-HIS-02 | 命令合并：拖拽/连续参数调整合并为单条 | P0 |
| FR-HIS-03 | 历史面板：操作名称、时间、跳转到任意状态 | P1 |
| FR-HIS-04 | 剪贴板：音符、剪辑、通道、FX 链、自动化；跨工程粘贴（兼容部分） | P1 |
| FR-HIS-05 | 撤销不得破坏音频线程状态；参数变更走无锁通道 | P0 |
| FR-HIS-06 | 历史持久化（随自动保存）可选 | P2 |

---

## 13. 协作（P2）

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-COL-01 | 协作房间：邀请链接、权限（编辑/评论/只读） | P2 |
| FR-COL-02 | CRDT 同步：音符/剪辑/参数/混音；离线合并 | P2 |
| FR-COL-03 | Presence：光标、选中、播放头、正在编辑对象 | P2 |
| FR-COL-04 | 冲突策略：按对象粒度合并；破坏性操作需确认 | P2 |
| FR-COL-05 | 版本历史与回滚；按人筛选变更 | P2 |
| FR-COL-06 | 音频试听同步（可选，WebRTC） | P2 |
| FR-COL-07 | 协作下的撤销：仅撤销自己的操作 | P2 |

---

## 14. 引导、帮助与可访问性

| 编号 | 需求 | 优先级 |
|---|---|---|
| FR-ONB-01 | 首次启动引导：选择目标（做 Beat/学音乐/导入工程） | P1 |
| FR-ONB-02 | 交互式课程：3–5 节，每节 2–5 分钟，带检查点 | P1 |
| FR-ONB-03 | 上下文提示/工具提示/参数说明（来自本地知识库） | P1 |
| FR-ONB-04 | 示例工程：Beat、House、管弦、Lo-Fi、教学 | P1 |
| FR-ONB-05 | 快捷键速查表与可自定义映射 | P1 |
| FR-ONB-06 | 键盘导航：Tab 焦点、方向键、可访问名称 | P1 |
| FR-ONB-07 | 高对比度/色盲模式/减少动画 | P1 |
| FR-ONB-08 | 中英双语；错误信息可操作 | P1 |

---

## 15. 默认快捷键（与 FL 对齐）

| 按键 | 功能 |
|---|---|
| F1 | 帮助 |
| F5 | 显示/隐藏 Playlist |
| F6 | 显示/隐藏 Channel Rack / Step Sequencer |
| F7 | 显示/隐藏 Piano Roll |
| F8 | Plugin Picker |
| F9 | 显示/隐藏 Mixer |
| F10 | MIDI 设置 |
| F11 | 工程信息/设置 |
| Alt+F8 | Browser |
| Ctrl+F8 | Project Picker |
| 空格 | Play/Pause |
| Ctrl+Z / Ctrl+Shift+Z | 撤销/重做 |
| Ctrl+C / Ctrl+V / Ctrl+X | 复制/粘贴/剪切 |
| Ctrl+B | 复制选区到右侧 |
| Ctrl+G | 胶合音符 |
| Ctrl+E | 打开 Edison（音频编辑） |
| Alt+Q | 量化 |
| Shift+Z | 缩放到选区 |
| Shift+Alt+T | 时间标记/拍号 |
| Ctrl+L | 通道分配到空闲 Mixer Track |
| Ctrl+Shift+L | 顺序分配 Mixer Track |

> 所有快捷键可自定义并持久化；冲突检测与一键恢复默认。

---

## 16. 上限与容量（建议值）

| 项目 | 上限 | 说明 |
|---|---:|---|
| Pattern 数 | 999 | 与 FL 对齐 |
| Step Sequencer 步数 | 512 | 与 FL 对齐 |
| Channel 数 | 500 | UI 虚拟化 |
| Playlist 轨 | 500 | 与 FL 对齐 |
| Mixer Insert | 500 + Current + Master | 未使用轨不分配 DSP |
| FX Slots/轨 | 10 | Patcher 可突破 |
| 音符/Pattern | 100,000（软上限） | 超出提示拆分 |
| 工程大小 | 2GB（软上限） | 受浏览器配额限制 |
| 采样数/工程 | 5,000 | 去重 + 流式 |
| 撤销步数 | 1,000 | 可配置 |
| 协作人数 | 8（P2） | 按后端能力 |

---

## 17. 关键错误与降级

| 场景 | 行为 |
|---|---|
| AudioContext 无法启动 | 显示"点击启用音频"手势按钮；编辑功能仍可用 |
| AudioWorklet 不支持 | 阻止进入工程，给出浏览器升级指引（不做 ScriptProcessor 降级） |
| 音频设备丢失 | 暂停播放、保留工程、提示重新选择设备 |
| 采样解码失败 | 标记损坏、静音占位、允许替换 |
| 工程解析失败 | 不覆盖原文件、显示错误码、提供导出诊断包 |
| 内存/配额不足 | 阻止导入大采样、提示清理缓存、降级流式 |
| 导出失败 | 清理临时文件、保留设置、可重试 |
| 路由环路 | 阻止连接并高亮冲突轨 |
| 插件崩溃（WASM） | 隔离该实例、旁通、保留其他轨 |

---

## 18. 需求追踪

- 本文 FR 与 `01-product-requirements.md` 的 AC 对应关系：
  - AC-01 ↔ FR-TRN、FR-SYS、FR-INS
  - AC-02 ↔ FR-SS
  - AC-03 ↔ FR-PR
  - AC-04 ↔ FR-PL
  - AC-05 ↔ FR-MX
  - AC-06 ↔ FR-AUT
  - AC-07 ↔ FR-PRJ、FR-HIS
  - AC-08 ↔ FR-EXP
  - AC-09 ↔ FR-REC、FR-EXP-09
  - AC-10 ↔ NFR-03/05
  - AC-11 ↔ NFR-09
  - AC-12 ↔ `07-roadmap-risks-qa-compliance.md` 合规清单

---

## 19. 附录：FL Studio 功能面覆盖审计

> 目的：证明"尽可能完整"不是口号。下表按 FL Studio 的实际功能面逐项对照，标注本项目落点与状态。
> 状态含义：**覆盖** = 已有 FR/规格；**部分** = 有替代或降级方案；**不做** = 明确非目标；**后续** = P2/P3。

### 19.1 应用外壳与全局

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| 主菜单与工具栏 | File/Edit/Add/Patterns/View/Options/Tools/Help | FR-SYS-01、02 | 覆盖 |
| 窗口显示/隐藏/停靠/浮动 | F5–F11、Alt+F8 | FR-SYS-01/02、快捷键表 | 覆盖 |
| Workspace/布局保存 | Workspace | FR-SYS-02/07 | 覆盖 |
| 主题/缩放 | Themes、UI Scaling | FR-SYS-04/05 | 覆盖 |
| 快捷键自定义 | Shortcut config | FR-ONB-05 | 覆盖 |
| 撤销/重做/历史 | Undo history | FR-HIS-01～06 | 覆盖 |
| 剪贴板跨窗口 | Copy/Paste | FR-HIS-04 | 覆盖 |
| 帮助/提示/教程 | Help、Lessons | FR-ONB-01～04 | 覆盖 |
| 中英双语/可访问性 | — | FR-ONB-06～08 | 覆盖 |

### 19.2 工程、设置与文件

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| 新建/模板 | New project / Templates | FR-PRJ-01 | 覆盖 |
| 工程设置/Song Info | F11 / Project Info | FR-PRJ-06 | 覆盖 |
| 保存/另存/打包采样 | Save / Save As | FR-PRJ-03、06 格式 | 覆盖 |
| 自动保存/崩溃恢复 | Autosave / Recovery | FR-PRJ-04/05 | 覆盖 |
| 版本历史/回滚 | — | FR-PRJ-07 | 覆盖 |
| 最近工程/Project Picker | Ctrl+F8 | FR-PRJ-02 | 覆盖 |
| FLP 导入 | File → Import FLP | FR-PRJ-08、06 映射 | 部分（尽力） |
| MIDI/音频设置 | F10 / Audio Settings | FR-REC-01、03 兼容 | 覆盖 |
| 工程统计/诊断 | — | FR-PRJ-10、03 可观测性 | 覆盖 |
| 分享/只读演示 | — | FR-PRJ-12 | 后续 |

### 19.3 走带、时间与录制

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| Play/Pause/Stop/Song/Pattern | Transport | FR-TRN-01/02 | 覆盖 |
| BPM/Tap/半速/微调 | Tempo controls | FR-TRN-03 | 覆盖 |
| 拍号/多拍号/多节奏 | Time signature | FR-TRN-04 | 覆盖 |
| Swing（全局/通道） | Swing | FR-TRN-04、FR-SS-04 | 覆盖 |
| 节拍器/Count-in | Metronome | FR-TRN-07 | 覆盖 |
| Punch In/Out | Recording range | FR-TRN-08 | 覆盖 |
| Loop/循环跳转 | Loop markers | FR-TRN-05 | 覆盖 |
| Time Marker/歌曲结尾 | Time markers | FR-TRN-09、FR-PL-11 | 覆盖 |
| 音频录制/监听 | Recording | FR-REC-01～04 | 覆盖 |
| MIDI 录制/Overdub | MIDI recording | FR-REC-05 | 覆盖 |
| 延迟补偿/设备切换 | Latency / device | FR-REC-04、FR-TRN-12 | 覆盖 |

### 19.4 创作核心

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| Channel Rack | F6 | FR-CR-01～12 | 覆盖 |
| Step Sequencer 512 步 | Step Sequencer | FR-SS-01～08 | 覆盖 |
| 通道设置/采样器 | Channel Settings | FR-CR-05 | 覆盖 |
| 通道分组/过滤/排序 | Groups/Filters | FR-CR-06/07 | 覆盖 |
| Layer 层叠 | Layer | FR-CR-08、04 工具表 | 覆盖 |
| Piano Roll 基础编辑 | F7 | FR-PR-01～16 | 覆盖 |
| Ghost Notes/音符分组 | Ghost/Group | FR-PR-04/08 | 覆盖 |
| 量化/Humanize/切片/胶合 | Quantize/Slice/Glue | FR-PR-09/11 | 覆盖 |
| Scale Snap/Chord Stamp | Scale/Chord | FR-PR-20/21 | 覆盖 |
| Chord Progression/Riff Machine | Composition tools | FR-PR-22/23 | 后续 |
| Piano Roll 脚本 | Piano Roll Scripts | FR-PR-24 | 后续 |
| 事件编辑器 | Event Editor | FR-PR-07、FR-AUT-03 | 覆盖 |
| Playlist 基础 | F5 | FR-PL-01～08 | 覆盖 |
| 多 Arrangement | Arrangements | FR-PL-09 | 覆盖 |
| Performance Mode | Performance | FR-PL-10 | 覆盖 |
| Track Mode/轨道绑定 | Track Mode | FR-PL-03/12 | 覆盖 |
| 剪辑增益/淡入淡出/切片 | Clip properties | FR-PL-05/06/13 | 覆盖 |
| 时间标记/循环/拍号 | Markers | FR-PL-11、FR-TRN-09 | 覆盖 |

### 19.5 混音、自动化与效果

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| Mixer 500 Insert + Master | F9 | FR-MX-01～14 | 覆盖 |
| 10 FX Slots/旁通/湿干 | FX slots | FR-MX-02 | 覆盖 |
| EQ/相位/立体声/推子 | Channel strip | FR-MX-03 | 覆盖 |
| Sends/Sidechain/PDC | Routing | FR-MX-04/05 | 覆盖 |
| Metering/LUFS/削波 | Meters | FR-MX-06 | 覆盖 |
| 分轨导出/独奏试听 | Split render | FR-MX-10、FR-EXP-05 | 覆盖 |
| FX 链/整轨预设 | FX presets | FR-MX-08 | 覆盖 |
| Automation Clip | Automation | FR-AUT-01/02 | 覆盖 |
| Event Automation | Event automation | FR-AUT-03 | 覆盖 |
| Envelope/LFO/Formula/Peak/XY/Keyboard | Controllers | FR-AUT-04、04 工具表 | 覆盖 |
| 参数链接/MIDI Learn/Macro | Link/Macro | FR-AUT-05 | 覆盖 |
| 自动化录制/平滑/简化 | Automation recording | FR-AUT-06/07 | 覆盖 |
| 时间线 BPM/拍号自动化 | Tempo automation | FR-AUT-08 | 覆盖 |

### 19.6 内容、插件与编辑器

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| Browser | Alt+F8 | FR-BR-01～11 | 覆盖 |
| Plugin Picker | F8 | FR-CR-03、FR-INS-01 | 覆盖 |
| 预设/插件数据库/收藏 | Presets/Plugin DB | FR-BR-05/06、FR-INS-04 | 覆盖 |
| 采样试听/波形/元数据 | Browser preview | FR-BR-03/07 | 覆盖 |
| 缺失采样重链 | Missing files | FR-BR-08、FR-PRJ-11 | 覆盖 |
| 54 个插件 Generator + 通道类型 | Instruments | 04 全表 | 覆盖（等价） |
| 原生效果器 | Effects | 05 全表（93 项，含 67/67 原生） | 覆盖（等价） |
| Patcher/模块化 | Patcher | FR-INS-01、04 E13 | 覆盖 |
| Edison/音频编辑 | Ctrl+E | FR-REC-06 | 覆盖 |
| NewTone/NewTime | Pitch/Time editors | FR-REC-08 | 后续 |
| Slicex/Slicer | Slicers | FR-REC-07、04 E08 | 覆盖 |
| Vocoder/Vocodex | Vocoders | 05 2.5 | 覆盖 |
| Gross Beat 类 | Time Warp | 05 3.6 | 覆盖 |
| Maximus | Maximus | 05 3.5 | 覆盖 |
| 视频参考 | Video Player | FR-PL-11、05 工具表 | 后续 |
| 可视化 | ZGameEditor | 05 工具表 | 后续 |

### 19.7 平台能力与非目标

| 功能面 | FL 对应 | 本项目落点 | 状态 |
|---|---|---|---|
| 原生 VST/AU 宿主 | Wrapper | 03 2.2、04 Out | 不做 |
| ReWire/Buzz | ReWired/Buzz Adapter | 04 Out | 不做 |
| 硬件 MIDI 输出 | MIDI Out | FR-REC、04 工具表 | 部分（Web MIDI 限制） |
| Fruity LSD | 旧式 MIDI | 04 Out | 不做 |
| FL Cloud/账号 | FL Cloud | 03 存储、07 开放问题 | 后续 |
| AI 助手（Gopher） | 2026 AI | FR-PR-24、07 P2 | 后续 |
| Stem Separation | 2026 新功能 | FR-REC-09 | 后续 |
| 实时协作 | — | FR-COL-01～07 | 后续 |
| 移动/平板 | FL Mobile | FR-SYS-06、07 P2 | 后续 |
| PWA/离线 | — | NFR、07 Phase 3 | 覆盖 |
| 云备份/分享 | — | FR-PRJ-12、07 P2 | 后续 |
| 无障碍/国际化 | — | FR-ONB-06～08 | 覆盖 |

**审计结论**

- FL 桌面版的**创作主链路**（Channel Rack → Pattern → Piano Roll → Playlist → Mixer → Automation → Export）已逐项覆盖。
- 插件生态以"等价实现"覆盖，原生 VST/AU/ReWire/Buzz 明确不做。
- 2026 新功能（AI、Stem Separation、云）进入 P2，不阻塞 1.0。
- 未覆盖项均为**平台不可行**或**明确非目标**，不存在"忘记做"的核心功能。