# Agent Skills 调研：mattpocock/skills + ponytail（Cursor 向）

- 性质：只读调研。不改业务代码，不往任何人机器安装 skill。
- 日期：2026-09-03
- 跳过：`grill-me` / `grilling` / `grill-with-docs`（用户已自有转写）
- 读者画像：Cursor 重度、全栈、少造轮子

## 核对锚点（必须能回溯）

| 仓库 | 核对 ref | 时间 | License |
| --- | --- | --- | --- |
| [mattpocock/skills](https://github.com/mattpocock/skills) | `6654f6b60cd9d5be8b54c6fafe44346dabeb3b76` | 2026-08-24 | MIT |
| [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | `2ed6c52c9d7e5e56942508591085fd45dea277d3`（npm `4.9.0`） | 2026-08-07 | MIT |

本文只根据上述原文归纳，不编造未出现的命令、hooks 或文件。

mattpocock 仓库把 skill 分成两类（README「Reference」）：

- **User-invoked**：frontmatter 有 `disable-model-invocation: true`，只有人打 `/name` 才会进上下文。
- **Model-invoked**：默认可被模型按 `description` 自动捞起，人也可以 `/name`。

Cursor 官方文档（[Skills](https://cursor.com/docs/skills)、[Rules](https://cursor.com/docs/rules)、[Cloud Agent best practices](https://cursor.com/docs/cloud-agent/best-practices)）与此对齐：`disable-model-invocation: true` 时只有显式 `/skill-name` 才会带上该 skill。

---

## 0. Cursor 最短安装约定（后面每条都引用这里）

### Skill 扫描目录（官方）

| 位置 | 作用域 | Cloud Agent / 远程 SSH / 自托管 worker |
| --- | --- | --- |
| `.cursor/skills/` 或 `.agents/skills/` | 本仓库 | 会带上（在仓里） |
| `~/.cursor/skills/` 或 `~/.agents/skills/` | 本机全局 | **不会复制过去** |
| `.claude/skills/`、`.codex/skills/` 及对应 `~/` | 兼容扫描 | 同左：用户级不进 Cloud |

官方还要求：`name` 必须等于「装 `SKILL.md` 的那一层文件夹名」（小写、数字、连字符）。因此不要把 `skills/engineering/tdd/` 整树原样丢进 `.cursor/skills/engineering/tdd/`——应拍平为 `tdd/SKILL.md`。

`npx skills@latest add mattpocock/skills` 历史上曾把 Cursor 全局技能写到 `~/.agents/skills` 而漏掉 `~/.cursor/skills`（vercel-labs/skills#421）。**最短可靠路径是手工拷目录**，不要依赖 installer 的默认全局目录。

### Rule 与 User Rules

- 项目常驻：`.cursor/rules/*.mdc`（必须是 `.mdc`；纯 `.md` 会被忽略）。
- 本机常驻：Cursor → Customize → Rules（User Rules）。Cloud Agent **会**吃 User Rules。
- Team Rules：Dashboard，Team/Enterprise。
- `alwaysApply: true` 的 rule **不会**被 `/migrate-to-skills` 转成 skill。

### 本调研的默认建议

- 个人纪律、跨仓复用、且 **不写仓库文件** 的 skill → `~/.cursor/skills/<name>/`（本机）。若也要给 Cloud Agent 用，再在具体仓放一份，或接受「云端没有这份 skill」。
- 会改 `CONTEXT.md` / ADR / 仓库结构的 skill → 先别装进会进 Cloud 的仓，除非团队同意这套产物。
- ponytail 在 Cursor 上的**官方适配是 rule，不是 skill**（见第 6 节）。

---

## 1. `tdd`（mattpocock）

来源：[`skills/engineering/tdd/SKILL.md`](https://github.com/mattpocock/skills/blob/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/engineering/tdd/SKILL.md)

### 一句话失败模式

挡住「先写实现再补测试、测实现细节、测同义反复、一次铺开全部测试」——让 red → green 产出**值得留下**的行为测试。

### 触发

- README 归类：**Model-invoked**。
- frontmatter **没有** `disable-model-invocation`。
- `description`：用户要 test-first、提到 `red-green-refactor`、或要 integration tests 时使用。
- 人也可以 `/tdd`。`implement` skill 会主动「Use /tdd where possible」。

### 依赖与副作用

- **读**：若存在则读 `CONTEXT.md`（测试名跟领域词）；尊重触及区域的 ADR。
- **不写** `CONTEXT.md` / ADR / hooks。
- **会停下来问人**：写任何测试前必须列出 seams，并等人确认。「No test is written at an unconfirmed seam。」
- **会调其它 skill**：接口形状本身有问题时 `Call the Skill tool with "codebase-design"`（词汇表：module / interface / depth / seam / adapter / leverage / locality）。重构不属于本循环，交给 `code-review`。
- 循环规则：一次一条垂直切片；禁止「先写完全部测试再实现」（horizontal slicing）；禁止同义反复断言；只在约定 seam 测公开行为。

### 附件（除 `SKILL.md` 外必须一起拷）

必须：

- `tests.md`（好坏测试对照）
- `mocking.md`（只在系统边界 mock；DI；SDK 风格接口）

可跳过（Cursor 用不上）：

- `agents/openai.yaml`（仅 Codex 展示名：`display_name: TDD`）

### Cursor 最短安装路径

`~/.cursor/skills/tdd/`（拍平，文件夹名必须是 `tdd`）：

```text
~/.cursor/skills/tdd/SKILL.md
~/.cursor/skills/tdd/tests.md
~/.cursor/skills/tdd/mocking.md
```

不要装进项目 `.cursor/skills/`，除非该仓明确要以 TDD 为默认实现纪律。不需要、也不该跑 `/setup-matt-pocock-skills` 才能用本 skill（setup 是给 triage / to-tickets / issue tracker 用的）。

软依赖（不拷则 seam 词汇会漂）：同级再装 `codebase-design`（见附录）。

### 适合 / 不适合

- 适合：愿意在写测试前对齐 seam、要留下回归、全栈里有明确公开接口的人。
- 不适合：一次性脚本、纯 UI 视觉微调、以及已经把「先对齐方案」做成硬门禁、却不想在实现阶段再被问一遍 seam 的人（会叠两次刹车）。
- 不适合与 ponytail **同时常驻**：见第 7 节。

### 重叠 / 冲突

- vs **diagnosing-bugs**：Phase 5 也是「先回归测试再修」，但 diagnosing-bugs 允许「没有正确 seam 就记下来、不硬写测试」。tdd 则要求先确认 seam。可并存：debug 用 diagnosing-bugs，新功能用 tdd。
- vs **ponytail**：正冲突。ponytail 要求「非平凡逻辑只留 **ONE** runnable check；不要框架、不要 fixtures；一行代码不用测」。tdd 要求在约定 seam 上写行为测试，且 `tests.md` 示例是集成风格。两边会抢「要不要写测试、写多大」。
- vs **improve-codebase-architecture / codebase-design**：互补。tdd 消费 seam 词汇；architecture skill 负责加深模块。
- vs **domain-modeling**：只读 glossary，不改模型。

### 风险

- 模型可能在用户只说「修个 bug」时自动捞起，然后停下来要 seam 确认——和「少停顿、少造轮子」打架。
- 缺 `codebase-design` 时，模型仍会被要求去 Call 它（空调用）。
- 「Refactoring is not part of the loop」会把整理工作推后；若未装 `code-review`，容易留下刚够绿的乱实现。
- `implement` 会驱动 `/tdd` 并在结束时 `/code-review` 再 commit——那是另一套流程，不要误装。

---

## 2. `diagnosing-bugs`（mattpocock）

来源：[`skills/engineering/diagnosing-bugs/SKILL.md`](https://github.com/mattpocock/skills/blob/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/engineering/diagnosing-bugs/SKILL.md)

### 一句话失败模式

挡住「先读代码编故事、没有能对**这个**症状变红的反馈环就开修」——硬 bug / 性能回退必须先有一条已经跑过的、red-capable 命令。

### 触发

- README 归类：**Model-invoked**。
- 无 `disable-model-invocation`。
- `description`：用户说 diagnose / debug this，或报告 broken / throwing / failing / slow。
- 六阶段，只允许在「明确说明理由」时跳过。

### 依赖与副作用

- **读**：`CONTEXT.md`（若有）、触及区域 ADR。
- **不写** glossary / ADR。
- **会写（临时）**：harness、replay fixture、带 `[DEBUG-xxxx]` 前缀的日志、可能的 HITL 脚本副本、最小化 repro。Phase 6 **必须**清掉这些。
- **会写（持久，有条件）**：Phase 5 在「正确 seam」存在时，先把最小化 repro 变成失败测试再修。没有正确 seam 时**记下来**，当作架构问题，不假装有回归。
- 展示命令 / 产物前必须把秘密改成 `<REDACTED>`。
- Phase 3：先列 3–5 条可证伪假设给人看；人不在也继续，不阻塞。
- 性能分支：禁止靠日志猜；先测基准再切分。

### 附件

必须：

- `scripts/hitl-loop.template.sh`（正文点名 `scripts/hitl-loop.template.sh`；最后手段：人在自己终端点，脚本结构化采集 `KEY=VALUE`）

可跳过：

- `agents/openai.yaml`（Codex 展示名）

### Cursor 最短安装路径

```text
~/.cursor/skills/diagnosing-bugs/SKILL.md
~/.cursor/skills/diagnosing-bugs/scripts/hitl-loop.template.sh
```

HITL 模板依赖本机交互式 TTY。Cloud Agent / 无 TTY 环境只能用 Phase 1 的其它环（测试、curl、headless、replay），不要指望 HITL。

### 适合 / 不适合

- 适合：难复现、偶发、性能回退、跨服务。和 Cursor 自带 debug 工作流同向（先证据后改）。
- 不适合：一眼能看出来的 typo、纯文案、以及用户已经给了最小复现+你只想直接补丁的情况（流程偏长）。

### 重叠 / 冲突

- vs **tdd**：Phase 5 会「测试先于修复」，但前提是 seam 正确；否则升格为架构发现。不互斥。
- vs **ponytail**：ponytail 的 bug 规则是「修根因、grep 所有 caller、共享函数上一道守卫」。diagnosing-bugs 要求先有红环再假设。可叠：先环后修，修的时候按 ponytail 修根因。冲突点在「要不要为此写一套测试框架」——ponytail 只要一个 assert/demo。
- vs **improve-codebase-architecture**：没有正确 seam 时，本文明确把锅递给架构侧。

### 风险

- 正文很长，自动触发会吃上下文。
- 模型可能在仓里留下 harness / `[DEBUG-]` 日志；Phase 6 清单必须执行。
- HITL 脚本会把用户输入回显给 agent——不要 `capture` 密码 / cookie。
- 「Be aggressive. Be creative. Refuse to give up.」在无权环境（无 prod、无 HAR）会空转；正文要求此时停下来要 access / 产物 / 临时探测许可。

---

## 3. `improve-codebase-architecture`（mattpocock）

来源：[`skills/engineering/improve-codebase-architecture/SKILL.md`](https://github.com/mattpocock/skills/blob/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/engineering/improve-codebase-architecture/SKILL.md)

### 一句话失败模式

挡住「agent 加速熵增、浅模块堆成泥球」——先扫描加深机会，用人选一条，再 grill 出加深后的模块形状。

README 原话：这是 **survey，不是 rescue**；老仓能找到候选，但不会替你拆泥球。建议「每隔几天跑一次」。

### 触发

- README 归类：**User-invoked**。
- frontmatter：`disable-model-invocation: true`。
- Codex 侧 `agents/openai.yaml` 还有 `allow_implicit_invocation: false`（Cursor 不读这个文件，但意图一致）。
- 只有人打 `/improve-codebase-architecture`（或同名 slash）才会跑。

### 依赖与副作用

- **读**：`CONTEXT.md`、`docs/adr/`、最近 `git log` 热点（用户没点名范围时）。
- **写到 OS 临时目录（不进仓）**：`$TMPDIR` / `/tmp` / `%TEMP%` 下 `architecture-review-<timestamp>.html`，并用 `xdg-open` / `open` / `start` 打开。Tailwind + Mermaid 走 CDN；Mermaid `securityLevel: "loose"`。
- **人选之后会写仓**（通过其它 skill）：
  - `Call the Skill tool with "grilling"`（用户已有转写，本调研不展开）。
  - `Call the Skill tool with "domain-modeling"`：新概念立刻写入 `CONTEXT.md`（没有就懒创建）；用户以「有负载的理由」拒绝时，**提议**写 ADR，不是自动写。
  - 想对比多种接口时再 Call `"codebase-design"` 的 design-it-twice 并行子代理。
- 报告阶段 **禁止**先提议接口；写完 HTML 只问「Which of these would you like to explore?」
- 与 ADR 冲突的候选：只有摩擦大到值得重开才列出，并打警告。

### 附件

必须：

- `HTML-REPORT.md`（scaffold、图模式、必须用的词、禁止替换词）

可跳过：

- `agents/openai.yaml`

运行时硬依赖（不装则第 3 步断）：

- `grilling`（用户已有）
- `domain-modeling`（见第 4 节）
- `codebase-design`（见附录；`DEEPENING.md` + `DESIGN-IT-TWICE.md`）

### Cursor 最短安装路径

slash-only，建议用户级：

```text
~/.cursor/skills/improve-codebase-architecture/SKILL.md
~/.cursor/skills/improve-codebase-architecture/HTML-REPORT.md
```

Cloud Agent 要跑的话必须放进**该仓** `.cursor/skills/improve-codebase-architecture/`，且该环境要能写临时文件、最好能打开浏览器（Cloud 上 `xdg-open` 往往没用，至少要把绝对路径告诉人）。

### 适合 / 不适合

- 适合：中大型、有热点文件、愿意花一小时 grill 一条加深的仓。
- 不适合：绿田、周末玩具、以及「少造轮子、先别refactor」的迭代周。README 自己说它不拆真正的泥球。

### 重叠 / 冲突

- vs **domain-modeling**：编排关系，不是竞争。architecture 是调查 + 选候选；domain-modeling 是改 glossary / 提议 ADR。缺后者，grilling 阶段无法按原文「inline 更新 CONTEXT.md」。
- vs **ponytail / ponytail-audit**：调查对象重叠（都扫仓），极性相反。architecture 要加深（可能合并浅包装、改 seam）；ponytail 要删、缩、禁「只有一个实现的 interface」。同一处改动，一边说「加深成深模块」，一边说 `yagni: AbstractRepository with one implementation. Inline it`。
- vs **tdd**：互补。加深后的 interface 就是下一轮 tdd 的 seam。
- 词汇警察很严：禁止用 component / service / API / boundary 代替 module / interface / seam。和日常全栈口语会打架，报告读起来像一套私教语言。

### 风险

- 临时 HTML 引 CDN + Mermaid `securityLevel: "loose"`：只应在可信本机打开。
- 人选后**会**改 `CONTEXT.md`（经 domain-modeling）。不要在「禁止 agent 写文档」的仓里跑完第 3 步。
- 依赖三件套；只拷这一个文件夹，grill 循环是空的。
- 子代理「有机探索」成本高、输出不稳定。

---

## 4. `domain-modeling`（可选，mattpocock）

来源：[`skills/engineering/domain-modeling/SKILL.md`](https://github.com/mattpocock/skills/blob/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/engineering/domain-modeling/SKILL.md)

### 一句话失败模式

挡住「领域词含糊、代码与口头不一致、决策不落盘」——这是**改模型**的纪律，不是读 `CONTEXT.md`（读 glossary 是其它 skill 各写一行的习惯）。

### 触发

- README 归类：**Model-invoked**。
- 无 `disable-model-invocation`。
- `description`：讨论术语、写/改 `CONTEXT.md`、写/改 ADR 时使用。
- `improve-codebase-architecture` 和 `grill-with-docs`（已跳过）会主动 Call 它。

### 依赖与副作用（这是本清单里副作用最大的之一）

会**懒创建 / 立刻改仓**：

- 默认单上下文：根 `CONTEXT.md` + `docs/adr/0001-slug.md` …
- 若已有根 `CONTEXT-MAP.md`：按图去各上下文的 `CONTEXT.md` 与 `src/<ctx>/docs/adr/`
- 术语一敲定就改 `CONTEXT.md`，禁止攒一批再写。
- `CONTEXT.md` **只能是 glossary**，禁止实现细节、禁止当 spec。
- ADR 必须同时满足：难逆、缺上下文会惊讶、真有取舍。格式见 `ADR-FORMAT.md`（可以短到一段）。

`/setup-matt-pocock-skills` 的 `domain.md` 还规定：其它 engineering skill **发现没有这些文件时保持沉默**，不要主动提议先建；由本 skill / grill-with-docs / improve-architecture 在真正敲定术语时再懒创建。

### 附件

必须：

- `CONTEXT-FORMAT.md`
- `ADR-FORMAT.md`

可跳过：`agents/openai.yaml`

### Cursor 最短安装路径

```text
~/.cursor/skills/domain-modeling/SKILL.md
~/.cursor/skills/domain-modeling/CONTEXT-FORMAT.md
~/.cursor/skills/domain-modeling/ADR-FORMAT.md
```

因为会改仓，更稳的是：**先不要全局装**；只在你明确要养 glossary 的仓放项目级 `.cursor/skills/domain-modeling/`。

### 适合 / 不适合

- 适合：长期产品、多人口头词不统一、已经决定要 `CONTEXT.md`。
- 不适合：少造轮子、仓里已经有 AGENTS.md / 自己的 glossary、以及不想多一类 markdown 产物的人。
- 已有 grill-me 转写的人：grill 阶段若再自动捞起本 skill，会在对话里改文件——和「先对齐、未确认不落盘」可能打架（原文是敲定即写，不等二次确认）。

### 重叠 / 冲突

- vs **improve-codebase-architecture**：后者是调用方；本 skill 是写盘方。装 architecture 不装本 skill = 第 3 步残缺。
- vs **tdd / diagnosing-bugs**：它们只读不写。装本 skill 才会让「读 CONTEXT.md」有东西可读。
- 不与 ponytail 直接抢代码，但会增加文档面；ponytail 会把「没人要的文档」当成 bloat。

### 风险

- Model-invoked + 描述很宽（「讨论术语」）→ 闲聊命名都可能被捞起并开写 `CONTEXT.md`。
- 多仓全局安装会把同一套 glossary 习惯带到不相关的仓。
- ADR 虽说 sparingly，模型仍可能手松。

---

## 5. `handoff`（可选，mattpocock）

来源：[`skills/productivity/handoff/SKILL.md`](https://github.com/mattpocock/skills/blob/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/productivity/handoff/SKILL.md)

### 一句话失败模式

挡住「换会话 / 换 agent 时上下文蒸发、把已有 spec/ADR/diff 再讲一遍」。

### 触发

- README 归类：**User-invoked**。
- `disable-model-invocation: true`
- `argument-hint: "What will the next session be used for?"`
- Codex：`allow_implicit_invocation: false`

### 依赖与副作用

- **写到用户 OS 临时目录，不写当前 workspace。**
- 要有「suggested skills」一节，点名下一个 agent 应 Call 的 skill。
- 已在其它产物里的内容只引用路径/URL，不复制。
- 必须 redact：API key、密码、PII。
- 用户参数 = 下一场焦点，用来裁剪文档。
- 不写 CONTEXT.md / ADR / hooks。

### 附件

无。`agents/openai.yaml` 可跳过。

### Cursor 最短安装路径

```text
~/.cursor/skills/handoff/SKILL.md
```

注意：Cloud Agent 的 `$TMPDIR` 是虚拟机临时盘，人在本机看不见。Cloud 上要用 handoff，应改口令让它写到 workspace 或你指定的路径——**这已偏离原文**，原文坚持 temp、不进仓。

### 适合 / 不适合

- 适合：本机长线程、要交接给另一个本机 agent。
- 不适合：当作「项目进度文档」；不适合当 Cloud → 本机的唯一桥（文件在云盘上）。

### 重叠 / 冲突

- 与其它项几乎正交。suggested skills 可能点名未安装的 skill（空指针）。
- 和 ponytail 无冲突（ponytail 不管散文；README 还建议和 caveman 配对管「怎么说话」）。

### 风险

- 临时文件易丢、不进 git。
- 全文很短，质量全靠模型临场总结。
- Cloud 上「按原文做」= 人拿不到文件。

---

## 6. ponytail 的 Cursor 适配

官方口径（README + [`docs/agent-portability.md`](https://github.com/DietrichGebert/ponytail/blob/2ed6c52c9d7e5e56942508591085fd45dea277d3/docs/agent-portability.md)）：

> Cursor：`.cursor/rules/ponytail.mdc` — **Always-on project rule**。
> instruction-only adapters（Cursor / Windsurf / Cline / Copilot / Kiro / Antigravity）**只有常驻规则，没有 commands**。
> Uninstall：删掉拷过去的 rule 文件。

Cursor **没有** lite/full/ultra 持久化、没有 `PONYTAIL_DEFAULT_MODE`、没有 `~/.config/ponytail/config.json`、没有 hooks。那些是 Claude / Codex / OpenCode / Qoder / Grok 等 plugin 宿主的事。

### 6.1 `.cursor/rules/ponytail.mdc`（官方最短路径）

来源：[`.cursor/rules/ponytail.mdc`](https://github.com/DietrichGebert/ponytail/blob/2ed6c52c9d7e5e56942508591085fd45dea277d3/.cursor/rules/ponytail.mdc)

```yaml
description: Ponytail, lazy senior dev mode. Always pick the simplest solution that works.
globs:
alwaysApply: true
```

正文与根 [`AGENTS.md`](https://github.com/DietrichGebert/ponytail/blob/2ed6c52c9d7e5e56942508591085fd45dea277d3/AGENTS.md) 几乎同一套压缩规则（AGENTS.md 多一句「也适用于改 ponytail 仓库的 agent」）。

梯子（理解问题**之后**再爬，不是代替阅读）：

1. YAGNI：要不要建
2. 仓里已有则复用
3. stdlib
4. 原生平台
5. 已安装依赖
6. 能一行就一行
7. 否则最小能用

另外：bug = 根因不是症状；grep 所有 caller，在共享函数上修一次。

「不懒」清单：读懂问题、信任边界校验、防丢数据的错误处理、安全、无障碍、硬件校准、用户明确要求的东西。非平凡逻辑必须留 **ONE** runnable check（assert demo / 一个小测试文件；**不要框架、不要 fixtures**）。平凡一行不用测。故意削角且有已知天花板时打 `ponytail:` 注释（天花板 + 升级路径）。

### 一句话失败模式

挡住「为日期选择器装组件库、为已有 helper 再写一遍、为尚未存在的变化预留抽象」。

### 触发（Cursor 官方路径）

- **规则常驻**（`alwaysApply: true`），不是 slash，也不是模型按需 skill。
- 无法 `/ponytail off`（那是 plugin 宿主的命令）。要关：删 rule，或不要 `alwaysApply`。

### 依赖与副作用

- **不写** `CONTEXT.md` / ADR / hooks / 配置文件。
- **可能写代码注释**：`ponytail: <ceiling>, <upgrade path>`。
- **可能写最小自检**：一个 `demo()` / `test_*.py` 级文件，明确反感测试框架。
- 不装 Node、不跑 lifecycle hooks。

### 附件

官方 Cursor 路径：**只拷这一个文件**。

```text
<repo>/.cursor/rules/ponytail.mdc
```

跨仓、且希望 Cloud Agent 也遵守：贴进 **User Rules**（Customize → Rules）。Cloud Agent 会吃 User Rules（官方 Cloud best practices）。不要同时再拷一份 skill，否则同一套梯子进上下文两次。

### 适合 / 不适合

- 适合：绿田、容易被 agent 堆依赖的全栈、想少造轮子的人。
- 不适合：已经把「先对齐方案 / 未确认不准写代码」做成硬门禁的人——mdc 比 SKILL.md 温和（只「Question complex requests」），但仍会推「最短能用 diff」，可能在方案未确认时动手。
- 不适合与 **tdd 同时常驻**（测试哲学相反）。

### 6.2 `skills/ponytail/SKILL.md`（官方 Cursor 适配故意没走这条）

来源：[`skills/ponytail/SKILL.md`](https://github.com/DietrichGebert/ponytail/blob/2ed6c52c9d7e5e56942508591085fd45dea277d3/skills/ponytail/SKILL.md)

比 mdc **厚一截**，多出来的都是 Cursor 官方路径拿不到的：

| SKILL.md 有、mdc 没有 | 含义 |
| --- | --- |
| `argument-hint: "[lite\|full\|ultra]"` | 强度档 |
| ACTIVE EVERY RESPONSE；off 仅 `"stop ponytail"` / `"normal mode"` | 会话内持久 |
| 输出格式：代码优先，最多三短行；`[code] → skipped: [X], add when [Y].` | 压散文 |
| 「Never stall on an answer you can default」「Ship the lazy version and question it in the same response」 | **先做后问** |
| lite / full / ultra 对照表 | ultra 会直接挑战需求 |
| `PONYTAIL_DEFAULT_MODE` / `~/.config/ponytail/config.json`（写在 help skill） | Cursor 无 hooks，无效 |
| description 极宽：ANY coding task | 若当 skill 安装，等于准常驻 |

若有人把 SKILL.md 拷进 `~/.cursor/skills/ponytail/`：

- 会变成 **模型自动 + slash**（无 `disable-model-invocation`）。
- lite/full/ultra **不会**跨会话持久（没有 hook 写 flag）。
- 「Never stall / 先做后问」会和用户现有的「先对齐方案」**正撞**。
- 再叠加 `alwaysApply` 的 mdc = 双重施压。

**Cursor 最短路径不要装这份 SKILL.md**，除非你明确要 slash 开关、并接受档位不持久、且准备改 description 收窄自动触发。

### 6.3 附属 skill：和主规则差大的才值得单独说

六个 skill 都只有 `SKILL.md`，无其它附件。官方 Cursor 适配**不带 commands**。若要 slash，须自己拷到 `~/.cursor/skills/<name>/`（文件夹名 = `name`）。

#### `ponytail-review` — 差得大，值得装成 slash

- 失败模式：正确性 review 看不见「能删的复杂度」。
- 只审 **diff**，只打删除清单，**不改代码**。
- 标签：`delete` / `stdlib` / `native` / `yagni` / `shrink`；结尾 `net: -<N> lines possible.`
- 明确不管正确性 / 安全 / 性能。
- 一条 smoke / assert 自检不算 bloat，禁止当成删除项。
- 与 mattpocock `code-review`（未在本次必研清单）正交：一边 Standards+Spec，一边只砍复杂度。

#### `ponytail-audit` — review 的全仓版

- 同样五个标签，按砍幅排序，`net: -<N> lines, -<M> deps`。
- 与 `improve-codebase-architecture` 最容易抢同一片仓：一边删抽象，一边加深模块。
- one-shot，不改代码。

#### `ponytail-debt` — 只有你真打了 `ponytail:` 才有用

- `grep -rnE '(#|//) ?ponytail:'`，收成账本。
- 默认只读；人要求才写 `PONYTAIL-DEBT.md`。
- 没打过 `ponytail:` 注释 → 每次都是 `No ponytail: debt. Clean ledger.`

#### `ponytail-gain` — 先别装

- 只渲染 README 里的 **旧 single-shot** 分数（5 题 × 3 模型，LOC 降 80–94%）。
- 同一份 README 已改口：agentic 基准均值约 **-54% LOC**，80–94% 是单任务天花板。
- 正文禁止报「本仓省了多少行」。对日常开发零信息。

#### `ponytail-help` — 先别装

- 卡片里大量 `/plugin`、auto-update、`PONYTAIL_DEFAULT_MODE`。
- Cursor 官方路径用不上。装了会教模型一套无效操作。

### 6.4 风险（ponytail 整体）

- 常驻 rule 每轮进上下文，和用户已有的长 User Rules 叠 token。
- 「ONE check, no frameworks」会拆掉认真的测试基建。
- 「No interface with one implementation」（SKILL.md 原文；mdc 写成「No abstractions that weren't explicitly requested」）会对抗 tdd/codebase-design 的 seam/adapter。
- 基准数字有两套叙事；不要用 gain skill 当决策依据。
- 与「先对齐、确认后再写」：mdc 较轻，SKILL.md 较凶。官方 Cursor 路径是较轻的那种。

---

## 7. 交叉重叠与冲突（矩阵）

| | tdd | diagnosing-bugs | improve-architecture | domain-modeling | handoff | ponytail rule |
| --- | --- | --- | --- | --- | --- | --- |
| **tdd** | — | 互补（红环 vs 实现循环） | 互补（seam 来自加深） | 只读 glossary | 无 | **冲突：测试规模** |
| **diagnosing-bugs** | 互补 | — | 无 seam 时升级 | 只读 | 无 | 弱冲突（要不要框架测试） |
| **improve-architecture** | 互补 | 接收「无 seam」 | — | **调用方 / 写盘方** | 无 | **极性相反（加深 vs 删）** |
| **domain-modeling** | 被读 | 被读 | 被调用 | — | 无 | 弱（多文档 vs 少文件） |
| **handoff** | 无 | 无 | 无 | 无 | — | 无 |
| **ponytail rule** | **冲突** | 弱冲突 | **冲突** | 弱 | 无 | — |

### ponytail vs tdd（最需要拍板）

| | tdd | ponytail（mdc / SKILL） |
| --- | --- | --- |
| 测试 | 约定 seam 上的行为测试；`tests.md` 是集成风 | 一个 assert/demo；禁框架与 fixtures |
| 抽象 | 要 seam / adapter；「两个 adapter 才是真 seam」 | 未点名的抽象一律不要；单实现 interface 要内联 |
| 节奏 | 先确认 seam，再 red → green | 最短能用 diff；SKILL 版还「先做后问」 |
| 重构 | 不在循环内，交给 code-review | 删除优于新增 |

不要两个都 `alwaysApply` / 都允许模型随便捞。若两个都想要：ponytail 用**项目 rule**（可关），tdd 保持 model-invoked 且 description 收窄到「用户明确说 test-first」。

### domain-modeling vs improve-architecture

不是二选一。architecture 是「找候选 + 选完再 grill」；domain-modeling 是「术语/ADR 落盘」。只装 architecture = HTML 能出，第 3 步按原文做不完。只装 domain-modeling = 没有扫描/可视化。

---

## 8. 和「已有工作流」的关系（本调研读者）

用户已有：grill-me 转写、以及「先对齐方案 / 未确认不准写代码」。

- **tdd** 的「先确认 seam」和先对齐**同向**，但会在实现阶段再问一轮。
- **diagnosing-bugs** 和 Cursor debug 纪律同向（先环后假设）。
- **ponytail SKILL.md** 的「Never stall / 先做后问」和先对齐**反向**。官方 Cursor 的 mdc **没有**这句，只有「Question complex requests」。
- **improve-architecture** 的 grilling 会复用已有 grilling 转写；不要再装一份 grill-me。
- **domain-modeling** 的「敲定即写」可能在未明确「可以改 CONTEXT.md」时落盘。
- **不要跑** `/setup-matt-pocock-skills`：它会改 `CLAUDE.md`/`AGENTS.md`、写 `docs/agents/issue-tracker.md` 等。那是整套 engineering 流水线（triage / to-tickets / implement）的脚手架，和「少造轮子」相反。tdd / diagnosing-bugs **不依赖**它。

---

## 9. 推荐安装顺序（Cursor 重度 / 全栈 / 少造轮子，不含 grill-me）

原则：先装「不写仓、不全局抢实现权」的；把会改仓或会和先对齐打架的放后面或不当常驻。

### 第 1 刀（建议现在就可以）

1. **`diagnosing-bugs` → `~/.cursor/skills/diagnosing-bugs/`**  
   拷 `SKILL.md` + `scripts/hitl-loop.template.sh`。  
   理由：填的是现有流程缺口（硬 bug 反馈环），不写 CONTEXT.md，和先对齐不抢「开没开写业务代码」。

2. **`tdd` → `~/.cursor/skills/tdd/`**  
   拷 `SKILL.md` + `tests.md` + `mocking.md`。  
   理由：实现阶段的测试纪律；自动触发面比 ponytail 窄（要提到 test-first / red-green / integration tests）。  
   可选：若经常要它解释 seam，再装 `codebase-design`（附录）。

3. **`handoff` → `~/.cursor/skills/handoff/`**（可选但便宜）  
   只 `SKILL.md`。仅 slash。本机交接有用；Cloud 上按原文会把文件写到 VM temp。

### 第 2 刀（按需，不要全局 always-on）

4. **ponytail 用官方 Cursor 路径，且不要当 User Rule 常驻。**  
   - 只在「容易被 agent 堆依赖」的**具体仓**放 `.cursor/rules/ponytail.mdc`。  
   - 不要拷 `skills/ponytail/SKILL.md`（避免「先做后问」+ 双重施压）。  
   - 该仓若同时开 tdd，接受测试哲学冲突，或临时删 mdc。  
   - 已有先对齐门禁时，mdc 比 SKILL 可共存，但仍会推最短 diff——用项目级，好关。

5. **`ponytail-review` → `~/.cursor/skills/ponytail-review/`**（slash）  
   实现完要砍 bloat 时用。比常驻 ponytail 安全：只出删除清单，不改代码。

### 第 3 刀（真要养领域语言 / 做架构调查再装）

6. **`codebase-design`**（附录）+ **`domain-modeling`**（项目级，不要全局）+ **`improve-codebase-architecture`**（用户级 slash）。  
   三件套一起才完整。单独装 architecture 没有第 3 步。

---

## 10. 先别装

| 项 | 原因 |
| --- | --- |
| `grill-me` / `grilling` / `grill-with-docs` | 用户已有转写 |
| `/setup-matt-pocock-skills` 以及它会写的 `docs/agents/*`、改 AGENTS.md | 整套流水线脚手架，造轮子 |
| `implement` / `to-spec` / `to-tickets` / `wayfinder` / `triage` | 同上，本次未点名且会驱动 tdd+commit |
| `ponytail` 的 `SKILL.md` 当 Cursor skill | 官方适配是 rule；SKILL 含「先做后问」和无效档位 |
| ponytail **User Rule 全局 always-on** + 已有先对齐门禁 | 实现权打架；token 再叠一层 |
| `ponytail.mdc` **和** `skills/ponytail` 同时装 | 同一梯子进两次 |
| `ponytail-gain` | 展示过时 single-shot 80–94%，和当前 agentic 均值 -54% 不是一回事 |
| `ponytail-help` | `/plugin`、config.json，Cursor 用不上 |
| `ponytail-debt` | 没打 `ponytail:` 注释就是空跑 |
| `ponytail-audit`（初期） | 和以后才装的 architecture 抢全仓叙事；先用 review 看 diff 即可 |
| plugin hooks / `PONYTAIL_DEFAULT_MODE` / `~/.config/ponytail/config.json` | Cursor 无此适配 |
| 各 skill 的 `agents/openai.yaml` | 仅 Codex |
| 用 `npx skills add` 装到默认全局目录然后不管 | 可能只落到 `~/.agents/skills`，Cursor 不一定看见 |
| 把 user-level skill 当成 Cloud Agent 也有 | 官方：不会复制 `~/.cursor/skills` |
| 未决定养 glossary 就全局装 `domain-modeling` | 会在随便一个仓懒创建 `CONTEXT.md` |
| 只装 `improve-codebase-architecture`、不装 `codebase-design` + `domain-modeling` | 第 3 步按原文做不完 |

---

## 附录 A. 隐藏依赖 `codebase-design`

不是本次必研五项，但 **tdd** 和 **improve-codebase-architecture** 正文都硬引用。

- 路径：[`skills/engineering/codebase-design/`](https://github.com/mattpocock/skills/tree/6654f6b60cd9d5be8b54c6fafe44346dabeb3b76/skills/engineering/codebase-design)
- 触发：Model-invoked；设计/加深接口、找 seam、要可测/可被 AI 导航时。
- 附件必须一起拷：`DEEPENING.md`、`DESIGN-IT-TWICE.md`
- 安装：`~/.cursor/skills/codebase-design/`（文件夹名 `codebase-design`）
- 副作用：设计接口时可能拉并行子代理（`DESIGN-IT-TWICE.md`）；不写 CONTEXT.md。
- 与 ponytail：深度模块 ≠ 预留抽象；「一个 adapter = 假 seam」其实和 ponytail「单实现不要 interface」**部分同向**。冲突发生在 tdd 为了可测先抽出 seam 的时候。

---

## 附录 B. 附件清单总表

| Skill | 必须拷 | 可跳过 | 运行时还会碰的仓内文件 |
| --- | --- | --- | --- |
| tdd | `tests.md`, `mocking.md` | `agents/openai.yaml` | 读 `CONTEXT.md` / ADR |
| diagnosing-bugs | `scripts/hitl-loop.template.sh` | `agents/openai.yaml` | 读 CONTEXT/ADR；临时 harness / `[DEBUG-]`；或然回归测试 |
| improve-codebase-architecture | `HTML-REPORT.md` | `agents/openai.yaml` | 读 CONTEXT/ADR；HTML 在 temp；grill 后经 domain-modeling 写 CONTEXT/或然 ADR |
| domain-modeling | `CONTEXT-FORMAT.md`, `ADR-FORMAT.md` | `agents/openai.yaml` | **写** `CONTEXT.md` / `CONTEXT-MAP.md` / `docs/adr/` |
| handoff | （无） | `agents/openai.yaml` | OS temp，不进仓 |
| ponytail（Cursor 官方） | `.cursor/rules/ponytail.mdc` 这一份 | 全部 `skills/ponytail*`、hooks、help/gain | 或然 `ponytail:` 注释、一个最小自检 |
| ponytail-review / audit / debt | 仅 `SKILL.md`（若你坚持要 slash） | — | debt 经询问才写 `PONYTAIL-DEBT.md` |
| codebase-design（隐藏） | `DEEPENING.md`, `DESIGN-IT-TWICE.md` | `agents/openai.yaml` | 不写 glossary |

---

## 附录 C. 原文索引

- mattpocock README：User-invoked vs Model-invoked；失败模式 #3 对应 tdd / diagnosing-bugs，#4 对应 improve-architecture；安装哲学是「小、可改、可组合」，反对 GSD/BMAD/Spec-Kit 式接管全程。
- mattpocock `implement`：`disable-model-invocation: true`；「Use /tdd where possible, at pre-agreed seams」然后 `/code-review` 再 commit。未列入推荐。
- ponytail README：Cursor 只拷 rule；commands 需要 skill-capable host。agentic 基准（FastAPI+React 模板，Haiku 4.5，12 题 n=4）：相对无 skill，LOC **-54%**、tokens -22%、cost -20%、time -27%、safe 100%。single-shot 80–94% 已降级为「隔离生成、含散文注水」。
- Cursor Skills：`disable-model-invocation`、目录表、user-level 不进 Cloud。
- Cursor Rules / Cloud best practices：User Rules 跨仓且 Cloud 会吃；项目 rule 跟仓走。
