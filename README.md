# Janus

Janus 是 macOS 上的通用受托行动层，用于把跨 App 的低风险机械路径推进到可检查、可编辑、可确认的结果。

Janus is a macOS universal entrusted action layer that helps users delegate low-risk cross-app mechanical paths while preserving confirmation, control, verification, and auditability.

---

## 中文

### 项目状态

本仓库当前是 Janus 的产品、设计、架构与原型文档工作区，不是已经打包发布的生产应用。

仓库中的 HTML 原型属于探索性 UI / 交互工件，用来验证产品边界、状态流和界面表达；它们不代表一个已完成、可安装、可长期运行的 macOS 应用。

当前优先级不是构建一个“桌面超级智能体”，而是验证一个窄范围、低风险的跨 App 草稿 / 准备闭环：Janus 能否把用户从重复的打开、切换、搜索、复制、粘贴、起草、检查中解放出来，并把结果停在可检查、可编辑、可确认的位置。

### Janus 解决什么问题

现代软件把大量时间转化为界面劳动：用户知道自己要什么，却必须在多个 App 之间反复打开、切换、查找、搬运信息、起草内容、检查状态。

Janus 的价值不是替用户作主，而是承担低风险、机械性的跨 App 路径，把用户带到一个可以检查、编辑、确认或停止的结果前。它保护用户主权：

- **目的**：任务边界由用户意图决定，不由当前界面或模型臆测扩张。
- **判断**：重要判断保留给用户，Janus 负责准备证据和结果。
- **身份**：以用户身份产生外部影响前必须确认。
- **责任**：审计记录必须说明做了什么、没做什么，以及为什么停下。

### Janus 是什么 / 不是什么

| Janus 是 | Janus 不是 |
| --- | --- |
| 通用受托行动层 | 聊天机器人 |
| 跨 App 机械路径助手 | 启动器 |
| evidence-first 的运行时概念 | 单 App 插件 |
| 用户可控的委托界面 | RPA 脚本集合 |
| 面向确认、验证、审计的行动层 | 超级 App |
| 把结果推进到可检查 / 可编辑 / 可确认状态的系统 | 绕过用户确认的全自动代理 |

### 核心产品界面

Janus 只引入三个顶层产品界面，不应额外扩张出新的顶层产品面：

1. **Janus Entry**
   全局 / 当前任务入口，也是停止、恢复、重新聚焦当前工作的锚点。它提供最小可见状态和控制，不承载完整任务解释、托管治理或新任务工作台。

2. **Janus Surface**
   当前受托任务的生命周期界面，覆盖理解、计划、权限 / 风险、执行、验证、确认、停止、回顾等状态。

3. **Delegation Center**
   跨任务的权限、记忆、审计和信任治理中心。它管理长期委托关系，但不替代当前任务的确认流。

### 安全与信任边界

Janus 的核心边界是：能力不等于授权，自动化不等于主权转移。

硬规则：

- **Capability ≠ Permission**：系统能做某事，不代表已经被允许做。
- **Memory ≠ Permission**：记住偏好不等于获得长期授权。
- **Learning Candidate ≠ Memory**：学习候选必须经过治理，不能自动成为记忆。
- **Current context ≠ task boundary**：当前上下文不是任务边界，任务边界来自用户意图和确认。
- **Single confirmation ≠ long-term authorization**：一次确认不等于长期授权。
- **Model confidence is not verification**：模型置信不是验证；必须有可审计证据或状态检查。
- 所有产生外部影响的动作都必须确认，包括发送、提交、上传、发布、审批、删除 / 覆盖、权限变更、支付、身份相关动作。
- 审计必须记录 **done** 和 **not done**：不仅说明完成了什么，也要说明没有发送、没有提交、没有删除、没有越权。

### 代表性用户场景

以下只是代表性场景，不是 Janus 的本体，也不意味着 Janus 只服务某个 App：

> 用户长按 Janus Entry，说：“打开微信，给张三起草一条消息，发送前让我确认。”

Janus 应该打开或切换到目标 App，搜索联系人，创建草稿，验证草稿存在且消息尚未发送，然后停在确认点。用户可以检查、编辑、确认发送，或停止任务。

### 技术方向

Janus 的技术方向是 **Evidence-first Native Runtime**：先以原生 macOS 运行时和结构化信号建立可验证行动，再逐步补充 OCR、视觉和模拟输入。

关键方向：

- 任务中心架构：围绕 Task / Plan / Step / Policy / Permission / Risk / Verification / Audit 建模。
- 原生 macOS runtime 优先，而不是先做浏览器插件或单 App 插件。
- 优先使用结构化信号、系统 API、Accessibility、应用状态与可验证回读。
- OCR / vision / simulated input 是补充路径，不是第一原则。
- 使用通用 primitives + thin adapters，而不是为每个 App 编写沉重脚本。
- 每一步都需要可解释的风险、权限、验证和审计状态。

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

当前推荐 MVP 不是通用桌面自治系统，而是窄范围跨 App 草稿 / 准备闭环：

- 选择 4-6 个高频、低风险、可明确停在确认前的任务。
- 验证 Janus 是否减少界面劳动，而不是增加新的管理负担。
- 衡量到达可检查结果的时间，而不是模型一次性回答的流畅度。
- 保证零未确认外部影响。
- 验证用户直接接管、停止、恢复是否可靠。
- 检查审计是否足够清楚地说明 done / not done。

---

## English

### Project status

This repository is currently a product, design, architecture, and prototype documentation workspace for Janus. It is not a packaged production app.

The existing HTML previews are exploratory UI artifacts. They are useful for testing product boundaries, lifecycle states, and interaction language, but they should not be treated as a finished macOS application.

The current priority is not to build a “desktop super agent.” The priority is to validate a narrow, low-risk cross-app drafting and preparation loop: can Janus reduce repetitive interface labor and bring the user to an inspectable, editable, confirmable result?

### Problem and value

Modern software often turns users into interface laborers. A user may already know the goal, but still has to open apps, switch windows, search, copy, paste, draft, and check results across multiple tools.

Janus is meant to take over low-risk mechanical paths, not human judgment. It should prepare the path and stop at a result the user can inspect, edit, confirm, or reject. It preserves user sovereignty across four dimensions:

- **Purpose**: the task boundary comes from user intent, not from the current screen or model extrapolation.
- **Judgment**: important decisions remain with the user; Janus prepares evidence and draft outcomes.
- **Identity**: actions that create external impact under the user’s identity require confirmation.
- **Responsibility**: audit records must show what was done, what was not done, and why the system stopped.

### What Janus is / is not

| Janus is | Janus is not |
| --- | --- |
| A universal entrusted action layer | A chatbot |
| A cross-app mechanical path assistant | A launcher |
| An evidence-first runtime concept | A single-app plugin |
| A user-controlled delegation interface | A collection of RPA scripts |
| An action layer designed around confirmation, verification, and auditability | A super App |
| A system that advances work to inspectable / editable / confirmable results | A fully autonomous agent that bypasses user confirmation |

### Core product surfaces

Janus has only three canonical top-level product surfaces. New top-level surfaces should not be introduced without changing the product ontology.

1. **Janus Entry**
   The global and current-work entry point. It is also the anchor for stop, recovery, and refocus. It should expose minimal state and control, not a full task explanation, governance console, or new-task dashboard.

2. **Janus Surface**
   The lifecycle interface for the current entrusted task: understanding, planning, permission and risk, execution, verification, confirmation, stop, and review.

3. **Delegation Center**
   The cross-task governance surface for permissions, memory, audit, and trust. It manages long-term delegation relationships, but it must not replace confirmation for the current task.

### Safety and trust boundaries

The central boundary is simple: capability is not authorization, and automation is not a transfer of user sovereignty.

Hard rules:

- **Capability ≠ Permission**: being able to do something does not mean Janus is allowed to do it.
- **Memory ≠ Permission**: remembering a preference is not long-term authorization.
- **Learning Candidate ≠ Memory**: a learning candidate must be governed before becoming memory.
- **Current context ≠ task boundary**: the current context is not the task boundary; user intent and confirmation define the boundary.
- **Single confirmation ≠ long-term authorization**: one confirmation does not grant durable permission.
- **Model confidence is not verification**: model confidence must not replace auditable evidence or state checks.
- External-impact actions require confirmation: sending, submitting, uploading, publishing, approval, deletion or overwrite, permission changes, payments, and identity actions.
- Audit must record both **done** and **not done**: not only what Janus completed, but also what it did not send, submit, delete, or authorize.

### Representative user scenario

This scenario is representative only. It is not the ontology of Janus, and it should not turn any specific app into the center of the product model.

> The user long-presses Janus Entry and says: “Open WeChat, draft a message to Zhang San, and ask me before sending.”

Janus should open or switch to the target app, search for the contact, create the draft, verify that the draft exists and the message has not been sent, then stop at confirmation. The user can inspect, edit, confirm sending, or stop the task.

### Technical direction

Janus points toward an **Evidence-first Native Runtime**: build verifiable native macOS action first, then use OCR, vision, or simulated input as fallback or supplemental paths.

Key directions:

- Task-centered architecture around Task / Plan / Step / Policy / Permission / Risk / Verification / Audit.
- Native macOS runtime first, rather than starting as a browser extension or single-app plugin.
- Prefer structured signals, system APIs, Accessibility, application state, and verifiable readback.
- Treat OCR, vision, and simulated input as supporting tools, not first principles.
- Use generic primitives plus thin adapters instead of heavy per-app scripts.
- Every step should carry explicit risk, permission, verification, and audit state.

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

The recommended MVP is not a general desktop autonomy system. It is a narrow cross-app drafting and preparation loop:

- Choose 4-6 high-frequency, low-risk tasks that can clearly stop before confirmation.
- Measure interface labor reduction, not just model fluency.
- Measure time to inspectable result.
- Require zero unconfirmed external impact.
- Validate user takeover, stop, and recovery.
- Check whether audit records clearly communicate both done and not done.
