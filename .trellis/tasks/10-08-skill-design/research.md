# 一手来源调研：Agent 技能的组织机制

> 原计划由后台 research agent 完成，但它因**网络不可用**失败（`ECONNREFUSED 127.0.0.1:7890`，代理不通），
> `research.md` 从未落盘。以下全部由我直接读取本机一手来源后记录，每条带绝对路径。
> 时间：2026-10-08。

## 一、最权威的两份规约（都在本机，无需联网）

| 文件 | 作用 |
| :--- | :--- |
| `C:\Users\Administrator\.zcode\skills\writing-for-agents\SKILL.md`（81 行） | 给 agent 写一切文档的通用规约（技能、AGENTS.md、被指针引到的文档） |
| `C:\Users\Administrator\.zcode\skills\writing-for-agents\SKILL-MECHANICS.md`（22 行） | **技能专章**：frontmatter、invocation 选择、router skills |
| `C:\Users\Administrator\.zcode\cli\plugins\cache\zcode-plugins-official\skill-creator\0.1.0\skills\skill-creator\SKILL.md`（159 行） | 官方"怎么写技能"的流程与发现路径 |

前者开篇即说：`"When the document you're writing is a skill, read SKILL-MECHANICS.md for frontmatter, invocation choice, and router skills."`（SKILL.md:8）

## 二、技能机制（可直接落地的结论）

### 2.1 invocation：两种，代价不同（SKILL-MECHANICS.md:7-14）

- **model-invoked**：保留 `description` → agent 能自主触发、**其它技能也能调它**。
  代价：description 是"顶层 context pointer，被迫常驻" = **永久上下文负载**。
  机制：**省略** `disable-model-invocation`，写面向模型的 description。
- **user-invoked**：`disable-model-invocation: true` → 只有人手动输入技能名才能触发，别的技能也够不到。
  代价：**零上下文负载**，但花"认知负载"（你得自己记住它存在）。
  机制：description 退回面向人的一句话摘要，触发词列表剥离。
- **选择判据（原文）**："Pick model-invocation only when the agent must reach the skill on its own, or another skill must."

### 2.2 如何按 invocation 拆分（SKILL-MECHANICS.md:16-18）

> "split off a model-invoked skill when you have a distinct leading word that should trigger it on its own (a trigger word you actually use in your prompts), or another skill must reach it. You pay context load for the new always-loaded description, so that independent reach has to be worth it."

即：拆出**独立技能**要付一份常驻 description；除非有"你真的会在 prompt 里用的独立触发词"，或"别的技能必须调到它"，否则不值得。

### 2.3 router skill 是什么（SKILL-MECHANICS.md:20-22）

> "When user-invoked skills multiply past what you can remember, that piled-up cognitive load is cured by a **router skill**: one user-invoked skill that names the others and when to reach for each... It can only hint, never fire them: user-invoked skills have no description, so nothing but the human can reach them."

**要点**：router 是化解**认知负载**的（技能太多、人记不住），它**不能替人触发** user-invoked 技能，只能提示。

### 2.4 渐进披露与目录（skill-creator/SKILL.md:70-89）

层 1 metadata（name+description）**恒在上下文**；层 2 `SKILL.md` 正文触发时才载入，**目标 < 500 行**；
层 3 附属文件（`references/`、`scripts/`、`assets/`）按需读，**理论上不限大小**。
原文的副作用示例（77-89 行）：正文变长就把领域细节拆进 reference 文件，并让 SKILL.md 说明**何时**读它们。

### 2.5 frontmatter 与落点（skill-creator/SKILL.md:41-68）

- 发现顺序（优先级从高到低）：`<project>/.zcode/skills/<name>/SKILL.md`、
  `<project>/.agents/skills/<name>/SKILL.md`、`~/.zcode/skills/<name>/`、`~/.agents/skills/<name>/`。
- **原文**："**Default to creating new skills under `.agents/skills/`** — it's the standard, cross-tool location for new skills."
- `.zcode/skills` 优先级更高，是**覆盖**同名技能的地方。
- 必填 frontmatter：`name`（小写 kebab-case，1–64 字符，**必须与目录名一致**）、`description`。
- description 是**首要触发信号**，要写"做什么 + 什么场景"，且"a little bit pushy"（原文 66 行给了改写示例）。
- 正文风格（91-95 行）：用祈使句；**讲清 why**（现代模型理解了缘由会更守规矩）；
  "If you find yourself writing all-caps MUSTs and NEVERs, that's usually a sign the rule needs better explanation rather than louder enforcement."；示例胜过规则。

## 三、本机真实多技能实例（形态证据）

| 实例 | 结构 | 出处 |
| :--- | :--- | :--- |
| `impeccable`（用户点名的那份） | **一个** 83 行 `SKILL.md` + 24 份 `reference/*.md`（24–145 行/份）+ `agents/*.toml` + `scripts/` | `C:\Users\Administrator\.zcode\skills\impeccable\` |
| `docx` | 315 行 `SKILL.md` + `references/*.md`（10 份）+ `scripts/` + `scenes/` + `env_setup/` | `...\documents\0.1.7\skills\docx\` |
| `xlsx` | `SKILL.md` + `engines/ quality/ scenes/ templates/` | `...\spreadsheets\0.1.7\skills\xlsx\` |
| `zcode-guide` | **一个 plugin 下挂 6 个平级技能**，各自 59–99 行、各有自己的 description | `...\zcode-guide\0.3.0\skills\` |

两点须注意：

1. **子目录名两种写法都存在**：官方 `skill-creator` 与本机 `docx` 用复数 **`references/`**；
   `impeccable` 用单数 `reference/`。**结论：跟官方规约用 `references/`。**
2. `impeccable` 的 `SKILL.md:40-74` 就是"主 + 子"的标准接线：
   `## Commands` 四列表（`Command | Category | Description | Reference`），最后一列是
   `[reference/x.md](reference/x.md)` 形式的相对链接；另有 `## Routing` 段规定
   "无参 → 读 `routing.md` 给上下文感知菜单；明确意图 → 直接载入对应 reference"。
   而 "**Never auto-run a command**"（impeccable/SKILL.md:70）也是它写在路由里的纪律。

## 四、`writing-for-agents` 里几条直接改变设计写法的规约

| 概念 | 原文要点（SKILL.md） | 对本项目的含义 |
| :--- | :--- | :--- |
| **context pointer** | description 就是指针；"One trigger per branch. Synonyms that rename a single branch are one branch written twice"（29-35） | 主技能 description 应按**意图分支**列触发词，同义改写只留一个 |
| **两个负载** | context load（常驻上下文）vs cognitive load（人得记着有哪些文档）（39-47） | 子技能形态的选择就是对这两种负载的取舍 |
| **information hierarchy** | in-file step → in-file reference → disclosed reference（51-62） | 全局纪律留在正文；单意图编排推下去 |
| **progressive disclosure** | "**Branching is the cleanest disclosure test: inline what every branch needs, and push behind a pointer what only some branches reach.**"（64-67） | 这条是子技能拆分的**官方判据** |
| **cache** | "The **environment** is a source of truth too (`package.json` scripts, config files, the directory layout, **`--help` output**), and a document that restates it is a **cache**: a copy of a lookup, earning its load only when the lookup is expensive."（末节） | `describe` 就是 environment。**这条是 D6「不得手抄契约」的官方理由**：手抄契约 = 一份没有收益的 cache |
| **negation** | "**Negation** is the failure mode beside this lever: steering by prohibition drags the forbidden behaviour into context and makes it _more_ available... **Prompt the positive**"（Leading words 节） | 规约应尽量写成正向目标，少用"禁止/不得" |
| **duplication / sprawl / sediment** | 单一真源；"Sprawl"＝文件过长即便每行都有用也是失败；不修剪的默认结局是"sediment"（层层沉积的陈旧内容） | 支持"删掉 `--reason`"这类编辑清理，也支持加反漂移门禁 |

## 五、对 workreport 的可用结论

1. **主技能必须是 model-invoked**（保留 description）—— 它面向 Agent 自主使用（本项目全套设计的前提就是"供 Agent 驱动"）。所以主技能付一份常驻 description，这钱花得值。
2. **子技能不应各自注册成独立技能**。据 SKILL-MECHANICS.md:16-18 的判据：我们没有"独立于 workreport 之外的触发词"——
   用户不会把"记录"当成一个脱离 `workreport` 的技能来喊，他说的是"帮我记一下…"。
   7 份独立技能 = 7 份常驻 description，换不来独立可达性。**正确做法是把它们作为 disclosed reference**
   （`references/*.md`），由主技能正文的**路由表**指向，按分支载入 —— 这正是 `impeccable` 的形态，
   也正合 progressive disclosure 的判据（"push behind a pointer what only some branches reach"）。
3. **router skill 那套不适用**：router 只用于"user-invoked 技能太多、人记不住"的场景
   （SKILL-MECHANICS.md:20-22），而我们是单个 model-invoked 技能。我们那个 `routing.md` 不是 router skill，
   而是"意图不明分支"用的普通 disclosed reference。**命名上要避免与 router skill 混淆。**
4. **子目录用 `references/`**，不用 `reference/`（跟官方规约，不跟 impeccable 的拼写）。
5. **主技能正文目标 < 500 行**（官方上限口径），我们自己再收紧到 ~110 行。
6. **写法纪律**：讲 why、祈使句、示例胜于规则、少用大写 MUST/NEVER、**多用正向表述**（negation 是反模式）。
7. **`describe` 是 environment 而非文档**：技能引用它、绝不复制它 —— 现在这条有了官方依据，
   可以写进技能自己的说明里（也值得进 PRD 的 D6 落点）。
