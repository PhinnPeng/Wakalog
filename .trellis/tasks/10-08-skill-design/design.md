# Agent 技能体系设计方案：主技能 + 子技能

> 设计对象：为 `workreport` CLI（0.1.0，14 条命令）编写一套 Agent 技能。
> **一手依据**见同目录 [`research.md`](research.md)（本机路径逐条带出处）；主要来自官方
> `skill-creator` 与 `writing-for-agents`（含其技能专章 `SKILL-MECHANICS.md`），
> 以及本机 `impeccable`／`docx`／`zcode-guide` 三个真实多技能实例。
> 需求与验收见 [`prd.md`](prd.md)。

## 一、技能要解决的问题（也就是它的边界）

CLI 已经把**能自描述的都自描述了**。实测 `workreport describe` 覆盖：

| 已自描述（技能里没有位置写它） | 实测 |
| :--- | :--- |
| 14 条命令的 name／description／readonly／flags | ✅ 每个 flag 带 `name`／`type`／`required`／`values` |
| 5 个全局 flag | `--format --pretty --quiet --db --no-color` |
| 19 个错误码 + 各自退出码 | ✅ |
| 4 个退出码的语义 | ✅ |
| 3 条跨字段约束（when／forbid／error_code） | ✅ |
| limits（content ≤ 2000、默认 limit 200、短 ID 12 字符） | ✅ |

**所以技能的职责只剩一类：CLI 描述不了的"编排知识"** —— 什么时候调哪个命令、哪一步不能省、
失败了算谁的账。这正是 PRD 5.5 的原话与 D6 的硬约束。

这条还有一层更精确的表述，来自 `writing-for-agents`（research.md §四「cache」行）：

> "The **environment** is a source of truth too (`package.json` scripts, config files, the directory layout,
> **`--help` output**), and a document that restates it is a **cache**: a copy of a lookup, earning its load
> only when the lookup is expensive."

`describe` 就是本工具的 environment。**手抄契约等于造一份没有收益的 cache** —— 这比
"D6 说不许抄"更能解释清楚为什么不该抄：抄了既要维护、又会过期，而查一次 `describe` 很便宜。

## 二、为什么是"主技能 + 子技能"，以及子技能**不该**各自注册

这是本方案最需要讲清的一点，因为两种做法看起来都像"主 + 子"。

`SKILL-MECHANICS.md`（research.md §2.1–2.3）把机制讲死了：

- **model-invoked** 技能保留 `description` → Agent 能自主触发、**其它技能也能调到它**；
  代价是这份 description **永久占用上下文**。
- **user-invoked** 技能（`disable-model-invocation: true`）→ 零上下文负载，但只有人能手动触发。
- **拆出独立技能的判据**（原文）："split off a model-invoked skill when you have a **distinct leading word
  that should trigger it on its own** (a trigger word you actually use in your prompts), or another skill
  must reach it."

套到 workreport 上：

| 问题 | 答案 |
| :--- | :--- |
| 主技能要不要被 Agent 自主触发？ | **要**（本工具整套设计的前提就是"供 Agent 驱动"）→ 主技能 model-invoked，付一份常驻 description，值。 |
| 7 个子部分有没有"独立于 workreport 之外的触发词"？ | **没有**。用户不会把"记录"当成一个脱离 `workreport` 的技能来喊，他说的是"帮我记一下今天的活"。 |
| 有没有别的技能必须调到某个子部分？ | **没有**（本项目只有这一个技能）。 |
| ⇒ 结论 | **子部分不注册成独立技能**，而是主技能正文用路由表指向的 **disclosed reference**（`references/*.md`），按分支载入。 |

这条同时命中 `writing-for-agents` 对渐进披露给出的判据（research.md §四）：

> "**Branching is the cleanest disclosure test: inline what every branch needs, and push behind a pointer
> what only some branches reach.**"

- **每个分支都要的** → 留在主技能正文（取契约的方式、三条全局纪律、路由表本身）。
- **只有某些分支才碰的** → 推下去（各意图的编排步骤）。

而这个形态有现成范本：`impeccable` 就是**一个** 83 行 `SKILL.md` + 24 份 `reference/*.md`，
主文件用一张四列表把 `Command` 接到 `Reference`（research.md §三）。

**命名澄清**：我们那个 `routing.md` **不是** `SKILL-MECHANICS.md` 说的 "router skill"。
router skill 是"user-invoked 技能太多、人记不住"时的解药，且它**只能提示、不能替人触发**
（research.md §2.3）。我们只有一个 model-invoked 技能，不存在这个问题。
`routing.md` 只是"意图不明"这一个分支用的普通 disclosed reference。

## 三、目录结构

```
workreport/
├── SKILL.md                 # 主技能：路由 + 全局纪律 + 取契约（目标 ≤ 110 行）
└── references/              # 子技能：按"用户此刻的意图"切，按分支载入
    ├── routing.md           # 意图不明时的推荐（普通 reference，非 router skill）
    ├── record.md            # 记录已完成的工作项
    ├── plan.md              # 记计划 / 查计划 / 计划转工作项
    ├── query.md             # 查、搜、列（信封与 truncated、全序分页）
    ├── revise.md            # 改内容 / 类型迁移 / 删除 / 复原
    ├── report.md            # 日报（只引用 PRD 5.2.9，不复述格式）
    └── troubleshoot.md      # 排障：退出码 3、doctor、配置、备份
```

- 子目录用**复数 `references/`** —— 跟官方规约（`skill-creator` 与官方 `docx` 都用复数），
  不跟 `impeccable` 的单数拼写（research.md §三）。
- 主技能与子文件之间只用**相对 Markdown 链接**连接（与 impeccable 同法）。
- 官方对正文的尺寸口径是"目标 < 500 行"（research.md §2.4）；我们自己再收紧到 ~110 行以上述为目标。

## 四、主技能 SKILL.md 的职责边界

| 放 | 不放 |
| :--- | :--- |
| 取契约的时机与方式（`describe --format json`），以及"失败即重取" | 任何参数名、枚举值、错误码、退出码（D6） |
| 三条全局纪律：按 `code` 分支、退出码 1/2/3 的处置、写操作非幂等 | 单个意图的步骤（挪进 `references/`） |
| 意图 → 子文件的路由表（四列，同 impeccable） | 日报的分组／排序规则（D35：真源是 PRD 5.2.9） |
| 一句"示例与 `describe` 冲突时以 `describe` 为准" | 可由 `doctor`／`describe` 得到的细节 |

**description 怎么写**（这是触发可靠性的全部，官方口径见 research.md §2.5）：
按**意图分支**列触发词（"记一条／补录／记计划／查计划／计划转工作项／改／删／恢复／生成日报／搜"），
同义改写只留一个（"Synonyms that rename a single branch are one branch written twice"），
并写明"Not for…"挡掉无关任务。**description 里放触发场景，不把正文内容搬进去。**

## 五、路由机制（三种入口）

| 入口 | 行为 |
| :--- | :--- |
| 意图明确（"记一条""把计划转成工作项""生成日报"） | 直接载入对应 `references/*.md`，按它执行 |
| 意图不明／無参（"用一下 workreport"） | 载入 `references/routing.md`，给 2–3 个带理由的推荐 + 全菜单，**不自动执行** |
| 任何一次调用失败 | 回 `describe` 取最新契约，按 `code` 分支处置（5.5.1） |

`routing.md` 的推荐应当**基于当前状态**而非固定顺序 —— 这是 impeccable 那套里最值得抄的一点：
它先跑一个 signals 脚本拿项目现状再决定推荐什么（research.md §三 提到的 `routing.md`）。
workreport 的对应物是三条**已有的只读命令**：`list --overdue`（有无遗留计划）、
`categories`（分类是否碎片化）、`doctor`（环境是否健康）。
**不为此发明新命令。**

## 六、子技能清单

| 子技能 | 何时载入 | 载入后必须做的事（编排知识，CLI 不会替你做） |
| :--- | :--- | :--- |
| `routing.md` | 意图不明 | 按上节给推荐；不自动执行 |
| `record.md` | "记录／补录／先记下来" | ①相对日期→绝对日期（用户本地日历，非 UTC）②先 `categories` 做分类归一化 ③含引号/换行/接近上限走 stdin ④`add` 冲突报错就改参数 |
| `plan.md` | "明天要做／查计划／把计划转工作项" | ①计划日期必给且由 Agent 解析 ②status 缺省 `planned` ③转换唯一入口是 `convert`，且先 `list`/`show` 定位 id |
| `query.md` | "查／搜／有哪些" | ①读 `.items` 并检查 `.truncated` ②中文按整串子串匹配，不是分词；搜不到换更短子串 ③相对日期区间由 Agent 解析 |
| `revise.md` | "改／删掉那条／恢复" | ①改前先读：一律先 `show`/`list` 拿 id ②`delete` 前向用户复述内容并确认（5.5.1）③`purge` 是唯一不可逆操作，须双确认 |
| `report.md` | "生成日报" | ①确认日期 ②计划区目标日由 Agent 算（CLI 不猜"下一个工作日"）③格式规则**引 PRD 5.2.9，不复述**（D35） |
| `troubleshoot.md` | 退出码 3／"坏了"／环境问题 | ①停止并转告人类，不重试、不改写数据 ②`doctor` 取环境全貌 ③区分"配置错"与"库错"与"运行时错" |

> 上表"必须做的事"一列里我刻意用**正向表述**（"先 `categories`"而不是"不要跳过归一化"）。
> 理由见 research.md §四「negation」：以禁令表述会把被禁的行为一起拉进上下文、反而让它更可用。
> 官方建议是先给正向目标；禁令只留给无法正向表达的死线。

## 七、待拍板的设计决策

- **SD-1 子技能的形态** —— 推荐 **A：一份 model-invoked 主技能 + `references/*.md` 披露文件（子文件不注册）**。
  判据是官方的两条（research.md §2.2、§四）：没有独立触发词、也没有别的技能要调它，故不值得为
  7 个子部分各付一份常驻 description。被拒替代 **B：各自注册为平级技能**（zcode-guide 那种形态）——
  它适合"多个技能共享同一个 plugin、各自有独立触发场景"的情形，而我们所有意图都从
  `workreport` 这一个词进去。**A 升级到 B 的成本很低**：给 `references/x.md` 加 frontmatter、
  挪成 `skills/<name>/SKILL.md` 即可，正文不用重写。
- **SD-2 技能落点** —— 推荐 **`.agents/skills/workreport/`**。依据：官方 `skill-creator` 原文
  "Default to creating new skills under `.agents/skills/`"（research.md §2.5），且 AGENTS.md 也把
  `.agents/skills/` 列为项目级技能目录（该目录与本项目当前都还不存在，需新建）。
  被拒替代：只写"放客户端目录"不给落点 —— 那样每个用户都得自己猜。
- **SD-3 部署示例放哪** —— 推荐放 `README.md` 一小节（PRD 5.5.2 要求提供）。
  被拒替代：塞进 `SKILL.md`，会稀释技能正文、抬高常驻负载。
- **SD-4 契约获取的时机** —— 推荐"**每会话首次 + 任一次调用失败时**"（5.5.1 原文），
  **不做本地缓存**。依据同 §四「cache」：缓存就是又造一份会过期的副本。
- **SD-5 是否同时加反漂移门禁** —— 推荐加（见下节），把 D6 从自觉变成可执行断言。

## 八、怎么证明它有效（验收）

1. **冷启动验收**：一个只拿到 `SKILL.md` + `describe` 输出、没有任何本项目上下文的 Agent，
   能否正确完成 7 个典型任务（记录、补录历史、记计划、查计划、计划转工作项、删除、生成日报）。
   **判据是编排纪律不被违反**（如删除前是否复述、分类是否先归一化），不是"命令跑通"。
   按官方 `skill-creator` 的流程，这一步应当是：写草案 → 用 2–3 个**真实用户口吻**的 prompt 试跑 →
   和用户一起看输出 → 改（research.md §2.5 引 skill-creator:97-137）。
2. **反漂移门禁**（建议一并落地）：在 `test/gates.test.ts` 加一条，断言技能文本里
   **不出现**任何错误码字面量、枚举值清单、以及 `--flag <...>` 形态的参数声明。
   这把 D6 从人的自觉变成可执行断言 —— 与既有门禁 A（错误码单点定义）同一思路，只是把对象从
   `src/**` 扩到技能文本。**这是本方案里我最建议一并做掉的一条**：没有它，技能会随 CLI 版本漂移，
   而且没人会发现（`writing-for-agents` 把这种不修剪的默认结局叫 "sediment"）。
