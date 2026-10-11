# 游戏工作室技能包分析报告（game-studio-analysis）

> 日期：2026-10-11。性质：**只读分析**（本报告是本轮唯一新增文件，未改动任何代码/文档/资产，未跑测试/构建）。
> 工具：Claude-Code-Game-Studios 技能包。本轮实际调用 4 个技能：`project-stage-detect`（阶段判定入口）、`architecture-review`（架构面）、`asset-audit`(资产面)、`content-audit`(章节内容面)。
> 边界声明见 §5；凡引用仓库事实均给 file:line，凡属分析者判断均标注。

---

## 1. 项目阶段判定

### 1.1 技能执行方式说明（如实记录）

- 技能包自带的确定性扫描脚本 `.claude/scripts/artifact-check.sh` **在本仓库不存在**（本仓库无 `.claude/` 目录），按技能「手工扫描兜底」流程执行；技能约定的 `design/`、`production/`、`prototypes/`、`docs/architecture/` 目录在本仓库均不存在——本仓库用 `docs/*.md` + 根目录 `story.md`/`levels.md` 自有布局承载同类信息。此布局差异本身是一项发现（见 §4-P4）。
- 技能自身的 workflow tier 解析为 `minimal`（无 project.yaml 可覆盖）。`minimal` 档下「缺 GDD/美术圣经/ADR/冲刺计划不算 gap」——因此 §4 的差距全部以「仓库自身文档承诺了但未做」为依据，不以「技能包模板有而仓库没有」为依据。

### 1.2 阶段结论

**四档判定（任务口径）：`existing-work`（现成项目），无争议。**
依据：45 个 TS 源文件（apps/game）+ 10 个 Go 文件（apps/backend）+ 2 个共享包 + 完整 28 篇 docs/ + 根目录 story.md(369 行)/levels.md(594 行)，git 近 17 天 69 commits（2026-09-25 起，最近 commit 2026-10-09）。

**技能细档判定：Pre-Production 后期（设计已量产、目标引擎工程未初始化），Stage Confidence: CONCERNS。**
按技能的七档启发式（Concept → Systems Design → Technical Setup → Pre-Production → Production → Polish → Release）逐条比对：

| 技能判据 | 本仓库实测 | 指向 |
| --- | --- | --- |
| 游戏概念文档 | 有，且极深（story.md 369 行十章三幕、docs/story-bible.md、docs/narrative-bible.md） | ≥ Systems Design |
| 系统分解 | 无 `systems-index.md`，但 `docs/gameplay-v3.md` §「玩法模块总表(M1-M7)」+ `levels.md` §1.2 模块引入矩阵实质承担该职能 | ≥ Systems Design |
| 引擎已定 | `docs/engine-selection.md:19-26`（2026-10-04 Steam-only 重估，首选 Godot 4.x）+ `README.md:9` | ≥ Technical Setup |
| 引擎已配置 | **无**：全仓无 `project.godot`，`apps/game` 是 Cocos 语境脚手架（creator-templates/、assets/scripts/cocos-*），engine-selection.md:26 明言「Cocos 脚手架不再推进」 | 卡在 Technical Setup |
| 代码根 ≥10 源文件 → Production | 字面满足（apps/game 45 个 TS 文件），但这些文件是**被推翻的引擎（Cocos）×被推翻的玩法代际（v2 投篮）**的原型，engine-selection.md:26 明确不再推进 | 按字面会误判 Production |

**CONCERNS 的由来（技能判定规则：信号歧义即 CONCERNS）**：文件数按字面够 Production，但代码属已废弃引擎与上一代玩法；活跃开发全部发生在文档侧。综合定性：**existing-work · Pre-Production 后期 · 引擎切换过渡态**——设计侧（叙事/关卡/玩法/合规/范围）已达到可进 Production 的密度，工程侧在 2026-10-04 双重转向（Steam-only + Godot）后尚未落第一块新地基。

---

## 2. 现状盘点

### 2.1 monorepo 结构（apps/packages/tools 各是什么）

| 目录 | 实际内容 | 与 2026-10-04 转向后方向的关系 |
| --- | --- | --- |
| `apps/game` | 45 个 TS 文件：引擎无关骨架（bootstrap/services/platform/config）+ Cocos 组件壳（`creator-templates/` 5 个、`assets/scripts/cocos-*`）+ **v2 代际玩法代码**（`assets/scripts/gameplay/shot-input.ts` 投篮操作等）+ 场景流 demo（`src/cocos-flow-demo.ts`）；9 个测试文件中的大部分在此 | **停推进**（engine-selection.md:26），代码保留未删 |
| `apps/backend` | 10 个 Go 文件（cmd/server、internal/…），Gin+Gorm+SQLite 薄网关（`docs/architecture.md:29`）；根 package.json:8-10 仍有 `build:backend`/`dev:backend`/`check:backend` 脚本 | **停建归档**（`docs/gameplay-v3.md:151`「零自建服务器定稿(2026-10-04 用户拍板)」），代码保留未删 |
| `packages/game-config` | 909 行 TS：**v1/v2 代际配置**（`configs/gameplay.v1.json`、`gameplay.v2.rhythm.json`、`gameplay.v2.basket.json`、`items.v1.json`、`meta.v2.json`）；无任何 v3 配置 | 引擎无关、保留；但 **v3 配置未启动**（见 §3.3） |
| `packages/platform-sdk` | `PlatformName = "wechat" \| "douyin" \| "web" \| "unknown"`（`src/index.ts:1`）+ MemoryPlatformSdk 纯接口实现 | 引擎无关、保留；但 **当前主平台 Steam 不在枚举里**（见 §4-P2） |
| `tools/art` | 10+ 个 AI 出图 prompt（backgrounds/character-*/ui-icons…）+ `exports/manifest.sample.json` 改名管线 + `raw/`/`exports/` 空目录 | 与引擎无关，**现行有效**（README.md:11「AI 出图先行」） |
| 根文档 | `story.md`(369 行，叙事完整稿) `levels.md`(594 行，逐关设计，39 个关卡条目) | 设计正典在 `docs/story.md`/`docs/levels.md`，根目录为扩写稿（story.md:3 头注声明基准关系） |

### 2.2 章节主线（story.md / levels.md）落地程度

**文档侧：基本齐装。**
- 十章主线 + 序章 + 隐藏章完整：`levels.md` §2 逐关设计覆盖 P0-1~P0-3、1-1~8-3、9-0（39 个 `####` 关卡条目），含 BOSS、变体、教学位（levels.md:73-279 章节标题序列）。
- 三幕结构 + 人心灯配额 + 羽色三段：`story.md:9-27`（幕表 + 「门槛在第一幕,抉择在终章」）。
- V1=Steam Demo 的范围已收敛成独立索引：`docs/v1-scope.md`（11 关、47 条台词、DEFER 表、缺口清单、验收定义）+ `docs/levels-demo-spec.md`(356 行，11 关全量配表与台词)。
- 玩法模块 M1-M7 的机制承接表与验收标准：`docs/gameplay-v3.md`（「十章机制承接」+ §「V1 验收标准(Steam Demo)」159 行起）。
- 合规闸门有专档：`docs/compliance.md`(86 行)、`docs/meme-collection.md`(180 行)。

**实现侧：≈0（相对 v3 方向）。**
- 无目标引擎（Godot）工程；无任何场景/关卡数据文件。
- 配置包里只有 v1/v2 代际数据；`docs/gameplay-v3.md:155` 点名的 `gameplay.v3.rhythm.json`/`gameplay.v3.fight.json`/`gameplay.v3.awaken.json` **均不存在**。
- 47 条 V1 台词只存在于 demo-spec 文档，代码/配置中无台词数据（grep 无命中）。
- 现存可运行物仅为 TS 层 demo（`apps/game/src/cocos-flow-demo.ts` 场景流、services 层测试），验证的是骨架而非章节玩法。

---

## 3. 技能逐个：查了什么 → 结论

### 3.1 `project-stage-detect`（阶段判定入口）

- **查了什么**：全仓目录/文档/代码盘点（概念文档、系统分解、引擎配置、代码根文件数、生产物、原型、测试）；按七档启发式从最先进档回扫。
- **结论**：existing-work；细档 Pre-Production 后期，Confidence **CONCERNS**（文件数字面达 Production 但代码属已弃引擎；引擎已定未配）。判定依据见 §1.2 表，全部有 file:line。
- **证据**：`docs/engine-selection.md:19-26`、`README.md:9-10`、apps/game 45 文件清单、无 `project.godot`（find 零命中）。

### 3.2 `architecture-review`（架构面；按其 `minimal` 档裁剪执行——该档明言不强制架构文档，故只做决策盘点与一致性，不做全量追溯矩阵）

- **查了什么**：架构决策文档清单（本仓库对应物：`docs/architecture.md`、`docs/engine-selection.md`、`docs/gameplay-v3.md` §平台与商业化/工程要点、`docs/v1-scope.md`）、决策与现状的一致性、设计需求→决策的覆盖。
- **结论**：
  1. **决策记录质量高**：engine-selection.md 是一份教科书级决策记录——结论、推翻条件（§0:12-17）、触发记录（§0.1:19「触发推翻条件 #2,用户拍板」）、迁移影响面（§4）、复核记录（§6/§7）俱全。
  2. **但决策变更未传播**：`docs/architecture.md` 整篇仍是旧决策（微信/抖音目标 ：5-6、Cocos :14、Go 网关+shield 实时层 :25-82），无任何「已废止」横幅；engine-selection.md:122-129（§4.2 处置表）自己列明了「切换时执行」的归档/改写清单，**至今未执行**（本轮为只读，未代执行）。
  3. **README 自相矛盾**：`README.md:9-10` 已写 Godot + backend 归档，同文件 `README.md:19`（目录树注释「Cocos Creator game project」）与 `README.md:43`（Next steps「Initialize a Cocos Creator project」）仍是旧口径——同文档内部新旧并存。
  4. **幽灵目录**：`docs/architecture.md:108` 推荐目录里的 `packages/protocol` 从未创建（packages/ 实际只有 game-config、platform-sdk）。
- **证据**：上行所引行号均为实读核验。

### 3.3 `asset-audit`（资产面）

- **查了什么**：按技能协议先查「引擎资产根 + 规范来源」。引擎资产根：Godot 工程不存在 → 按技能规则记 `engine unresolved`（未就此停步，因为仓库另有 Cocos 语境 `apps/game/assets/` 可作为事实资产根扫描）；规范来源：无 `design/art/art-bible.md`，采用仓库自有规范（`docs/asset-naming-conventions.md`、`docs/ai-asset-import-rules.md`、`docs/placeholder-assets-checklist.md`）。
- **结论（Verdict: NOT ASSESSED——按技能协议，零资产即此判定，不是 COMPLIANT）**：
  - **全仓 0 个二进制美术/音频文件**（png/jpg/webp/mp3/ogg/wav find 计数 = 0）；`apps/game/assets/` 下 textures/audio/prefabs/bundle 等目录全部只有 `.gitkeep`；`tools/art/raw/`、`exports/` 为空（仅 manifest.sample.json）。
  - Naming/Size/Format/Orphaned 四项检查 = `NOT ASSESSED — no assets`（无对象可查，不等于合规）。
  - 有价值的侧面结论：**出图管线就绪但生产未启动**——prompt 模板（tools/art/prompts/ 10+ 文件）、改名 manifest 管线（manifest.sample.json 的 raw→exports 映射）、占位资产清单（placeholder-assets-checklist.md 按 Boot/Home/Gameplay/Result 场景列明）三者齐备；`docs/v1-scope.md:68` 亦自认「占位资产清单已有,生产未启动」。命名规范（小写下划线+类型前缀）与技能包默认 Godot 规范同构，切换引擎无损。

### 3.4 `content-audit`（章节内容面；其规范输入 `design/gdd/systems-index.md` 不存在，按技能协议本应整体 NOT ASSESSED——以下为借用其「指定 vs 已实现」方法的适配版盘点，判定方法属分析者）

- **查了什么**：设计文档声明的数量口径 vs 代码/配置中可找到的实现物。
- **适配版差距表**：

| 内容类型 | 设计侧指定 | 实现侧找到 | 状态 |
| --- | --- | --- | --- |
| 关卡/场景 | 39 关（levels.md §2）+ V1 子集 11 关（v1-scope §1.1） | 0（无引擎场景文件） | NOT STARTED |
| 玩法模块 M1-M7 | 7 个（gameplay-v3 模块总表） | 0 个属 v3；现存 gameplay 代码是 v2 投篮代际 | NOT STARTED（v2 原型除外） |
| v3 配置文件 | ≥3 个（gameplay-v3.md:155 点名） | 0（configs/ 只有 v1/v2 五个 json） | NOT STARTED |
| 台词数据 | 47 条（v1-scope §1.2，demo-spec 全量） | 0（仅存在于文档） | NOT STARTED |
| 叙事/关卡设计文档 | 十章三幕+逐关 | story.md 369 行 + levels.md 594 行 + demo-spec 356 行 + v1-scope 80 行 | **COMPLETE（文档级）** |

- **结论**：内容设计与内容实现完全脱节——设计侧密度已达量产标准，实现侧因引擎转向归零重启动。这不是设计缺口，是**工程重启缺口**（与 §1 阶段判定互证）。

---

## 4. 差距与建议（现状 vs 技能体系对应阶段做法，按优先级）

> P0 最高。每条给依据；「技能体系做法」指该技能包对对应阶段的通行要求（且受 §1.1 所述 minimal tier 约束，不拿模板目录空喊）。

- **P0 · 初始化 Godot 4.x 工程**（差距：引擎 2026-10-04 已定但零工程文件）。
  技能体系里 Pre-Production → Production 的门槛就是「引擎已配置 + 代码根产出」。现状卡在门前提已半年量级的文档（非literal时间，指文档轮次已达第十五~二十八轮，见 docs/story.md、gameplay-v3.md 各轮记录）。
  依据：`docs/engine-selection.md:26`（「引擎工程初始化改按 Godot 4.x 重排」）、v1-scope.md:65（自认缺口 #1）、全仓无 project.godot。
  建议：新建 `apps/game-godot`（或替换 apps/game 内容前先归档 Cocos 脚手架），Boot/Home/Gameplay/Result 四场景壳已有 Cocos 版语义可平移（creator-templates/ 五个组件壳 + docs/gameplay-node-binding.md 的钩子语义是引擎无关的，engine-selection.md:128 已点名「改写为引擎无关版」这条路）。

- **P0 · 执行决策变更传播（engine-selection §4.2 处置表）**（差距：4+ 篇文档仍按已推翻决策表述，含 README 自相矛盾）。
  依据：`docs/engine-selection.md:122-129` 处置表全部待办；`README.md:19`/`:43` vs `README.md:9-10`；`docs/v1-scope.md:9`（「引擎定 Cocos Creator 3.8.x」）与 `:65`（按 cocos-project-plan 初始化）整段过时；`docs/gameplay-v3.md:156`（工程要点仍写「Cocos 工程:沿用 cocos-project-plan.md」——注意其 :151 平台小节已按 2026-10-04 修订，同文档新旧并存）；`docs/architecture.md` 整篇无废止横幅。
  建议：按处置表一次性清账（归档 cocos-* 系列、architecture.md 加废止横幅或重写、v1-scope/engine 许口径刷新、README 目录树与 Next steps 对齐）。本轮只读未代做。

- **P1 · 补齐 Steam 侧平台抽象**（差距：platform-sdk 只覆盖微信/抖音/web，当前主平台 Steam 缺位）。
  依据：`packages/platform-sdk/src/index.ts:1` 枚举无 `steam`；`docs/gameplay-v3.md:151` 定稿「Steamworks 管玩家侧」（榜单/成就/云存档全走 Steamworks）+ 文末自注「微信版后行时另起平台侧(走 platform-sdk)」——即 platform-sdk 被定位为后行平台用，Steam 侧需要新的桥接设计（Godot 下对应 GodotSteam，engine-selection.md:23 已核实其全 API 覆盖）。
  建议：在 Godot 工程内先接 GodotSteam 满足 V1（Demo 不需要榜单全量），platform-sdk 保持冻结为后行平台契约，勿现在强行扩枚举。

- **P1 · v3 配置代际启动**（差距：game-config 停在 v1/v2，v3 点名文件为零）。
  依据：`packages/game-config/configs/` 实测仅 v1/v2 五个 json；`docs/gameplay-v3.md:155` 点名三个 v3 文件；`docs/v1-scope.md:67` 自认「demo-spec → game-config 字段映射未做」。
  建议：V1 只需 demo-spec 三章的配表映射（节奏判定三档/格斗帧数据/唤醒链），不必一次性铺十章——与 DEFER 表（v1-scope §2）对齐。

- **P2 · backend 与 Cocos 代码的物理处置**（差距：两块停推进代码仍占工作树，脚本入口仍活）。
  依据：README.md:10「停建归档」但 `apps/backend/` 10 个 Go 文件在树、根 package.json:8-14 仍暴露 build:backend/dev:backend 脚本；engine-selection.md:26 Cocos 脚手架「不再推进」但无归档动作。文档说「归档」，仓库里是「原地保留」——两者不同。
  建议：明确到 `docs/archive/` 或单独分支/目录，或至少从根脚本入口摘除，避免后续会话误当活代码。

- **P2 · 合规终审前置**（差距：A3/A6 指向性组合检查未跑，是文案入库闸门）。
  依据：`docs/v1-scope.md:66` 自认缺口 #2、§1.4 列为「V1 必过」；`docs/compliance.md`(86 行) 已备方法论。
  建议：作为 47 条台词进 game-config 的前置步骤执行（正好卡在 P1 配表工作之前）。

- **P3 · 美术圣经（art bible）缺位但暂不阻塞**（差距：有命名规范+出图 prompt，无风格基准文档）。
  依据：`docs/asset-naming-conventions.md`（命名）、`docs/art-tools.md`（工具链）、`tools/art/prompts/character-base.md`（角色基准 prompt）已覆盖「怎么出图」，但无色彩/构图/母题基准文档。story.md:41 点名的美术母题（「染蛊者瞳孔里映着别人的瞳孔,无限倒影」）只存在于叙事侧。
  建议：首批 AI 出图跑通后再沉淀 art bible（把成功的 prompt+选图标准固化），顺序上不必提前——与 README.md:11「AI 出图先行、不手收素材」的策略一致。

- **P4 · 技能包脚手架的选择性采纳（低优先、防过度工程）**。
  现状：本仓库未采用技能包约定目录（design/gdd、docs/architecture/adr-*, production/），以 docs/ 平面布局 + 文内轮次记录替代。**多数替代物质量不低**（gameplay-v3 M1-M7 表≈systems-index；engine-selection §0.1≈ADR；v1-scope≈milestone scope；各文档「口径核对记录」≈review trail）。
  建议：不整包迁移。若引入，优先两项：①`docs/architecture/` 存放 ADR（把 engine-selection §0.1 这类决策抽出成独立可引用单元）；②用 `content-audit` 可机读的 systems-index 替代人读轮次记录做进度索引。其余（sprint/gate 类）对单人+多会话调度的工作流收益存疑，暂缓。

---

## 5. 边界声明：技能工具产出 vs 分析者判断

**技能工具产出（按其自身协议执行后的结论）：**
- 阶段判定的启发式与七档框架、CONCERNS 置信规则（project-stage-detect）；
- 「零资产→NOT ASSESSED 而非 COMPLIANT」「引擎资产根未解析」「缺规范→指名补规范的技能」等判定协议（asset-audit 明文反伪造条款）;
- 「输入缺失→NOT ASSESSED — NO DATA」的诚实失败态（content-audit / architecture-review 同款条款）;
- minimal tier 下「缺 GDD/ADR/冲刺计划不算 gap」的范围约束（三个技能共享的 tier 解析）。

**分析者的适配与判断（技能协议之外的自由裁量，均已在上文就地标注）：**
- 技能规范路径（design/gdd 等）→ 本仓库实际布局（docs/*）的映射关系；
- architecture-review 在 minimal 档下裁剪为「决策盘点+一致性」而非全量追溯矩阵；content-audit 在 NO DATA 判定之外的「适配版差距表」（§3.4，方法借用、表内容自算）;
- 阶段综合定性「existing-work · Pre-Production 后期 · 引擎切换过渡态」的措辞；
- §4 全部差距的优先级排序与具体建议（含「不建议整包迁移脚手架」的取舍）；
- agents 视角借用声明：本轮**未 spawn 任何 subagent**（遵守任务约束），technical-director / producer / game-designer 视角由主会话在 §3.2 / §4 排序 / §3.4 中以分析形式体现。

**数据可信度备注**：所有文件计数与行号引用来自本次实跑（find/grep/wc/read，2026-10-11）；未运行任何构建/测试（任务约束）；技能包的 receipt/freshness 机制因脚本不存在未启用——若近期有并行会话提交，本报告结论以 HEAD `a1739d5` 时点为准。
