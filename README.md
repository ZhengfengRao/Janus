# Janus

Janus 是 macOS 上的全能受托 Agent：用户给出目标、授权和风险预算，Janus 自主理解、计划、操作现有 App 并完成任务；超出授权或责任边界时升级确认。

Janus is a full-capability entrusted Agent for macOS: users give goals, authorization, and risk budgets; Janus understands, plans, operates existing apps, completes tasks, and escalates when it exceeds delegated authority or responsibility boundaries.

---

## 中文

### 项目状态

本仓库当前是 Janus 的产品、设计、架构与原型文档工作区，不是已经打包发布的生产应用。

仓库中的 HTML 原型属于探索性 UI / 交互工件，用来验证产品边界、状态流和界面表达；它们不代表一个已完成、可安装、可长期运行的 macOS 应用。

Janus 的长期产品愿景是 **macOS 全能受托 Agent**：用户表达目标、约束、授权和风险预算，Janus 在既有 App 中自主完成跨 App 任务，并在超出授权、风险预算或责任边界时升级确认。当前验证切片仍从受控任务开始，目的是证明 runtime、自主执行、恢复、审计和授权治理，而不是把低风险草稿/准备闭环误认为最终产品身份。

### Janus 解决什么问题

现代软件把大量时间转化为界面劳动：用户知道自己要什么，却必须在多个 App 之间反复打开、切换、查找、搬运信息、起草内容、检查状态。

Janus 的价值不是只把用户带到草稿前，而是在授权范围内完成电脑任务：理解目标，制定路径，操作现有 App，验证结果，汇报已完成事项；只有当行动需要新的判断、身份承担、责任确认，或超出既定授权 / 风险预算时，才向用户升级确认。它保护用户主权：

- **目的**：任务边界由用户目标、约束和授权决定，不由当前界面或模型臆测扩张。
- **判断**：Janus 可自主执行已授权路径；新的价值判断、责任判断或边界变更必须回到用户。
- **身份**：以用户身份产生外部影响，需要一次性确认或明确的 standing authorization 与风险预算覆盖。
- **责任**：审计记录必须说明做了什么、没做什么、基于什么授权，以及为什么继续、完成或停下。

### Janus 是什么 / 不是什么

| Janus 是 | Janus 不是 |
| --- | --- |
| 全能受托 Agent | 聊天机器人 |
| 被授权的电脑任务执行者 | 启动器 |
| 自主策略 / runtime / 风险预算系统 | 单 App 插件 |
| 跨 App 任务执行者 | 固定 RPA 脚本集合 |
| 可撤回、可审计、可恢复的数字工作者 | 无管理的黑箱 Agent |
| 在授权范围内完成任务、超界升级确认的系统 | 把模型置信当作行动权力的工具 |

### 自主性阶梯

Janus 的自主性不是“全自动或全手动”的二选一，而是由授权范围、风险预算、验证能力和用户治理共同决定：

| Level | 名称 | Janus 可做什么 | 边界 |
| --- | --- | --- | --- |
| L0 | Observe / Suggest | 观察当前上下文、解释状态、提出建议 | 不改变状态 |
| L1 | Assist / Prepare | 打开、搜索、整理、起草、预填 | 结果可检查、可编辑 |
| L2 | Auto-complete reversible tasks | 自动完成可撤销、本地或低外部影响任务 | 必须可验证、可恢复 |
| L3 | Authorized recurring execution | 按 standing authorization 执行重复任务 | 受对象、动作、频率、期限约束 |
| L4 | Risk-budgeted autonomous execution | 在风险预算内完成外部可见或跨 App 多步任务 | 超出预算、对象或影响范围即升级 |
| L5 | Escalation for high-stakes boundaries | 金钱、法律、账号安全、不可逆、组织责任等高利害边界 | 需要明确授权、组织策略或硬停审查 |

### 核心产品界面

Janus 只引入三个顶层产品界面，不应额外扩张出新的顶层产品面：

1. **Janus Entry**
   全局 / 当前任务入口，也是停止、恢复、重新聚焦当前工作的锚点。它提供最小可见状态和控制，不承载完整任务解释、托管治理或新任务工作台。

2. **Janus Surface**
   当前受托任务的生命周期界面，覆盖理解、计划、自主策略 / 风险预算检查、执行、验证、升级确认、停止、接管、回顾等状态。

3. **Delegation Center**
   跨任务的授权、风险预算、standing authorization、记忆、审计、恢复和信任治理中心。它管理长期委托关系，但不替代当前任务的现场判断与升级确认。

### 授权与信任边界

Janus 的核心边界是：能力不等于授权，自动化不等于主权转移。授权、停止、接管、审计和恢复不是降低自动化的刹车，而是让更高自主性成立的控制基础设施。

硬规则：

- **Capability ≠ Permission**：系统能做某事，不代表已经被允许做。
- **Memory ≠ Permission**：记住偏好不等于获得长期授权。
- **Learning Candidate ≠ Memory**：学习候选必须经过治理，不能自动成为记忆。
- **Current context ≠ task boundary**：当前上下文不是任务边界，任务边界来自用户目标、约束、授权和风险预算。
- **Single confirmation ≠ long-term authorization**：一次确认不等于长期授权；长期授权必须有范围、期限、撤销路径和审计。
- **Model confidence is not verification**：模型置信不是验证；必须有可审计证据或状态检查。
- 外部影响不再等同于每次都手动确认；它必须被一次性明确确认、standing authorization 或风险预算策略覆盖。未覆盖、超范围或责任边界不清的外部影响必须升级确认或停下。
- 审计必须记录 **done** 和 **not done**：不仅说明完成了什么，也要说明没有发送、没有提交、没有删除、没有越权，以及使用了哪条授权策略。

### 代表性用户场景

以下只是代表性场景，不是 Janus 的本体，也不意味着 Janus 只服务某个 App：

> 用户长按 Janus Entry，说：“帮我准备明天的客户会：整理下载目录里的最新材料，更新项目文档，给团队发一条例行同步，日程如果还没建就创建一个。例行同步可以按我之前授权的模板发送；客户承诺类内容需要先问我。”

Janus 应该理解目标和约束，检查已有 standing authorization，组织文件、打开并更新文档、验证日程状态、在授权范围内发送例行同步；如果内容涉及客户承诺、缺少授权对象、风险预算不足或验证失败，应在 Janus Surface 中升级确认。用户可以随时停止、接管目标 App 现场、修改授权或查看审计。

另一个低风险验证切片仍然有效：

> “打开微信，给张三起草一条消息，发送前让我确认。”

这个场景用于验证跨 App runtime、目标定位、草稿创建、未发送证明和确认缝，但它不是 Janus 的长期产品上限。

### 技术方向

Janus 的技术方向是 **Evidence-first Native Runtime**：先以原生 macOS 运行时和结构化信号建立可验证行动，再逐步补充 OCR、视觉和模拟输入。

关键方向：

- 任务中心架构：围绕 Task / Plan / Step / Autonomy Policy / Permission / Risk Budget / Verification / Audit 建模。
- 原生 macOS runtime 优先，而不是先做浏览器插件或单 App 插件。
- 优先使用结构化信号、系统 API、Accessibility、应用状态与可验证回读。
- OCR / vision / simulated input 是补充路径，不是第一原则。
- 使用通用 primitives + thin adapters，而不是为每个 App 编写沉重脚本。
- 每一步都需要可解释的授权、风险预算、验证、恢复和审计状态。

### 仓库地图

```text
docs/
  janus-complete-product-design.md
  janus-product-experience-design.md
  janus-technical-architecture-design.md
  janus-core-technology-validation-report.md
  janus-agent-product-design.md
  product_review.md
.superpowers/brainstorm/
  prototype HTML explorations
```

### 推荐阅读顺序

1. `README.md`
2. `docs/janus-complete-product-design.md`
3. `docs/janus-product-experience-design.md`
4. `docs/janus-technical-architecture-design.md`
5. `docs/janus-core-technology-validation-report.md`
6. `docs/product_review.md`

### 当前验证重点 / MVP 方向

当前 MVP 是全能受托 Agent 的分阶段验证，不是对长期愿景的降级。建议采用三条验证轨道：

- **Track A：即时跨 App 执行**：从任意工作现场发起，完成打开、定位、整理、起草、预填、验证、停止 / 接管。
- **Track B：授权的自动重复 / 可撤销任务**：在 standing authorization 和风险预算内，自动完成例行、可恢复、可审计的任务，不要求每一步确认。
- **Track C：复杂多步目标准备与执行**：处理跨 App、多对象、多约束目标，在授权范围内完成任务，遇到责任、判断或风险超界时升级确认。

早期低风险 / 草稿 / 准备任务仍是第一验证切片，用来证明 runtime、自主策略、恢复和审计可靠；它不是 Janus 的最终产品身份。

---

## English

### Project status

This repository is currently a product, design, architecture, and prototype documentation workspace for Janus. It is not a packaged production app.

The existing HTML previews are exploratory UI artifacts. They are useful for testing product boundaries, lifecycle states, and interaction language, but they should not be treated as a finished macOS application.

The long-term product vision of Janus is a **full-capability entrusted Agent for macOS**: users express goals, constraints, authorization, and risk budgets; Janus autonomously completes cross-app tasks in existing apps; it escalates when delegated authority, risk budget, or responsibility boundaries are exceeded. The current validation slice still starts with constrained tasks to prove runtime, autonomy, recovery, audit, and authorization governance. That slice is not the final product identity.

### Problem and value

Modern software often turns users into interface laborers. A user may already know the goal, but still has to open apps, switch windows, search, copy, paste, draft, and check results across multiple tools.

Janus is not only meant to stop at drafts. Its default aspiration is to complete computer tasks when authorized: understand the goal, plan the path, operate existing apps, verify outcomes, and report what happened. It asks only when a new judgment, identity commitment, responsibility boundary, or authorization / risk-budget exception appears. It preserves user sovereignty across four dimensions:

- **Purpose**: the task boundary comes from user goals, constraints, and authorization, not from the current screen or model extrapolation.
- **Judgment**: Janus may autonomously execute authorized paths; new value judgments, responsibility judgments, or boundary changes return to the user.
- **Identity**: external impact under the user’s identity requires either explicit one-time confirmation or standing authorization with a risk budget.
- **Responsibility**: audit records must show what was done, what was not done, what authorization was used, and why Janus continued, completed, or stopped.

### What Janus is / is not

| Janus is | Janus is not |
| --- | --- |
| A full-capability entrusted Agent | A chatbot |
| An authorized computer-task operator | A launcher |
| An autonomy policy / runtime / risk-budget system | A single-app plugin |
| A cross-app task executor | A fixed collection of RPA scripts |
| A recoverable, auditable digital worker | An unmanaged black-box agent |
| A system that completes tasks within delegated authority and escalates outside it | A tool that treats model confidence as authority |

### Autonomy model

Janus autonomy is not a binary choice between fully manual and fully automatic. It is determined by authorization scope, risk budget, verification capability, and user governance:

| Level | Name | What Janus may do | Boundary |
| --- | --- | --- | --- |
| L0 | Observe / Suggest | Observe current context, explain state, suggest next steps | No state change |
| L1 | Assist / Prepare | Open, search, organize, draft, prefill | Result remains inspectable and editable |
| L2 | Auto-complete reversible tasks | Complete reversible local or low-impact tasks automatically | Must be verifiable and recoverable |
| L3 | Authorized recurring execution | Execute recurring tasks under standing authorization | Bound by object, action, frequency, and duration |
| L4 | Risk-budgeted autonomous execution | Complete externally visible or multi-app work within a risk budget | Escalate outside budget, object, or impact scope |
| L5 | Escalation for high-stakes boundaries | Money, legal, account security, irreversible, or organizational responsibility boundaries | Requires explicit authorization, organizational policy, or hard-stop review |

### Core product surfaces

Janus has only three canonical top-level product surfaces. New top-level surfaces should not be introduced without changing the product ontology.

1. **Janus Entry**
   The global and current-work entry point. It is also the anchor for stop, recovery, and refocus. It should expose minimal state and control, not a full task explanation, governance console, or new-task dashboard.

2. **Janus Surface**
   The lifecycle interface for the current entrusted task: understanding, planning, autonomy policy / risk-budget check, execution, verification, escalation, stop, takeover, and review.

3. **Delegation Center**
   The cross-task governance surface for authorization, risk budgets, standing authorization, memory, audit, recovery, and trust. It manages long-term delegation relationships, but it must not replace situated judgment and escalation for the current task.

### Authorization and trust boundaries

The central boundary is simple: capability is not authorization, and automation is not a transfer of user sovereignty. Authorization, stop, takeover, audit, and recovery are not brakes that make automation smaller; they are the control infrastructure that enables higher autonomy.

Hard rules:

- **Capability ≠ Permission**: being able to do something does not mean Janus is allowed to do it.
- **Memory ≠ Permission**: remembering a preference is not long-term authorization.
- **Learning Candidate ≠ Memory**: a learning candidate must be governed before becoming memory.
- **Current context ≠ task boundary**: the current context is not the task boundary; user goals, constraints, authorization, and risk budgets define the boundary.
- **Single confirmation ≠ long-term authorization**: one confirmation does not grant durable permission; durable authorization requires scope, duration, revocation, and auditability.
- **Model confidence is not verification**: model confidence must not replace auditable evidence or state checks.
- External impact does not always imply manual confirmation each time. It must be covered by explicit one-time confirmation, standing authorization, or a risk-budget policy. Uncovered, out-of-scope, or unclear responsibility boundaries must escalate or stop.
- Audit must record both **done** and **not done**: not only what Janus completed, but also what it did not send, submit, delete, or authorize, and which authorization policy was used.

### Representative user scenario

This scenario is representative only. It is not the ontology of Janus, and it should not turn any specific app into the center of the product model.

> The user long-presses Janus Entry and says: “Prepare tomorrow’s client meeting: organize the latest materials from Downloads, update the project doc, send the team a routine update, and create a calendar event if it doesn’t exist. The routine update can be sent under my existing template authorization; anything that creates a client commitment should ask me first.”

Janus should understand the goal and constraints, check standing authorization, organize files, open and update the document, verify calendar state, and send the routine update within authorization. If the content creates a client commitment, lacks authorized recipients, exceeds the risk budget, or fails verification, Janus should escalate in Janus Surface. The user can stop, take over in the target app, modify authorization, or inspect the audit trail at any time.

A lower-risk validation slice remains valid:

> “Open WeChat, draft a message to Zhang San, and ask me before sending.”

That scenario validates cross-app runtime, target selection, draft creation, proof of non-sending, and confirmation seams. It is not the ceiling of the product.

### Technical direction

Janus points toward an **Evidence-first Native Runtime**: build verifiable native macOS action first, then use OCR, vision, or simulated input as fallback or supplemental paths.

Key directions:

- Task-centered architecture around Task / Plan / Step / Autonomy Policy / Permission / Risk Budget / Verification / Audit.
- Native macOS runtime first, rather than starting as a browser extension or single-app plugin.
- Prefer structured signals, system APIs, Accessibility, application state, and verifiable readback.
- Treat OCR, vision, and simulated input as supporting tools, not first principles.
- Use generic primitives plus thin adapters instead of heavy per-app scripts.
- Every step should carry explicit authorization, risk budget, verification, recovery, and audit state.

### Repository map

```text
docs/
  janus-complete-product-design.md
  janus-product-experience-design.md
  janus-technical-architecture-design.md
  janus-core-technology-validation-report.md
  janus-agent-product-design.md
  product_review.md
.superpowers/brainstorm/
  prototype HTML explorations
```

### How to read this repository

1. `README.md`
2. `docs/janus-complete-product-design.md`
3. `docs/janus-product-experience-design.md`
4. `docs/janus-technical-architecture-design.md`
5. `docs/janus-core-technology-validation-report.md`
6. `docs/product_review.md`

### Validation focus / MVP direction

The current MVP direction is staged validation of the full-capability entrusted Agent vision, not a downgrade of that vision. Recommended validation tracks:

- **Track A: Immediate cross-app execution**: start from any work context and complete opening, locating, organizing, drafting, prefilling, verification, stop, and takeover.
- **Track B: Authorized automatic recurring / reversible tasks**: under standing authorization and risk budgets, complete routine, recoverable, auditable work without per-step confirmation.
- **Track C: Complex multi-step goal preparation and execution**: handle multi-app, multi-object, constrained goals; complete within authorization and escalate when judgment, responsibility, or risk exceeds scope.

Early low-risk drafting and preparation tasks are still the first validation slice. They prove runtime, autonomy policy, recovery, and audit reliability. They are not Janus’s final product identity.
