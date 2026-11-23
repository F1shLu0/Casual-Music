# 04 · 乐器目录（Instruments Catalog）

> 目标：**覆盖 FL Studio 全部原生 Generator（54 项）的等价能力**，并在此基础上扩展到 **100+ 乐器定义、128 GM 音色、60+ 鼓组**；133 个原生插件的逐项核对见第 8 章附录 A。
> 原则：不复刻商标/UI/官方预设/官方采样；只实现"同类合成/采样能力"。

---

## 1. 设计方法：引擎家族 + 乐器定义

不采用"一个乐器一套独立 DSP"的做法（维护成本爆炸），而是：

```
DSP Engine Family（约 18 个）
        │  参数 schema + 调制路由
        ▼
Instrument Definition（60+，含默认参数、宏、UI、预设库）
        │  + 采样映射 / 波表 / 模型参数
        ▼
Preset（工厂 1,500+ / 用户自定义）
```

好处：

- 新乐器通常只是"新参数默认值 + 新采样映射 + 新 UI"，无需新 DSP。
- 自动化、预设、混音、PDC、离线渲染天然统一。
- GM/管弦/世界乐器可用"采样器家族 + 映射表"批量生成。

### 1.1 引擎家族清单

| ID | 家族 | 核心技术 | 派生乐器（示例） |
|---|---|---|---|
| E01 | 虚拟模拟减法 | 多振荡器 + 零延迟反馈滤波 + 饱和 | 3x Osc、Sawer、PoiZone、Wasp、SimSynth、MiniSynth、Transistor Bass |
| E02 | FM/PM | 多算子、算法矩阵、反馈 | DX10、Toxic、Sytrus(FM 部分)、GMS |
| E03 | 加法/重合成 | 谐波编辑器、FFT 重合成、图像映射 | Harmor、Harmless、Morphine、Ogun、Autogun、BeepMap |
| E04 | 物理建模 | 弦/管/膜/板、激励-共振 | Sakura、Plucked!、Drumaxx、BassDrum、Fruit Kick、Drumpad、DrumSynth Live、FL Slayer |
| E05 | 采样器（单层） | 单采样、循环、起止、反向 | Channel Sampler、Audio Clip、BooBass（采样版） |
| E06 | 多采样器 | 分区、力度层、循环、round-robin、SFZ/SF2 | DirectWave、FL Keys、SoundFont Player、FLEX（采样层） |
| E07 | 鼓采样器 | 16 垫、4 层、力度、 choking、多输出 | FPC、Drumpad（采样模式） |
| E08 | 切片器 | 瞬态检测、切片、触发、time-stretch | Slicex、Fruity Slicer 2 |
| E09 | 颗粒 | 颗粒尺寸/密度/喷雾/位置/冻结 | Fruity Granulizer、Wave Traveller（颗粒模式） |
| E10 | 波表 | 波表集、扫描、变形、 unison | 新增：Wavetable、Spectral Table |
| E11 | 鼓合成 | 桥接 T 网络/正弦+噪声+click | 808/909 类、BassDrum、Fruit Kick |
| E12 | 形式/语音 | 共振峰、音节序列、文本转音 | Speech Synth、Vocal Formant |
| E13 | 模块化 | 节点图、Patcher 式嵌套 | Patcher、Modular Grid |
| E14 |  ROMpler/预设机 | 采样+合成混合、宏 | FLEX、GMS、Kepler/Exo |
| E15 | 弦/弓/吹管专用 | 弓压/气息/滑音/颤音 | 独奏小提琴、萨克斯、笛、二胡 |
| E16 | 合唱/管弦合奏 | 多声部采样 + 交叉淡入 | 弦乐群、铜管群、合唱 |
| E17 | 芯片/复古 | 脉冲/三角/噪声、低位深 | Chiptune 系列 |
| E18 | 纹理/电影 | 颗粒 + 长采样 + 调制 | Drone、Riser、Texture、Granular Choir |

---

## 2. FL Studio 原生 Generator 等价映射（56 项）

> 说明："等价"指功能类别与可控维度相近，不复制其 UI、预设名与采样。

### 2.1 合成器（Synthesis）

| FL 原型 | 本项目乐器 | 引擎 | 关键能力 | 优先级 |
|---|---|---|---|---|
| 3x Osc | Triple Osc | E01 | 3 振荡器、波形/相位/失谐/立体声、滤波、ADSR、LFO | P0 |
| Sawer | Sawer | E01 | 超级锯齿、模拟滤波、Unison、内置延迟/合唱 | P1 |
| PoiZone | PoiZone | E01 | 减法、双滤波、琶音器/步进、调制矩阵 | P1 |
| Wasp | Wasp | E01 | 多振荡器、多模滤波、硬同步 | P1 |
| Wasp XT | Wasp XT | E01 | 扩展振荡器/滤波/调制 | P1 |
| SimSynth | SimSynth | E01 | 双振荡器 + 子振荡、滤波包络 | P1 |
| MiniSynth | MiniSynth | E01 | 移动风格减法、宏控制 | P1 |
| Transistor Bass | Transistor Bass | E01 | 303 式酸音、滑音、 accent、驱动 | P1 |
| Kepler | Kepler | E14 | 复古复音（Juno 风格）合唱/滤波 | P2 |
| Kepler Exo | Kepler Exo | E14 | 复古复音扩展、更多波形/调制 | P2 |
| Fruity DX10 | DX10 | E02 | 2 算子 FM、简易算法、反馈 | P1 |
| Toxic Biohazard | Toxic | E02 | 多算子 FM、滤波、FX、矩阵 | P1 |
| Sytrus | Sytrus | E02/E03 | 6 算子 FM/RM、滤波器、包络/LFO 矩阵、加法谐波 | P0（核心） |
| Groove Machine Synth | GMS Synth | E02 | FM/RM + 采样混合、 groovebox 集成 | P1 |
| Harmor | Harmor | E03 | 加法 + 图像 + 重合成、大规模 unison、频谱操作 | P1（旗舰） |
| Harmless | Harmless | E03 | 加法减法混合、谐波塑形 | P1 |
| Morphine | Morphine | E03 | 4 层加法、重合成、音色 morph | P2 |
| Ogun | Ogun | E03 | 加法谐波生成、金属/钟声 | P2 |
| Autogun | Autogun | E03 | 一键式加法音色生成（受控随机） | P2 |
| BeepMap | BeepMap | E03 | 图像 → 频谱/音色 | P2 |
| Plucked! | Plucked | E04 | Karplus-Strong 拨弦、阻尼/亮度 | P1 |
| Sakura | Sakura | E04/E15 | 弦建模、拨/弓/槌激励、共鸣 | P1 |
| FL Slayer | Slayer | E04 | 电吉他建模、拾音/放大器/箱体 | P1 |
| BassDrum | BassDrum | E04/E11 | 底鼓建模、 pitch/decay/click | P1 |
| Fruit Kick | Fruit Kick | E04/E11 | 底鼓合成、失真/压缩 | P1 |
| Drumaxx | Drumaxx | E04/E11 | 16 垫物理建模鼓 | P1 |
| Drumpad | Drumpad | E04/E11 | 打击垫建模 | P1 |
| Fruity DrumSynth Live | DrumSynth Live | E11 | 多类鼓合成（膜/金属/噪声） | P1 |
| Speech Synth | Speech Synth | E12 | 共振峰语音、音节/音高/语速 | P2 |

### 2.2 采样器与播放（Sampling）

| FL 原型 | 本项目乐器 | 引擎 | 关键能力 | 优先级 |
|---|---|---|---|---|
| Channel Sampler | Sampler | E05 | 单采样、循环、起止、反向、ADSR、Declick | P0 |
| DirectWave | DirectWave | E06 | 多分区/力度层/循环/round-robin、SFZ/SF2 导入 | P1 |
| FL Keys | FL Keys | E06 | 原声钢琴/ Rhodes / Wurli / Clav 等键盘 ROMpler | P1 |
| FPC | FPC | E07 | 16 垫、4 层、力度、choke、多输出 | P1 |
| SoundFont Player | SoundFont Player | E06 | SF2 播放、GM 映射 | P1 |
| FLEX | FLEX | E14/E06 | 预设机、宏、采样+合成层 | P1 |
| Slicex | Slicex | E08 | 自动切片、逐片参数、time-stretch、多输出 | P1 |
| Fruity Slicer 2 | Slicer 2 | E08 | 切片触发、循环、简化编辑 | P1 |
| Fruity Granulizer | Granulizer | E09 | 颗粒尺寸/密度/位置/喷雾/冻结 | P1 |
| Wave Traveller | Wave Traveller | E09 | 刮擦/变形/颗粒、实时控制 | P2 |
| BooBass | BooBass | E05/E06 | 电贝斯采样/建模混合 | P1 |
| Groove Machine | Groove Machine | E07/E02 | 鼓机/采样/合成 groovebox | P1 |
| FL Studio Mobile Rack | Mobile Rack | E14/E06 | 移动端乐器/效果机架、轻量预设 | P2 |
| GMS | GMS | E14 | GM 128 音色 + 鼓组 | P0（GM） |
| Audio Clip | Audio Clip | E05 | 音频剪辑通道、time-stretch | P0 |
| Automation Clip | Automation Clip | — | 自动化曲线（非乐器） | P0 |

### 2.3 工具、控制器与模块化（Tools / Controllers）

| FL 原型 | 本项目 | 说明 | 优先级 |
|---|---|---|---|
| Patcher | Patcher | 节点式乐器/效果嵌套、宏、多输出 | P1 |
| Synthmaker (Flowstone) | Modular Grid | 内置模块化（不做用户脚本编译器） | P2 |
| Fruity Envelope Controller | Envelope Controller | 多段包络作为调制源 | P1 |
| Fruity Keyboard Controller | Keyboard Controller | 键盘位置/力度/释放作为调制源 | P1 |
| Fruity Peak Controller | Peak Controller | 音频包络跟随作为调制源 | P1 |
| Fruity Formula Controller | Formula Controller | 公式生成控制信号（受控表达式） | P1 |
| Fruity X-Y Controller | XY Controller | 二维控制器、可录制 | P1 |
| Dashboard | Dashboard | 自定义控制面板 | P2 |
| Control Surface | Control Surface | 自定义控制器布局 | P2 |
| VFX Sequencer | VFX Sequencer | 高级步进/琶音序列器 | P1 |
| Layer | Layer | 多层叠加、独奏层、交叉淡入 | P1 |
| MIDI Out | MIDI Out | 外部 MIDI 设备/软件 | P2 |
| Fruity LSD | Fruity LSD | 旧式 MIDI 输出/兼容工具（Web 无对应，标记 Out） | Out |
| ReWired | — | 不复刻（Web 无 ReWire） | Out |
| Fruity Video Player | Video Sync | 视频参考同步（非乐器） | P2 |
| Fruity Dance | — | 不复刻（纯视觉彩蛋） | Out |
| Fruity Vibrator | — | 不复刻（硬件触觉） | Out |
| Buzz Generator Adapter | — | 不复刻（Buzz 生态） | Out |
| Wrapper (VST/AU) | — | 不宿主原生插件；提供"渲染为采样" | Out |

**覆盖统计**：功能等价 48 项；明确不复刻 6 项（ReWired、Dance、Vibrator、Buzz、Wrapper、Flowstone 编译器）；纯工具/非乐器 2 项（Video、Automation）。核心乐器能力无缺口。

---

## 3. 乐器扩展库（本项目增量）

> 目标：让"乐器尽可能全面、种类多"。以下为 60+ 乐器定义，按演奏法与音色族组织。

### 3.1 键盘类（Keys）

| 乐器 | 引擎 | 说明 | 采样/合成 | 优先级 |
|---|---|---|---|---|
| Concert Grand | E06 | 9 层力度、踏板共鸣、半踏 | 采样（Salamander 类） | P0 |
| Upright Piano | E06 | 立式钢琴、机械噪声层 | 采样 | P1 |
| Felt Piano | E06 | 毛毡钢琴、柔和 | 采样 | P1 |
| Prepared Piano | E06 | 预制钢琴（螺丝/纸片） | 采样 | P2 |
| Rhodes EP | E06/E01 | 电钢琴、tine 层、颤音 | 采样+合成 | P0 |
| Wurlitzer EP | E06 | 簧片电钢、过载 | 采样 | P1 |
| CP-70 EP | E06 | 电钢琴、明亮 | 采样 | P2 |
| FM Electric Piano | E02 | DX7 风格 FM 电钢 | 合成 | P1 |
| Clavinet | E06 | 击弦 clav、放克 | 采样 | P1 |
| Harpsichord | E06 | 拨弦羽管键琴、双排 | 采样 | P1 |
| Celesta | E06 | 钢片琴 | 采样 | P1 |
| Toy Piano | E06 | 玩具钢琴 | 采样 | P2 |
| Accordion | E06/E15 | 手风琴、风箱/簧片 | 采样 | P2 |
| Harmonium | E06/E15 | 印度风琴、持续簧片 | 采样 | P2 |
| Melodica | E15 |  melodica 吹奏 | 采样 | P2 |

### 3.2 吉他/拨弦（Guitars & Plucked）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| Acoustic Steel Guitar | E06 | 钢弦木吉他、指弹/扫弦 | P1 |
| Acoustic Nylon Guitar | E06 | 尼龙弦、古典 | P1 |
| 12-String Guitar | E06 | 十二弦、宽 stereo | P2 |
| Electric Clean Guitar | E06/E04 | 清音电吉他、拾音切换 | P1 |
| Electric Crunch Guitar | E06/E04 | 失真节奏 | P1 |
| Electric Lead Guitar | E04/E06 | 独奏、揉弦/推弦 | P1 |
| Bass Finger | E06 | 指弹电贝斯 | P1 |
| Bass Pick | E06 | 拨片贝斯 | P1 |
| Bass Slap | E06 | Slap 贝斯、泛音 | P2 |
| Upright Bass | E06/E04 | 爵士低音提琴 | P1 |
| Ukulele | E06 | 尤克里里 | P2 |
| Banjo | E06 | 班卓琴 | P2 |
| Mandolin | E06 | 曼陀林 | P2 |
| Sitar | E06/E15 | 西塔琴、共鸣弦/滑音 | P1 |
| Guzheng | E06/E15 | 古筝、按音/颤音 | P1 |
| Pipa | E06/E15 | 琵琶、轮指 | P2 |
| Koto | E06/E15 | 日本筝 | P2 |
| Harp | E06 | 竖琴、滑奏 | P1 |
| Kalimba | E06 | 拇指琴 | P1 |
| Marimba | E06 | 马林巴 | P1 |
| Vibraphone | E06 | 颤音琴、马达 | P1 |
| Glockenspiel | E06 | 钟琴 | P1 |
| Steel Drum | E06 | 钢鼓 | P2 |

### 3.3 管弦：弦乐（Orchestral Strings）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| String Ensemble | E16 | 弦乐群、分奏、长/短弓 | P1 |
| Solo Violin | E15 | 独奏小提琴、连奏/颤音/拨弦 | P1 |
| Solo Viola | E15 | 中提琴 | P1 |
| Solo Cello | E15 | 大提琴 | P1 |
| Double Bass | E15/E06 | 低音提琴 | P1 |
| String Quartet | E16 | 四重奏合奏预设 | P2 |
| Tremolo Strings | E16 | 颤弓 | P2 |
| Pizzicato Strings | E16 | 拨弦 | P2 |
| Spiccato Strings | E16 | 跳弓 | P2 |

### 3.4 管弦：铜管（Brass）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| Trumpet | E15 | 小号、弱音器 | P1 |
| Trombone | E15 | 长号、滑管 | P1 |
| French Horn | E15 | 圆号 | P1 |
| Tuba | E15 | 大号 | P1 |
| Brass Ensemble | E16 | 铜管群、swell | P1 |
| Muted Trumpet | E15 | 弱音小号 | P2 |
| Flugelhorn | E15 | 柔音号 | P2 |

### 3.5 管弦：木管（Woodwinds）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| Flute | E15 | 长笛、气息噪声/超吹 | P1 |
| Piccolo | E15 | 短笛 | P1 |
| Clarinet | E15 | 单簧管 | P1 |
| Oboe | E15 | 双簧管 | P1 |
| Bassoon | E15 | 巴松 | P1 |
| Alto Sax | E15 | 中音萨克斯 | P1 |
| Tenor Sax | E15 | 次中音萨克斯 | P1 |
| Bari Sax | E15 | 上低音萨克斯 | P2 |
| Soprano Sax | E15 | 高音萨克斯 | P2 |
| Recorder | E15 | 竖笛 | P2 |
| Pan Flute | E15 | 排箫 | P2 |
| Dizi | E15 | 中国笛 | P1 |
| Shakuhachi | E15 | 尺八 | P2 |
| Bagpipes | E15/E16 | 风笛、持续音管 | P2 |

### 3.6 打击与鼓组（Drums & Percussion）

| 类别 | 内容 | 优先级 |
|---|---|---|
| 原声鼓组 | Rock、Pop、Jazz、Funk、Brush、Studio、Vintage | P0 |
| 电子鼓组 | 808、909、707、CR-78、LinnDrum、DMX、Simmons | P0 |
| 混合鼓组 | Acoustic + Electronic layered、Lo-Fi、Tape | P1 |
| 管弦打击 | Timpani、Snare、Bass Drum、Cymbals、Glockenspiel、Xylophone、Tubular Bells | P1 |
| 世界打击 | Conga、Bongo、Djembe、Taiko、Frame Drum、Cajon、Tabla、Darbuka、Shaker、Tambourine、Cowbell、Agogo、Woodblock | P1 |
| 效果打击 | Clap、Snap、Stomp、Metal、Industrial、Glitch | P1 |
| 鼓组总量目标 | 60+ kits，每 kit 16–64 件 | — |

### 3.7 世界乐器（World）

| 乐器 | 引擎 | 优先级 |
|---|---|---|
| Tabla | E06/E15 | P1 |
| Erhu | E15 | P1 |
| Dizi | E15 | P1 |
| Guzheng | E06/E15 | P1 |
| Pipa | E06/E15 | P2 |
| Sitar | E06/E15 | P1 |
| Sarangi | E15 | P2 |
| Bansuri | E15 | P2 |
| Koto | E06/E15 | P2 |
| Shakuhachi | E15 | P2 |
| Gamelan | E06/E16 | P2 |
| Didgeridoo | E15 | P2 |
| Kalimba | E06 | P1 |
| Steel Drum | E06 | P2 |
| Bandoneon | E06/E15 | P2 |
| Charango | E06 | P2 |
| Mbira | E06 | P2 |
| Duduk | E15 | P2 |
| Oud | E06 | P2 |
| Qanun | E06 | P2 |

### 3.8 人声与合唱（Voices）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| Solo Voice Ah | E06/E16 | 元音采样、连奏 | P1 |
| Solo Voice Ooh | E06/E16 | 柔和元音 | P1 |
| Choir | E16 | SATB 合唱、元音切换 | P1 |
| Vocal Chops | E06 | 切片式人声短句（授权采样） | P2 |
| Vocal Formant Synth | E12 | 共振峰合成、元音 morph | P1 |
| Speech Synth | E12 | 文本/音节转语音 | P2 |
| Vocoder | 效果 | 见效果器目录 | P1 |
| Beatbox Kit | E07 | 口技鼓组（授权采样） | P2 |

### 3.9 合成器扩展（Original Synths）

| 乐器 | 引擎 | 说明 | 优先级 |
|---|---|---|---|
| Wavetable | E10 | 波表扫描/变形、unison | P1 |
| Spectral Pad | E10/E03 | 频谱纹理、慢速演化 | P2 |
| Granular Choir | E09/E16 | 颗粒合唱/纹理 | P2 |
| Analog Mono | E01 | 单音贝斯/Lead、滑音 | P1 |
| Acid Bass | E01 | 303 风格 | P1 |
| Reese Bass | E01/E10 | 双锯失谐、滤波运动 | P1 |
| Sub Bass | E01 | 纯净低频、谐波控制 | P1 |
| Supersaw Lead | E01 | 多层锯齿、宽 stereo | P1 |
| Trance Gate Pad | E01/E10 | 门限节奏 pad | P1 |
| FM Bass | E02 | FM 低音 | P1 |
| FM Bell | E02 | 钟声/金属 | P1 |
| Additive Organ | E03 | 拉杆风琴式加法 | P2 |
| Chip Pulse | E17 | 方波/脉冲、低位深 | P1 |
| Chip Triangle | E17 | 三角波、NES 风格 | P1 |
| Chip Noise | E17 | 噪声鼓/效果 | P1 |
| FM Chip | E17/E02 | 复古 FM 芯片 | P2 |
| Drone | E18 | 长音纹理、缓慢调制 | P2 |
| Riser | E18 | 上升噪声/调性 riser | P2 |
| Impact | E18 | 电影撞击/低频 | P2 |
| Texture | E18 | 颗粒环境纹理 | P2 |

### 3.10 GM 与通用音色

- **GM 128**：按 GM 标准分组（Piano、Chromatic Percussion、Organ、Guitar、Bass、Strings、Ensemble、Brass、Reed、Pipe、Synth Lead、Synth Pad、Synth Effects、Ethnic、Percussive、Sound Effects）。
- **GM 鼓组**：Standard、Room、Power、Electronic、TR-808、Jazz、Brush、Orchestra、SFX。
- 实现方式：E06 多采样器 + GM 映射表；缺失音色用合成引擎兜底，保证 128 个 Program 全部可发声。
- 优先级：P0（"128 音色全部可发声"是硬指标）。

---

## 4. 每个乐器的标准规格

为避免"只有名字没有能力"，每个乐器定义必须包含：

| 项 | 要求 |
|---|---|
| 基本信息 | 名称、类别、标签、作者、版本、许可证 |
| 引擎与 DSP | 家族 ID、复音策略、CPU 等级 |
| 参数 | ID、名称、范围、单位、曲线、默认、可自动化、调制目标 |
| 宏 | 4–8 个宏，映射到核心参数 |
| 预设 | 工厂 ≥10（旗舰 ≥50），用户可存 |
| 演奏法 | 连奏/滑音/颤音/力度层/round-robin（适用时） |
| 采样 | 映射表、循环点、力度层、授权来源 |
| UI | 通用 Schema UI 或自定义 Canvas UI；可缩放 |
| 输出 | 单/多输出、Mixer 路由建议 |
| 测试 | 基准工程、黄金音频、CPU 预算 |

---

## 5. 预设与内容目标

| 内容 | 1.0 目标 |
|---|---|
| 乐器定义 | ≥100（含 54 个 Generator 等价 + 扩展乐器） |
| 引擎家族 | 18 |
| 工厂预设 | ≥1,500 |
| GM Program | 128 全部可发声 |
| 鼓组 | ≥60 |
| 效果器 | ≥90（覆盖全部原生效果器，见 `05-effects-catalog.md`） |
| 采样库 | 核心包 ≤300MB；扩展包按需下载 |
| 教程工程 | ≥10 |
| 模板 | ≥12 |

### 5.1 采样包策略

| 包 | 内容 | 估算大小 | 加载方式 |
|---|---|---|---|
| Core | 3x Osc 波表、基础鼓组、GM 精简、钢琴/EP | 200–300MB | 首装/首次使用 |
| Drums | 60 kits 多层 | 300–500MB | 按需 |
| Orchestral | 弦/铜/木管/打击 | 800MB–1.5GB | 按需，提供 Lite 版 |
| World | 世界乐器 | 200–400MB | 按需 |
| Keys | 钢琴/电钢/键盘扩展 | 300–600MB | 按需 |
| Chiptune | 芯片波表/鼓 | <20MB | 内置 |
| GM Full | 完整 GM 采样 | 200–400MB | 按需 |

> 所有包支持"Lite/Full"两档，Lite 用短采样+循环，降低首屏与移动端压力。

---

## 6. 复刻优先级与里程碑

### M1（MVP，P0 乐器）

1. Sampler（E05）
2. Triple Osc（E01）
3. Drum Kit 808/909（E07）
4. GM Lite（E06，128 Program 可发声）
5. Audio Clip（E05）
6. Piano（E06，Salamander 类）
7. Sytrus 核心（E02，FM 6 算子）
8. Automation/Envelope Controller（调制基础）

### M2（v0.5）

- FPC、Slicex、Granulizer、DirectWave、FL Keys、Sawer、PoiZone、Transistor Bass、Plucked、Sakura、Drumaxx、BassDrum、Fruit Kick。
- 弦乐群、铜管群、木管独奏、合唱、竖琴、Marimba、Kalimba。
- 60 鼓组中的 30。

### M3（v1.0）

- Harmor/Harmless、Morphine/Ogun、Toxic、Wasp、SimSynth、Kepler。
- 世界乐器 20、管弦全量、GM Full。
- Patcher、Modular Grid、VFX Sequencer、Formula/Peak/XY Controller。

### M4（v1.x）

- 重合成/图像合成、Speech、Wave Traveller、Granular Choir、Drone/Riser/Impact。
- 第三方 WASM 乐器 SDK。

---

## 7. 质量与合规

- **不采样 FL Studio 原生内容**；所有采样必须来自 CC0/CC BY/明确商用授权，并记录来源、许可证、哈希。
- 乐器名称避免与 FL 商标混淆；"Sytrus 等价"在 UI 中使用自有名称（如 "Hex FM"）。
- 预设不得复制官方预设参数与名称；工厂预设自行设计。
- 采样包提供"许可证"页与"替换/删除"能力，满足分发合规。
- 每个乐器提供 CPU 等级标签（Light/Standard/Heavy），便于用户控制工程负载。

---

## 8. 附录 A：133 个原生插件全量核对表

> 核对依据：`references/raw/plugin_descriptions.tsv`（官方 133 个插件页面条目）与 `references/raw/plugins/`。
> 分类按官方手册分组：Generator 54 / Effect 67 / Controller 5 / Editor-Tool 7。
> 状态：**等价**＝本项目有对应实现；**Out**＝明确不复刻并给出替代；**工具**＝编辑器/辅助工具。

### 8.1 Generator（54 项）

| # | FL 插件 | 本项目 | 引擎 | 优先级 | 状态 |
|---|---|---|---|---|---|
| 1 | 3x Osc | Triple Osc | E01 | P0 | 等价 |
| 2 | Autogun | Autogun | E03 | P2 | 等价 |
| 3 | BassDrum | BassDrum | E04/E11 | P1 | 等价 |
| 4 | BeepMap | BeepMap | E03 | P2 | 等价 |
| 5 | BooBass | BooBass | E05/E06 | P1 | 等价 |
| 6 | Control Surface | Control Surface | — | P2 | 等价（自定义控制器） |
| 7 | Dashboard | Dashboard | — | P2 | 等价（自定义面板） |
| 8 | DirectWave | DirectWave | E06 | P1 | 等价 |
| 9 | Drumaxx | Drumaxx | E04/E11 | P1 | 等价 |
| 10 | Drumpad | Drumpad | E04/E11 | P1 | 等价 |
| 11 | FL Keys | FL Keys | E06 | P1 | 等价 |
| 12 | FL Slayer | Slayer | E04 | P1 | 等价 |
| 13 | FL Studio Mobile Rack | Mobile Rack | E14/E06 | P2 | 等价（轻量机架） |
| 14 | FLEX | FLEX | E14/E06 | P1 | 等价 |
| 15 | FPC | FPC | E07 | P1 | 等价 |
| 16 | Fruit Kick | Fruit Kick | E04/E11 | P1 | 等价 |
| 17 | Fruity Dance | — | — | — | Out（纯视觉彩蛋） |
| 18 | Fruity DrumSynth Live | DrumSynth Live | E11 | P1 | 等价 |
| 19 | Fruity DX10 | DX10 | E02 | P1 | 等价 |
| 20 | Fruity Granulizer | Granulizer | E09 | P1 | 等价 |
| 21 | Fruity LSD | — | — | — | Out（旧式 MIDI） |
| 22 | Fruity Slicer | Slicer | E08 | P1 | 等价 |
| 23 | Fruity Slicer 2 | Slicer 2 | E08 | P1 | 等价 |
| 24 | Fruity Vibrator | — | — | — | Out（硬件触觉） |
| 25 | Fruity Video Player | Video Sync | — | P2 | 等价（参考视频同步） |
| 26 | GMS | GMS | E14 | P0 | 等价 |
| 27 | Groove Machine | Groove Machine | E07/E02 | P1 | 等价 |
| 28 | Harmless | Harmless | E03 | P1 | 等价 |
| 29 | Harmor | Harmor | E03 | P1 | 等价 |
| 30 | Kepler | Kepler | E14 | P2 | 等价 |
| 31 | Kepler Exo | Kepler Exo | E14 | P2 | 等价 |
| 32 | MIDI Out | MIDI Out | — | P2 | 部分（Web MIDI 输出） |
| 33 | MiniSynth | MiniSynth | E01 | P1 | 等价 |
| 34 | Morphine | Morphine | E03 | P2 | 等价 |
| 35 | Ogun | Ogun | E03 | P2 | 等价 |
| 36 | Patcher | Patcher | E13 | P1 | 等价 |
| 37 | Plucked! | Plucked | E04 | P1 | 等价 |
| 38 | PoiZone | PoiZone | E01 | P1 | 等价 |
| 39 | ReWired | — | — | — | Out（Web 无 ReWire） |
| 40 | Sakura | Sakura | E04/E15 | P1 | 等价 |
| 41 | Sawer | Sawer | E01 | P1 | 等价 |
| 42 | SimSynth | SimSynth | E01 | P1 | 等价 |
| 43 | Slicex | Slicex | E08 | P1 | 等价 |
| 44 | SoundFont Player | SoundFont Player | E06 | P1 | 等价 |
| 45 | Speech Synth | Speech Synth | E12 | P2 | 等价（Web Speech/共振峰） |
| 46 | Synthmaker | Modular Grid | E13 | P2 | 等价（不含用户脚本编译器） |
| 47 | Sytrus | Sytrus | E02/E03 | P0 | 等价 |
| 48 | Toxic Biohazard | Toxic | E02 | P1 | 等价 |
| 49 | Transistor Bass | Transistor Bass | E01 | P1 | 等价 |
| 50 | VFX Sequencer | VFX Sequencer | — | P1 | 等价 |
| 51 | Wasp | Wasp | E01 | P1 | 等价 |
| 52 | Wasp XT | Wasp XT | E01 | P1 | 等价 |
| 53 | Wave Traveller | Wave Traveller | E09 | P2 | 等价 |
| 54 | Wrapper (VST/AU) | — | — | — | Out（不宿主原生插件，提供"渲染为采样"） |

**非插件通道类型（不计入 133）**：Layer（P1，见 2.3）、Automation Clip（P0）、Audio Clip（P0）、Sampler（P0）。

### 8.2 Effect（67 项）

| # | FL 插件 | 本项目 | 优先级 | 状态 |
|---|---|---|---|---|
| 1 | Distructor | Distructor | P1 | 等价 |
| 2 | Effector | Effector | P1 | 等价 |
| 3 | Emphasis | Emphasis | P2 | 等价 |
| 4 | Emphasizer | Emphasizer | P2 | 等价 |
| 5 | EQUO | EQUO | P1 | 等价 |
| 6 | Frequency Shifter | Frequency Shifter | P1 | 等价 |
| 7 | Frequency Splitter | Frequency Splitter | P1 | 等价 |
| 8 | Fruity 7 Band EQ | 7-Band EQ | P1 | 等价 |
| 9 | Fruity Balance | Balance | P0 | 等价 |
| 10 | Fruity Bass Boost | Bass Boost | P2 | 等价 |
| 11 | Fruity Big Clock | Big Clock | P2 | 等价 |
| 12 | Fruity Blood Overdrive | Blood Overdrive | P1 | 等价 |
| 13 | Fruity Center | Center | P2 | 等价 |
| 14 | Fruity Chorus | Chorus | P1 | 等价 |
| 15 | Fruity Compressor | Compressor | P1 | 等价 |
| 16 | Fruity Convolver | Convolver | P1 | 等价 |
| 17 | Fruity dB Meter | dB Meter | P1 | 等价 |
| 18 | Fruity Delay | Delay | P1 | 等价 |
| 19 | Fruity Delay 2 | Delay 2 | P1 | 等价 |
| 20 | Fruity Delay 3 | Delay 3 | P1 | 等价 |
| 21 | Fruity Delay Bank | Delay Bank | P2 | 等价 |
| 22 | Fruity Fast Dist | Fast Dist | P1 | 等价 |
| 23 | Fruity Fast LP | Fast LP | P1 | 等价 |
| 24 | Fruity Filter | Multi-Mode Filter | P1 | 等价 |
| 25 | Fruity Flanger | Flanger | P1 | 等价 |
| 26 | Fruity Flangus | Flangus | P1 | 等价 |
| 27 | Fruity Free Filter | Free Filter | P1 | 等价 |
| 28 | Fruity Limiter | Limiter | P0 | 等价 |
| 29 | Fruity Love Philter | Love Philter | P1 | 等价 |
| 30 | Fruity Multiband Compressor | Multiband Compressor | P1 | 等价 |
| 31 | Fruity Mute 2 | Mute 2 | P1 | 等价 |
| 32 | Fruity PanOMatic | PanOMatic | P2 | 等价 |
| 33 | Fruity Parametric EQ | Parametric EQ 6 | P1 | 等价 |
| 34 | Fruity Parametric EQ 2 | Parametric EQ 8 | P0 | 等价 |
| 35 | Fruity Phase Inverter | Phase Inverter | P1 | 等价 |
| 36 | Fruity Phaser | Phaser | P1 | 等价 |
| 37 | Fruity Reeverb | Reverb | P1 | 等价 |
| 38 | Fruity Reeverb 2 | Reverb 2 | P0 | 等价 |
| 39 | Fruity Scratcher | Scratcher | P2 | 等价（黑胶刮擦） |
| 40 | Fruity Send | Send | P0 | 等价 |
| 41 | Fruity Soft Clipper | Soft Clipper | P1 | 等价 |
| 42 | Fruity Spectroman | Spectrum Analyzer | P1 | 等价 |
| 43 | Fruity Squeeze | Squeeze | P1 | 等价 |
| 44 | Fruity Stereo Enhancer | Stereo Enhancer | P1 | 等价 |
| 45 | Fruity Stereo Shaper | Stereo Shaper | P1 | 等价 |
| 46 | Fruity Vocoder | Vocoder | P1 | 等价 |
| 47 | Fruity WaveShaper | WaveShaper | P1 | 等价 |
| 48 | Gross Beat | Time Warp | P1 | 等价 |
| 49 | Hardcore | Hardcore | P1 | 等价 |
| 50 | Hyper Chorus | Hyper Chorus | P1 | 等价 |
| 51 | Low Lifter | Low Lifter | P2 | 等价 |
| 52 | LuxeVerb | LuxeVerb | P1 | 等价 |
| 53 | Maximus | Maximus | P1 | 等价 |
| 54 | Multiband Delay | Multiband Delay | P1 | 等价 |
| 55 | Pitch Shifter | Pitch Shifter | P1 | 等价 |
| 56 | Pitcher | Pitcher | P1 | 等价 |
| 57 | Soundgoodizer | Soundgoodizer | P1 | 等价 |
| 58 | Spreader | Spreader | P1 | 等价 |
| 59 | Transient Processor | Transient Processor | P1 | 等价 |
| 60 | Transmitter | Transient Splitter | P1 | 等价 |
| 61 | Transporter | Transporter | P2 | 等价 |
| 62 | Tuner | Tuner | P2 | 等价 |
| 63 | Vintage Chorus | Vintage Chorus | P1 | 等价 |
| 64 | Vintage Phaser | Vintage Phaser | P1 | 等价 |
| 65 | Vocodex | Vocodex | P1 | 等价 |
| 66 | Wave Candy | Oscilloscope | P1 | 等价 |
| 67 | ZGameEditor Visualizer | Visualizer | P2 | 等价（可选导出） |

### 8.3 Controller（5 项）

| # | FL 插件 | 本项目 | 优先级 | 状态 |
|---|---|---|---|---|
| 1 | Fruity Envelope Controller | Envelope Controller | P1 | 等价 |
| 2 | Fruity Formula Controller | Formula Controller | P1 | 等价（受控表达式，禁 eval） |
| 3 | Fruity Keyboard Controller | Keyboard Controller | P1 | 等价 |
| 4 | Fruity Peak Controller | Peak Controller | P1 | 等价 |
| 5 | Fruity X-Y Controller | XY Controller | P1 | 等价 |

### 8.4 Editor / Tool（7 项）

| # | FL 插件 | 本项目 | 优先级 | 状态 |
|---|---|---|---|---|
| 1 | Edison | Audio Editor | P1 | 等价 |
| 2 | Newtone | Pitch Editor | P2 | 等价 |
| 3 | Newtime | Time Editor | P2 | 等价 |
| 4 | Fruity NoteBook | NoteBook | P2 | 等价 |
| 5 | Fruity HTML NoteBook | HTML NoteBook | P2 | 等价（沙箱渲染） |
| 6 | editortool_equalize | EQ Tool（Edison 内） | P2 | 等价 |
| 7 | editortool_reverb | Reverb Tool（Edison 内） | P2 | 等价 |

### 8.5 核对统计

| 类别 | 官方条目 | 等价/部分 | Out | 覆盖率（不含 Out） |
|---|---:|---:|---:|---:|
| Generator | 54 | 49（48 等价 + 1 部分） | 5（Dance、LSD、Vibrator、ReWired、Wrapper） | 90.7% |
| Effect | 67 | 67 | 0 | 100% |
| Controller | 5 | 5 | 0 | 100% |
| Editor / Tool | 7 | 7 | 0 | 100% |
| **合计** | **133** | **128** | **5** | **96.2%** |

> 说明：`MIDI Out` 记为"部分"（Web MIDI 输出能力受浏览器与系统限制）。5 项 Out 均为平台不可行（ReWire/原生 VST 宿主/硬件触觉/旧式 MIDI/纯视觉彩蛋），不影响创作主链路。
> 上述 133 项之外，本项目还**新增**若干等价体系没有的效果器与扩展乐器（见 `05` 2.1–2.7 中标"（新增）"的条目与本章 3.1–3.9），因此最终插件数量大于 133。

---

## 9. 附录 B：GM 128 音色映射表

> 硬指标：**128 个 Program 全部可发声**，不依赖用户额外下载。
> 实现：`E06 多采样器 + GM 映射表`；缺采样时由合成引擎兜底（保证"有声音"而非"空通道"）。
> 采样来源：GeneralUser GS（可用）/ 自有采样；**不使用** FL Studio 或官方 SoundFont。

### 9.1 钢琴（1–8）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 1 | Acoustic Grand Piano | Concert Grand | E06 | 采样 |
| 2 | Bright Acoustic Piano | Bright Piano | E06 | 采样 |
| 3 | Electric Grand Piano | CP-70 EP | E06 | 采样 |
| 4 | Honky-tonk Piano | Honky-Tonk | E06 | 采样 |
| 5 | Electric Piano 1 | Rhodes EP | E06/E01 | 采样+合成 |
| 6 | Electric Piano 2 | FM Electric Piano | E02 | 合成 |
| 7 | Harpsichord | Harpsichord | E06 | 采样 |
| 8 | Clavinet | Clavinet | E06 | 采样 |

### 9.2 色彩打击（9–16）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 9 | Celesta | Celesta | E06 | 采样 |
| 10 | Glockenspiel | Glockenspiel | E06 | 采样 |
| 11 | Music Box | Music Box | E06/E04 | 采样 |
| 12 | Vibraphone | Vibraphone | E06 | 采样 |
| 13 | Marimba | Marimba | E06 | 采样 |
| 14 | Xylophone | Xylophone | E06 | 采样 |
| 15 | Tubular Bells | Tubular Bells | E06/E04 | 采样 |
| 16 | Dulcimer | Dulcimer | E06/E15 | 采样 |

### 9.3 风琴（17–24）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 17 | Drawbar Organ | Additive Organ | E03 | 合成 |
| 18 | Percussive Organ | Percussive Organ | E03 | 合成 |
| 19 | Rock Organ | Rock Organ | E03/E01 | 合成 |
| 20 | Church Organ | Church Organ | E06/E03 | 采样 |
| 21 | Reed Organ | Harmonium | E06/E15 | 采样 |
| 22 | Accordion | Accordion | E06/E15 | 采样 |
| 23 | Harmonica | Harmonica | E06/E15 | 采样 |
| 24 | Tango Accordion | Bandoneon | E06/E15 | 采样 |

### 9.4 吉他（25–32）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 25 | Acoustic Guitar (nylon) | Acoustic Nylon Guitar | E06 | 采样 |
| 26 | Acoustic Guitar (steel) | Acoustic Steel Guitar | E06 | 采样 |
| 27 | Electric Guitar (jazz) | Electric Clean Guitar | E06/E04 | 采样 |
| 28 | Electric Guitar (clean) | Electric Clean Guitar | E06/E04 | 采样 |
| 29 | Electric Guitar (muted) | Muted Guitar | E06/E04 | 采样 |
| 30 | Overdriven Guitar | Crunch Guitar | E06/E04 | 采样 |
| 31 | Distortion Guitar | Lead Guitar | E04/E06 | 采样 |
| 32 | Guitar Harmonics | Guitar Harmonics | E04 | 建模 |

### 9.5 贝斯（33–40）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 33 | Acoustic Bass | Upright Bass | E06/E04 | 采样 |
| 34 | Electric Bass (finger) | Bass Finger | E06 | 采样 |
| 35 | Electric Bass (pick) | Bass Pick | E06 | 采样 |
| 36 | Fretless Bass | Fretless Bass | E06/E15 | 采样 |
| 37 | Slap Bass 1 | Bass Slap | E06 | 采样 |
| 38 | Slap Bass 2 | Bass Slap 2 | E06 | 采样 |
| 39 | Synth Bass 1 | Analog Mono | E01 | 合成 |
| 40 | Synth Bass 2 | Sub Bass | E01 | 合成 |

### 9.6 弦乐（41–48）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 41 | Violin | Solo Violin | E15 | 采样 |
| 42 | Viola | Solo Viola | E15 | 采样 |
| 43 | Cello | Solo Cello | E15 | 采样 |
| 44 | Contrabass | Double Bass | E15/E06 | 采样 |
| 45 | Tremolo Strings | Tremolo Strings | E16 | 采样 |
| 46 | Pizzicato Strings | Pizzicato Strings | E16 | 采样 |
| 47 | Orchestral Harp | Harp | E06 | 采样 |
| 48 | Timpani | Timpani | E06 | 采样 |

### 9.7 合奏（49–56）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 49 | String Ensemble 1 | String Ensemble | E16 | 采样 |
| 50 | String Ensemble 2 | String Ensemble (alt) | E16 | 采样 |
| 51 | Synth Strings 1 | Synth Strings | E01/E16 | 合成 |
| 52 | Synth Strings 2 | Synth Strings (alt) | E01/E16 | 合成 |
| 53 | Choir Aahs | Choir | E16 | 采样 |
| 54 | Voice Oohs | Solo Voice Ooh | E06/E16 | 采样 |
| 55 | Synth Voice | Vocal Formant Synth | E12 | 合成 |
| 56 | Orchestra Hit | Orchestra Hit | E06 | 采样 |

### 9.8 铜管（57–64）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 57 | Trumpet | Trumpet | E15 | 采样 |
| 58 | Trombone | Trombone | E15 | 采样 |
| 59 | Tuba | Tuba | E15 | 采样 |
| 60 | Muted Trumpet | Muted Trumpet | E15 | 采样 |
| 61 | French Horn | French Horn | E15 | 采样 |
| 62 | Brass Section | Brass Ensemble | E16 | 采样 |
| 63 | Synth Brass 1 | Synth Brass | E01 | 合成 |
| 64 | Synth Brass 2 | Synth Brass (alt) | E01 | 合成 |

### 9.9 簧管（65–72）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 65 | Soprano Sax | Soprano Sax | E15 | 采样 |
| 66 | Alto Sax | Alto Sax | E15 | 采样 |
| 67 | Tenor Sax | Tenor Sax | E15 | 采样 |
| 68 | Baritone Sax | Bari Sax | E15 | 采样 |
| 69 | Oboe | Oboe | E15 | 采样 |
| 70 | English Horn | English Horn | E15 | 采样 |
| 71 | Bassoon | Bassoon | E15 | 采样 |
| 72 | Clarinet | Clarinet | E15 | 采样 |

### 9.10 吹管（73–80）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 73 | Piccolo | Piccolo | E15 | 采样 |
| 74 | Flute | Flute | E15 | 采样 |
| 75 | Recorder | Recorder | E15 | 采样 |
| 76 | Pan Flute | Pan Flute | E15 | 采样 |
| 77 | Blown Bottle | Blown Bottle | E15/E18 | 采样 |
| 78 | Shakuhachi | Shakuhachi | E15 | 采样 |
| 79 | Whistle | Whistle | E15 | 采样 |
| 80 | Ocarina | Ocarina | E15 | 采样 |

### 9.11 合成主音（81–88）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 81 | Lead 1 (square) | Chip Pulse | E17 | 合成 |
| 82 | Lead 2 (sawtooth) | Supersaw Lead | E01 | 合成 |
| 83 | Lead 3 (calliope) | Calliope Lead | E01 | 合成 |
| 84 | Lead 4 (chiff) | Chiff Lead | E01/E18 | 合成 |
| 85 | Lead 5 (charang) | Charang Lead | E01 | 合成 |
| 86 | Lead 6 (voice) | Voice Lead | E12/E01 | 合成 |
| 87 | Lead 7 (fifths) | Fifths Lead | E01 | 合成 |
| 88 | Lead 8 (bass + lead) | Bass Lead | E01 | 合成 |

### 9.12 合成铺底（89–96）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 89 | Pad 1 (new age) | New Age Pad | E01/E10 | 合成 |
| 90 | Pad 2 (warm) | Warm Pad | E01/E10 | 合成 |
| 91 | Pad 3 (polysynth) | Poly Pad | E01 | 合成 |
| 92 | Pad 4 (choir) | Choir Pad | E16/E01 | 采样+合成 |
| 93 | Pad 5 (bowed) | Bowed Pad | E15/E01 | 采样+合成 |
| 94 | Pad 6 (metallic) | Metallic Pad | E10/E03 | 合成 |
| 95 | Pad 7 (halo) | Halo Pad | E18 | 合成 |
| 96 | Pad 8 (sweep) | Sweep Pad | E10 | 合成 |

### 9.13 合成效果（97–104）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 97 | FX 1 (rain) | Rain FX | E18 | 合成 |
| 98 | FX 2 (soundtrack) | Soundtrack FX | E18 | 合成 |
| 99 | FX 3 (crystal) | Crystal FX | E03/E10 | 合成 |
| 100 | FX 4 (atmosphere) | Atmosphere FX | E18 | 合成 |
| 101 | FX 5 (brightness) | Brightness FX | E10 | 合成 |
| 102 | FX 6 (goblins) | Goblins FX | E12/E18 | 合成 |
| 103 | FX 7 (echoes) | Echoes FX | E18 | 合成 |
| 104 | FX 8 (sci-fi) | Sci-Fi FX | E18/E17 | 合成 |

### 9.14 民族（105–112）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 105 | Sitar | Sitar | E06/E15 | 采样 |
| 106 | Banjo | Banjo | E06 | 采样 |
| 107 | Shamisen | Shamisen | E06/E15 | 采样 |
| 108 | Koto | Koto | E06/E15 | 采样 |
| 109 | Kalimba | Kalimba | E06 | 采样 |
| 110 | Bag pipe | Bagpipes | E15/E16 | 采样 |
| 111 | Fiddle | Erhu / Fiddle | E15 | 采样 |
| 112 | Shanai | Shanai | E15 | 采样 |

### 9.15 打击（113–120）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 113 | Tinkle Bell | Tinkle Bell | E06 | 采样 |
| 114 | Agogo | Agogo | E06 | 采样 |
| 115 | Steel Drums | Steel Drum | E06 | 采样 |
| 116 | Woodblock | Woodblock | E06 | 采样 |
| 117 | Taiko Drum | Taiko | E06/E11 | 采样 |
| 118 | Melodic Tom | Melodic Tom | E11 | 合成 |
| 119 | Synth Drum | Synth Drum | E11 | 合成 |
| 120 | Reverse Cymbal | Reverse Cymbal | E06 | 采样 |

### 9.16 音效（121–128）

| # | GM Program | 本项目乐器 | 引擎 | 兜底 |
|---|---|---|---|---|
| 121 | Guitar Fret Noise | Fret Noise | E06 | 采样 |
| 122 | Breath Noise | Breath Noise | E06 | 采样 |
| 123 | Seashore | Seashore | E18 | 采样 |
| 124 | Bird Tweet | Bird Tweet | E18 | 采样 |
| 125 | Telephone Ring | Telephone Ring | E06 | 采样 |
| 126 | Helicopter | Helicopter | E18 | 采样 |
| 127 | Applause | Applause | E06 | 采样 |
| 128 | Gunshot | Gunshot | E06 | 采样 |

### 9.17 GM 鼓组映射

| GM Drum Kit | 本项目鼓组 | 引擎 | 优先级 |
|---|---|---|---|
| Standard | Acoustic Studio | E07 | P0 |
| Room | Room Kit | E07 | P1 |
| Power | Power Kit | E07 | P1 |
| Electronic | Electronic Kit | E07/E11 | P0 |
| TR-808 | 808 Kit | E07/E11 | P0 |
| Jazz | Jazz Brush | E07 | P1 |
| Brush | Brush Kit | E07 | P1 |
| Orchestra | Orchestral Perc | E07/E06 | P1 |
| SFX | SFX Kit | E07 | P2 |

> 鼓组映射覆盖 GM 通道 10（打击）的 35 个标准键位；未映射键位回退到最近语义音色，保证"每个键都有声音"。

---

## 10. 附录 C：引擎家族覆盖矩阵

> 目的：确认 18 个引擎家族**全部有落地乐器**，避免"建了引擎没人用"或"乐器没有引擎"。

| 引擎 | 家族 | 代表乐器（本目录） | 乐器数（估） | 里程碑 |
|---|---|---|---:|---|
| E01 | 虚拟模拟减法 | Triple Osc、Sawer、PoiZone、Wasp、SimSynth、MiniSynth、Transistor Bass、Analog Mono、Reese Bass、Sub Bass、Supersaw Lead | 11+ | M1/M2 |
| E02 | FM/PM | Sytrus、DX10、Toxic、GMS、FM Electric Piano、FM Bass、FM Bell | 7+ | M1/M3 |
| E03 | 加法/重合成 | Harmor、Harmless、Morphine、Ogun、Autogun、BeepMap、Additive Organ | 7+ | M3 |
| E04 | 物理建模 | Sakura、Plucked、Slayer、BassDrum、Fruit Kick、Drumaxx、Drumpad、Guitar Harmonics | 8+ | M2 |
| E05 | 采样器（单层） | Sampler、Audio Clip、BooBass | 3 | M1 |
| E06 | 多采样器 | DirectWave、FL Keys、SoundFont Player、Concert Grand、管弦/世界采样乐器、GM 主体 | 40+ | M1/M2 |
| E07 | 鼓采样器 | FPC、Groove Machine、60+ 鼓组、GM 鼓组 | 60+ kits | M1/M2 |
| E08 | 切片器 | Slicex、Slicer、Slicer 2 | 3 | M2 |
| E09 | 颗粒 | Granulizer、Wave Traveller、Granular Choir | 3 | M2/M4 |
| E10 | 波表 | Wavetable、Spectral Pad、Metallic Pad、Sweep Pad | 4+ | M3 |
| E11 | 鼓合成 | DrumSynth Live、Melodic Tom、Synth Drum、808/909 合成层 | 4+ | M2 |
| E12 | 形式/语音 | Speech Synth、Vocal Formant Synth、Voice Lead、Goblins FX | 4 | M3/M4 |
| E13 | 模块化 | Patcher、Modular Grid | 2 | M3 |
| E14 | ROMpler/预设机 | FLEX、GMS、Kepler、Kepler Exo、Mobile Rack | 5 | M2/M3 |
| E15 | 弦/弓/吹管专用 | 独奏弦乐、铜管、木管、二胡、尺八、Duduk | 30+ | M2 |
| E16 | 合唱/管弦合奏 | String Ensemble、Brass Ensemble、Choir、Gamelan | 8+ | M2 |
| E17 | 芯片/复古 | Chip Pulse、Chip Triangle、Chip Noise、FM Chip | 4 | M3 |
| E18 | 纹理/电影 | Drone、Riser、Impact、Texture、Rain/Soundtrack FX | 8+ | M4 |

**结论**

- 18 个引擎家族全部有乐器落地，无空引擎。
- 乐器数量分布以 E06/E07/E15 为主（采样类占比高），符合"种类多、开箱即用"的目标。
- 深度合成器（E01/E02/E03）集中在 M1–M3，保证旗舰品质而非只有数量。