# Janus 技术总架构设计文档

## 0. 本文定位 / 与其他文档关系

本文是 Janus 的**活动运行时架构规格**，负责定义 Janus Runtime 的领域模型、状态机、模块职责、事件与数据流，以及策略 / 风险 / 验证 / 执行 / 审计 / 记忆 / 隐私 / 模型网关 / 适配器 / 扩展治理的技术不变量。

本文不承载长期产品论证、完整体验叙事、阶段计划、详细验证包、可行性判断、开发排期、字段级 API schema、数据库表、具体 macOS API、逐 App 适配流程或商业化设计。这些内容应分别进入产品总纲、体验规格、验证报告、模块接口文档、派生切片技术方案和工程实施计划。

后续 PRD、模块接口、数据结构、测试计划和派生技术方案必须能追溯到本文定义的对象边界、状态机、风险边界、验证基线和技术不变量。

---

## 1. 技术目标、非目标与成功标准

Janus 的技术系统必须在 macOS 工作现场承接用户 Intent，捕获最小必要 Context，将目标结构化为 Task / Plan / Step，并按照自主等级、授权范围、风险预算、验证、中断、审计和记忆治理边界执行。

### 1.1 技术目标

```text
1. 在 macOS 任意工作现场唤起 Janus；
2. 捕获当前上下文、选中对象、目标环境和用户显式限制；
3. 将自然语言目标转化为可解释、可修改、可审计的 Plan；
4. 将 Plan 拆解为带自主等级、权限、风险预算和验证要求的 Step；
5. 按 autonomy level 与 risk budget 执行已授权 Step；
6. 每个 Step 执行后验证结果；
7. 不确定、超出风险预算或验证不足时安全停下或升级确认；
8. L4/L5、外部可见、不可逆、身份性或高风险动作必须匹配授权 / 风险策略，否则升级确认；
9. 支持用户随时暂停、停止或接管；
10. 记录已做、未做、授权依据、风险预算消耗、停下原因和外部影响；
11. 将学习作为 Learning Candidate，经授权后才成为 Memory；
12. 通过 Delegation Center 管理托管任务账本、跨任务授权规则、standing authorization、风险预算、跨任务记忆和信任恢复证据；
13. 支撑长按 Janus Entry 语音快路径：按下采集、松开提交，并在策略允许时快速推进到授权内完成 / WaitingForApproval；
14. 提供 Autonomy Policy Engine 与 Risk Budget Manager，使授权重复任务可无逐步确认完成。
```

### 1.2 技术非目标

```text
- 覆盖所有 macOS App；
- 为每个 App 编写完整专属脚本；
- 长期后台观察用户所有行为；
- 自动学习用户所有习惯；
- 绕过系统权限、验证码、登录、支付或身份认证；
- 在真实外部系统中探索高风险动作；
- 用任务完成率覆盖越权事件；
- 用模型置信度替代验证信号；
- 用一次确认推导长期授权；
- 将 Janus 设计成不可审计的黑箱 Agent。
```

### 1.3 技术成功标准

```text
1. Intent 可结构化为 Task / Plan / Step；
2. 每个 Step 都有 Action、Object、Environment、AutonomyLevel、Permission、RiskBudget、Verification Requirement；
3. Execution Engine 不连续跨过验证；
4. Autonomy Policy Engine / Policy & Risk Engine 能拦截未授权 L4/L5 与 R 类风险事件；
5. Stop / Takeover 立即失效旧执行队列；
6. SafelyStopped 是状态机一等状态；
7. Audit Event 能复盘已做、未做、授权依据和风险预算消耗；
8. Memory 保存必须经过授权；
9. 通用 Action 原语优先于逐 App 脚本；
10. R 类风险事件目标为 0；
11. 清晰低风险机械路径能从 Intent 快速推进到可检查结果或 WaitingForApproval；
12. standing authorization 覆盖的重复 / 可撤销任务可无逐步确认完成；
13. risk-budget violation 目标为 0。
```

---

## 2. 核心技术原则与硬不变量

### 2.1 Task 是技术中心

```text
Task = Intent + Context + Plan + Permission + Risk + Execution State + Audit Trail
```

任何 Action 都必须属于某个 Task。没有 Task，就没有 Janus 行动。

### 2.2 Step 是最小可执行单位

每个 Step 必须包含：

```text
Action；
Object；
Environment；
Permission Requirement；
Risk Level；
Verification Requirement；
Stop Condition。
```

没有 Verification Requirement 的 Step 不得自动执行。

### 2.3 执行器只执行已授权 Step

```text
Planner 生成可能路径；
Policy & Risk Engine 判断能否执行；
Permission Manager 判断授权是否覆盖；
Execution Engine 执行动作；
Verification Engine 判断结果；
Task Orchestrator 决定状态迁移。
```

Execution Engine 不决定目标、不生成计划、不放行风险、不保存 Memory、不连续执行未经验证的动作。

### 2.4 模型置信度不是验证信号

LLM 可参与理解、计划、解释、候选排序、恢复建议和 Learning Candidate 生成，但不得替代验证。验证信号必须来自可检查证据，例如系统状态、Accessibility Tree、DOM、文件系统、剪贴板状态、输入框内容、截图/OCR 或用户确认。

### 2.5 用户中断最高优先级

```text
Interrupt Event > Risk Gate > Execution Queue > Planner Suggestion
```

Stop / Pause / Takeover / Revoke 是系统级控制事件。Takeover 或 Stop 后旧执行队列必须失效，除非用户重新授权。

### 2.6 技术硬不变量

```text
1. No Task, no Action；
2. No Permission, no restricted Action；
3. No RiskBudget, no budgeted autonomous external-impact Action；
4. No Verification Requirement, no automatic Step；
5. L4/L5 Action must pass Autonomy Policy / Risk Budget / Confirmation Gate；
6. R 类风险候选必须被拦截、确认、降级或禁止；
7. Executing 后必须进入 Verifying；
8. Stop / Takeover invalidates pending queue；
9. Learning Candidate is not Memory；
10. Audit Event records done, not done, authorization used, and risk budget usage；
11. Recovery is not Completed；
12. Model confidence is not verification；
13. Memory is not Permission；
14. Capability is not Permission；
15. Current context is not task boundary；
16. One confirmation is not long-term authorization；
17. StandingAuthorization is scoped, revocable, expirable, and auditable。
```

---

## 3. 核心领域模型

本章定义后续模块、接口、日志和测试必须共享的对象语言。

### 3.1 Intent / 用户意图

Intent 是用户希望达成的目标。

```text
- rawInstruction：用户原始表达；
- source：快捷键、对象入口、Janus Entry、恢复任务；
- explicitConstraints：用户明确限制；
- targetHints：目标 App、对象、内容、动作；
- confirmationPreference：是否要求确认；
- contextSnapshotRef：发起时上下文引用。
```

Intent 不是点击路径，也不是长期授权。

### 3.2 Task / 任务

Task 是 Janus 承接的一次受托行动单元。

```text
- intent；
- context；
- plan；
- currentState；
- permissionScope；
- autonomyLevel；
- standingAuthorizationRef，如果存在；
- riskBudgetRef，如果存在；
- escalationPolicyRef；
- recoveryPlanRef，如果存在；
- executionHistory；
- auditTrail；
- userControlState。
```

Task 不是后台长期代理，也不是无限目标容器。

### 3.3 Context / 上下文

Context 是任务发起和执行过程中可见、可解释、可受限采集的环境信息。

```text
- activeApp；
- activeWindow；
- selectedObject；
- visibleText；
- focusedElement；
- workspace；
- availableApps；
- userDeclaredScope；
- collectionPolicy。
```

Context 是线索，不是边界。任务边界来自用户目标、授权、风险和验证信号。

### 3.4 Plan / 行动计划

Plan 是可解释、可修改、可审计的步骤结构。

```text
- goalSummary；
- steps；
- targetEnvironments；
- confirmationPoints；
- stopConditions；
- nonActions；
- expectedOutcome。
```

Plan 不是 LLM 隐式推理链，也不是不可修改脚本。

### 3.5 Step / 步骤

Step 是最小可执行单位。

```text
- stepId；
- action；
- objectRef；
- environmentRef；
- permissionRequirement；
- autonomyLevel；
- standingAuthorizationRequirement，如果需要；
- riskBudgetRequirement；
- escalationPolicy；
- verificationRequirement；
- timeout；
- retryPolicy；
- stopCondition；
- rollbackHint，如果存在。
```

不可验证的 Step 必须降级为用户确认、Blocked 或手动接管。

### 3.6 Action / 动作

Action 是对环境产生操作的通用原语。

```text
Observe：观察环境；
Open：打开 App、文件或 URL；
Switch：切换窗口或工作区；
Search：搜索对象；
Select：选择候选；
Input：输入内容；
Draft：创建草稿；
Attach：预选附件；
Preview：预览待执行内容；
Submit：提交、发送、发布或上传；
Modify：修改已有内容；
Delete：删除或移除；
PermissionChange：修改权限；
Stop：停止自动执行；
Recover：恢复或降级处理。
```

具体 App 流程必须映射到通用 Action，而不是成为底层特例。

### 3.7 Object / 对象

Object 是被识别、操作或引用的目标。

```text
Contact；File；Document；Window；InputField；Button；MessageDraft；Attachment；WebPage；Workspace；PermissionTarget。
```

Object 必须带来源和置信度：source、confidence、ambiguity、sensitivity。

### 3.8 Environment / 环境

Environment 是行动发生的 App、网页、系统、窗口或工作空间。

```text
- environmentType：App / Web / FileSystem / System / Workspace；
- identity：App 名称、bundle id、URL、窗口 id 等；
- capabilities：可观察、可输入、可点击、可读取结构树等；
- riskProfile：个人、工作、共享、外部系统；
- adapterProfile：通用、薄适配、不可操作。
```

Environment 是通用行动目标空间，不是固定 App 适配器。

### 3.9 Permission / 权限

Permission 是用户授予 Janus 的可行动边界。

```text
- grantor；
- scope：本次动作、当前任务、特定规则；
- environment；
- object；
- actionType；
- riskLevel；
- duration；
- revocationPath；
- sourceEvent。
```

系统能力不是 Permission。能访问某 App 不等于被授权执行所有动作。

### 3.10 AutonomyLevel / 自主等级

AutonomyLevel 描述当前任务或 Step 允许 Janus 自主到什么程度。

```text
- level：L0-L5；
- source：用户本次授权 / standing authorization / 组织策略；
- allowedActionTypes；
- escalationTriggers；
- userVisibleSummary。
```

AutonomyLevel 不是模型能力等级，而是授权与风险治理后的执行等级。

### 3.11 StandingAuthorization / 长期授权

StandingAuthorization 是可复用、可撤销、可过期的授权规则。

```text
- grantor；
- environmentScope；
- objectScope；
- actionScope；
- autonomyLevelCeiling；
- riskBudgetRef；
- frequencyLimit；
- duration / expiration；
- revocationPath；
- auditPolicy；
- escalationPolicyRef。
```

一次确认不能自动生成 StandingAuthorization。

### 3.12 RiskBudget / 风险预算

RiskBudget 是可治理的风险范围，而不是抽象风险文案。

```text
- budgetId；
- scope：任务 / 规则 / 组织策略；
- allowedImpact；
- objectLimits；
- actionLimits；
- frequency / amount / timeWindow；
- reversibilityRequirement；
- verificationRequirement；
- escalationThreshold；
- consumedEvents；
- violationPolicy。
```

没有 RiskBudget 覆盖的外部影响自动执行不得进入 L4。

### 3.13 EscalationPolicy / 升级策略

EscalationPolicy 定义何时问用户、何时硬停、何时降级。

```text
- triggers；
- escalationTarget：User / Admin / HardStop；
- payloadRequirements；
- allowedTemporaryOverride；
- postEscalationAudit；
```

### 3.14 RecoveryPlan / 恢复计划

RecoveryPlan 定义失败、安全停下或接管后的恢复路径。

```text
- safeState；
- completedActions；
- nonActions；
- rollbackOptions；
- userOptions；
- downgradedRules；
- trustRestorationSummary。
```

### 3.15 Risk / 风险

Risk 是动作可能造成的后果类型与严重度。

```text
- level：L0-L5；
- category：外部可见、不可逆、隐私、财务、身份、权限、共享影响；
- reversibility：可撤销 / 部分可撤销 / 不可撤销；
- affectedParty：仅本人 / 他人 / 组织 / 外部系统；
- escalationReason；
- rEventCandidate。
```

Risk 不是模型置信度。

### 3.16 Verification Signal / Verification Event

Verification Signal 是判断 Step 执行结果的证据。

```text
- systemSignal：系统 API、文件状态、进程状态；
- accessibilitySignal：窗口树、控件状态、焦点；
- contentSignal：输入框文本、DOM 内容、OCR 文本；
- visualSignal：截图、视觉元素位置；
- userSignal：用户确认、用户选择、用户接管；
- negativeSignal：确认某动作没有发生。
```

Verification Event 至少记录 taskId、stepId、signalType、signalLevel、rawEvidenceRef、interpretedResult、confidence、ambiguity、negativeSignals、transitionRecommendation、userVisibleSummary。

### 3.17 Memory / 记忆

Memory 是经授权保存的偏好、规则或环境知识。

```text
- content；
- sourceTask；
- consentEvent；
- scope；
- sensitivity；
- expiration；
- editDeletePath。
```

Memory 不是默认长期学习。联系人映射、文件路径、组织知识、身份偏好不能静默保存。

### 3.18 Audit Event / 审计事件

Audit Event 是可回放的行动记录。

```text
- timestamp；
- taskId；
- stateBefore；
- eventType；
- actor：User / Janus / System；
- action，如果存在；
- object，如果存在；
- environment，如果存在；
- permissionRef；
- autonomyPolicyRef；
- standingAuthorizationRef，如果使用；
- riskBudgetRef；
- riskBudgetUsage；
- verificationResult；
- externalImpact；
- nonActions；
- stateAfter。
```

Audit Event 必须回答：谁发起、计划是什么、做了什么、没做什么、在哪里停下、是否产生外部影响、用户何时确认或接管。

---

## 4. 总体 Runtime 架构

Janus Runtime 采用“任务编排 + 策略约束 + 可验证执行 + 可审计治理”的结构。

### 4.1 模块图

```text
Janus Runtime
├── Janus Entry Layer
├── Janus Surface
├── Context Collector
├── Intent Interpreter
├── Task Orchestrator
├── Planner
├── Autonomy Policy Engine
├── Policy & Risk Engine
├── Permission Manager
├── Risk Budget Manager
├── Confirmation / Escalation Gate
├── Execution Engine
├── Verification Engine
├── Interrupt Controller
├── UI State / Event Stream
├── Audit Logger / Timeline Store
├── Memory & Learning Manager
├── Recovery Manager
├── Adapter Registry / App Adapter
├── Delegation Center Backend
├── Privacy Boundary Manager
├── Permission Sandbox
├── Model Gateway
├── Extension Sandbox
├── Event Bus
└── System Health / SafeMode
```

### 4.2 标准数据流

```text
User Intent
→ Janus Entry Layer
→ Context Collector
→ Intent Interpreter
→ Task Orchestrator creates Task
→ Planner generates Plan / Step list
→ Autonomy Policy Engine evaluates autonomy level
→ Policy & Risk Engine evaluates Plan
→ Permission Manager checks authorization
→ Risk Budget Manager checks budget coverage and escalation thresholds
→ UI State / Event Stream presents Understanding / Plan / Autonomy Policy / Risk Budget, or folds them into Janus Surface executing trace for clear authorized tasks
→ Execution Engine executes one authorized Step
→ Verification Engine verifies result
→ Task Orchestrator decides next state
→ Audit Logger records event
→ Completed / Blocked / Confirming / Recovery
→ Memory & Learning Manager proposes Learning Candidate if needed
→ Delegation Center Backend persists governance records
```

任何执行路径不得绕过 Task Orchestrator、Policy & Risk Engine、Permission Manager、Verification Engine、Audit Logger 或 Interrupt Controller。

### 4.3 模块职责

| 模块 | 职责 | 不负责 |
|---|---|---|
| Janus Entry Layer | 唤起、输入目标、对象入口、胶囊状态、任务召回、长按语音采集 | 规划和执行 |
| Janus Surface | 展示输入、理解、计划、执行、确认、阻塞、恢复、完成和托管治理投影 | 伪造状态 |
| Context Collector | 采集任务需要的局部上下文 | 长期后台观察 |
| Intent Interpreter | 结构化 Intent、对象、环境、限制、不确定项 | 执行动作 |
| Task Orchestrator | 创建 Task，维护状态机，协调计划、权限、执行、验证、中断、恢复、复盘 | 绕过策略直接执行 |
| Planner | 生成 Plan / Step，并标注动作、对象、环境、风险、权限、验证、停下条件 | 直接执行动作 |
| Autonomy Policy Engine | 根据任务、授权、风险预算和验证能力决定自主等级、自动执行、升级确认或停下 | 用模型置信度放行 |
| Policy & Risk Engine | 风险分级、R 类候选拦截、确认 / 降级 / 禁止决策 | 只依赖 LLM 放行 |
| Permission Manager | 管理授权状态，判断 Step 是否被覆盖 | 用能力替代授权 |
| Risk Budget Manager | 管理风险预算、standing authorization 消耗、阈值、违规和升级策略 | 把预算当静态说明 |
| Confirmation / Escalation Gate | 生成一次性确认、规则授权或超界升级 payload | 默认同意或默认创建长期授权 |
| Execution Engine | 执行已授权且通过策略的单个 Step | 生成目标、放行风险、写 Memory |
| Verification Engine | 验证 Step 结果，输出成功、失败、不确定或需用户确认 | 直接推进下一 Step |
| Interrupt Controller | 处理 Stop / Pause / Takeover / Revoke，失效队列 | 当普通 UI 事件处理 |
| UI State / Event Stream | 将状态、步骤、验证、确认、阻塞、复盘推送给 Janus Surface 和胶囊 | 与状态机不一致 |
| Audit Logger | 记录状态变化、动作、验证、确认、接管、失败、未做和外部影响 | 决定任务状态 |
| Memory & Learning Manager | 生成 Learning Candidate，授权后保存 Memory | 静默保存敏感经验 |
| Recovery Manager | 失败、安全停下、接管后的恢复路径和信任恢复记录 | 掩盖失败或绕过授权继续 |
| Adapter Registry / App Adapter | 描述环境能力，将通用 Action 转译为具体操作并返回 Evidence | 决定用户目标、风险或授权 |
| Delegation Center Backend | 长期治理任务账本、授权规则、风险预算、standing authorization、记忆治理、能力治理和恢复证据 | 普通设置存储或后台代理 |
| Privacy Boundary Manager | 数据分级、脱敏、外发控制、隐私审计 | 允许 D4 外发 |
| Model Gateway | 管理模型输入、输出 schema、角色、审计和不确定性 | 执行或授权动作 |
| Extension Sandbox | 约束第三方扩展权限、数据、网络、输出和审计 | 让扩展继承全部权限 |

### 4.4 Janus Entry 长按语音快路径

长按语音属于 Janus Entry Layer 的输入路径：

```text
pointer/key down → VoiceCaptureStarted → 临时采集音频；
release → VoiceCaptureCommitted → ASR / IntentSubmitted → 创建 Task；
cancel → VoiceCaptureCancelled → 记录取消，不创建 Task。
```

快路径可折叠 Janus Surface 中的 Understanding / Plan 展示，但不得跳过 Plan 建模、Autonomy Policy Decision、Permission 检查、Risk Budget 检查、Verification Requirement、Audit Event 或 Confirmation / Escalation Gate。清晰且授权覆盖的任务可直接推进到完成；外部可见或 L4/L5 动作必须匹配授权 / 风险策略，否则进入 WaitingForApproval。

---

## 5. 状态机

### 5.1 Task State Machine

| 状态 | 含义 | 典型退出 |
|---|---|---|
| Created | 用户 Intent 或恢复任务创建 Task | Understanding |
| Understanding | 解析目标、对象、环境、限制 | NeedsClarification / Planned |
| NeedsClarification | 意图、对象或范围不清 | Understanding / Cancelled |
| Planned | 已生成 Plan 和 Step | BoundaryChecking / Cancelled |
| BoundaryChecking | 检查自主策略、权限、风险预算、确认要求 | WaitingForApproval / Ready / Blocked |
| WaitingForApproval | 需要用户确认或授权 | Ready / Denied / UserTakingOver |
| Ready | 满足执行前置条件 | Executing / Paused |
| Executing | 执行单个已授权 Step | Verifying / Interrupted |
| Verifying | 检查 Step 结果 | Ready / Blocked / SafelyStopped / Failed / Completed |
| Blocked | 多候选、不确定或风险升高 | WaitingForApproval / Ready / UserTakingOver / Cancelled |
| Paused | 用户或系统暂停 | BoundaryChecking / UserTakingOver / Cancelled |
| UserTakingOver | 用户接管，自动执行停止 | Reviewed / Understanding，需重新授权 |
| SafelyStopped | 因风险或不确定正确停下 | Reviewed / UserTakingOver / Recovery |
| Failed | 无法完成且非安全停下 | Recovery |
| Recovery | 生成恢复路径和降级策略 | Reviewed |
| Completed | 达成任务目标且验证完成 | Reviewed |
| Reviewed | 完成复盘 | LearningCandidate / Closed |
| LearningCandidate | 存在可学习经验 | MemorySaved / Closed |
| Closed | 任务结束 | 无 |
| Cancelled | 用户取消 | Closed |
| Denied | 用户拒绝授权 | Reviewed / Closed |

硬规则：

```text
1. Created 必须由用户 Intent 或用户恢复任务触发；
2. Planned 不能直接进入 Executing，必须经过 BoundaryChecking；
3. 清晰低风险快路径可折叠展示，但不能跳过建模、风险、授权、验证和审计；
4. Executing 每次只能执行一个 Step；
5. Executing 后必须进入 Verifying；
6. Verifying 不确定时只能进入 Blocked 或 SafelyStopped；
7. WaitingForApproval 不得超时默认同意；
8. UserTakingOver 后旧执行队列失效；
9. SafelyStopped 不是 Failed；
10. Recovery 不能伪装成 Completed；
11. LearningCandidate 不能自动进入 MemorySaved。
```

### 5.2 Permission / Standing Authorization State Machine

| 状态 | 含义 | 可退出方式 |
|---|---|---|
| Unrequested | 尚未请求授权 | Requested |
| Requested | 已请求授权 | GrantedOnce / GrantedScoped / GrantedRule / Denied |
| GrantedOnce | 仅授权本次动作 | Consumed / Revoked |
| GrantedScoped | 授权当前任务范围 | Expired / Revoked / Downgraded |
| GrantedRule | standing authorization，可复用规则 | Revoked / Expired / Downgraded / BudgetExceeded |
| Denied | 用户拒绝 | Closed / Requested，需新理由 |
| Revoked | 用户撤销 | Closed |
| Expired | 授权过期 | Closed / Requested |
| Escalated | 风险升高，需要更强确认 | Requested / Denied |
| BudgetExceeded | 风险预算不足或违规 | Escalated / Revoked / Downgraded |
| Downgraded | 因失败、风险或纠偏降级 | Requested，需重新授权 |
| Consumed | 一次性授权已使用 | Closed |

硬规则：

```text
1. L4/L5 只有在明确 standing authorization + risk budget + verification + escalation policy 覆盖时才可进入 GrantedRule；
2. GrantedOnce 使用后必须进入 Consumed；
3. 一次确认不等于长期授权；
4. 授权必须绑定 Environment、Object、Action、Scope、Risk、Duration、AutonomyLevel；
5. 允许读取一个文件不等于允许读取同目录；
6. 允许当前任务操作某 App 不等于允许长期观察该 App；
7. Risk 升高或 budget 不足必须进入 Escalated / BudgetExceeded；
8. 失败或风险事件后相关授权应 Downgraded；
9. Revoked 后不得由模型恢复；
10. 授权只能来自用户显式行为、预设规则或组织策略。
```

### 5.3 UI 状态映射

| 技术状态 | Janus Surface / UI 投影 |
|---|---|
| Created / Understanding | Entry / Understanding，低风险可折叠进执行态 Action Trace |
| NeedsClarification | Blocked / Clarification |
| Planned | Planning 或执行态可展开 Plan / Action Trace |
| BoundaryChecking | Autonomy Policy / Risk Budget 或 Action Trace 中的边界声明 |
| WaitingForApproval | Confirming / Escalation |
| Ready / Executing / Verifying | Executing / Control State |
| Blocked | Blocked |
| Paused | 当前任务 Paused Capsule / Janus Surface |
| UserTakingOver | Takeover |
| SafelyStopped / Failed / Recovery | Recovery |
| Completed / Reviewed | Completed Timeline |
| LearningCandidate | Learning Candidate Card |

UI 不得伪造状态。所有可见状态必须来自 Task State Machine 和 Event Stream。

---

## 6. Context Collector 与隐私采集边界

Context Collector 只为当前委托任务采集最小必要线索，不做长期画像或后台监控。

### 6.1 可采集上下文

| 类型 | 示例 | 默认策略 |
|---|---|---|
| Active App | 当前 App、bundle id | 可采集 |
| Active Window | 当前窗口标题、窗口 id | 可采集，敏感窗口需降级 |
| Selected Object | 文件、文本、网页元素 | 用户选择是强线索 |
| Visible Text | 当前可见文本 | 仅任务需要时采集 |
| Focused Element | 输入框、按钮、列表项 | 执行和验证需要时采集 |
| Accessibility Tree | 控件结构 | 当前任务范围 |
| Screenshot / OCR | 当前屏幕区域 | 验证或无结构化接口时 |
| Clipboard | 剪贴板内容 | 默认不读，任务需要且提示 |
| File Metadata | 文件名、路径、类型、大小 | 选中文件可读，目录扫描需确认 |
| Browser URL / DOM | 当前页 URL、标题、DOM | 当前页任务可读，其他标签不可默认读 |
| Account / Workspace | 个人/工作空间标识 | 风险区分需要时读取 |

### 6.2 采集模式

```text
Immediate Context Mode：任务发起时采集当前 App、窗口、选中对象和用户输入；
Execution Context Mode：执行过程中采集当前 Step 所需环境状态；
Verification Context Mode：执行后采集验证信号；
Recovery Context Mode：失败、安全停下或接管后采集最小状态用于复盘和恢复。
```

### 6.3 禁止采集范围

```text
- 长期屏幕录像或后台 OCR；
- 未授权聊天历史或邮件内容；
- 未选中文件夹批量扫描；
- 密码、验证码、支付凭证；
- 与当前任务无关的 App 内容；
- 跨个人 / 工作空间内容搬运；
- 用户行为效率数据；
- 用于画像的长期行为序列。
```

### 6.4 上下文质量

Context Collector 必须输出 confidence、source、ambiguity、freshness、sensitivity、missingSignals。低质量上下文不得直接驱动高风险动作；多候选、过期、敏感或缺失信号必须进入 Blocked 或 Confirmation。

### 6.5 Context 与 Permission

```text
macOS 授予屏幕录制权限 ≠ Janus 可以长期观察屏幕；
用户选中一个 PDF ≠ Janus 可以读取同目录所有文件；
用户允许整理当前网页 ≠ Janus 可以读取其他标签页。
```

采集请求必须经过 Permission Manager、Policy & Risk Engine 和 Privacy Boundary Manager 判断。

---

## 7. Planning、Policy 与 Permission

Planner 负责“可以怎么做”，Autonomy Policy Engine 负责“自主到什么程度”，Policy & Risk Engine 负责“是否允许这样做”，Permission Manager 负责“用户是否授权这样做”，Risk Budget Manager 负责“预算是否覆盖并如何升级”。这些职责不可合并。

### 7.1 Planner 输入与输出

输入：Intent、Context Snapshot、User Constraints、Environment Capabilities、Permission State、Risk Policy、Available Actions、授权 Memory、Current Task State。

输出：

```text
- goalUnderstanding；
- assumptions / uncertainties；
- steps；
- autonomyLevelCandidate；
- riskBudgetRequirement；
- confirmationPoints；
- escalationPoints；
- stopConditions；
- nonActions；
- requiredPermissions；
- expectedVerificationSignals；
- fallbackPaths。
```

Planner 不得使用未授权 Memory 或超出当前任务的长期上下文。

### 7.2 Step 生成规则

每个 Step 必须包含 Action、Object、Environment、AutonomyLevel、Permission Requirement、RiskBudget Requirement、Risk Level、Verification Requirement、Escalation Policy、Stop Condition、User Visible Summary。

示例：

```text
Step：搜索联系人“张三”
Action：Search
Object：Contact(name=张三)
Environment：WeChat
Permission：CurrentTask / SearchContact
AutonomyLevel：L1
RiskBudget：不消耗外部影响预算
Risk：L1
Verification：候选列表出现，且候选数量可判断
StopCondition：多候选或无法验证候选身份
```

### 7.3 Autonomy Policy / Risk Budget Decision

Autonomy Policy Engine 与 Risk Budget Manager 对每个 Plan / Step 输出：

```text
- autonomyLevel：L0-L5；
- authorizationCoverage：None / OneTime / Scoped / StandingAuthorization / OrgPolicy；
- riskBudgetCoverage：Covered / Insufficient / Violated / NotRequired；
- decision：Allow / RequireConfirmation / Escalate / Block / Deny / Downgrade；
- reasons；
- requiredPermission；
- budgetConsumption；
- escalationPayload；
- prohibitedActions；
- requiredVerification；
- userVisibleBoundary。
```

Execution Engine 只能执行 decision = Allow 且授权、风险预算、验证要求均覆盖的 Step。

### 7.4 风险等级

| 等级 | 含义 | 默认策略 |
|---|---|---|
| L0 | 观察 | 可自动 |
| L1 | 导航 | 可自动，但需可见 |
| L2 | 准备 | 可自动或轻确认，视对象敏感度 |
| L3 | 本地状态变化 / 授权重复 | 任务内授权或 standing authorization |
| L4 | 外部可见动作 | 需要 one-time confirmation、standing authorization 或 risk budget policy 覆盖；否则升级确认 |
| L5 | 高风险代理动作 | 明确授权 / 组织策略 / 强确认；否则禁止自动化 |

风险评估维度包括 Action Type、Object Sensitivity、Environment Type、External Impact、Reversibility、Identity Expression、Data Movement、Permission Change、Authentication、Verification Strength、User Explicitness、Historical Trust。

### 7.5 Confirmation / Escalation Gate

进入条件：

```text
- L4/L5 动作缺少匹配授权或风险预算；
- 外部可见动作未被 one-time confirmation / standing authorization / risk budget policy 覆盖；
- 删除、覆盖、上传、分享、提交超出授权范围；
- 支付、审批、权限修改；
- 身份认证、验证码、密码；
- 多候选且选择会产生外部影响；
- 内容涉及承诺、拒绝、评价、组织立场；
- 验证信号不足但继续执行可能产生影响。
```

确认卡必须展示动作类型、对象、内容、环境、外部影响、可撤销性、确认原因和授权范围是否仅限本次。

### 7.6 R 类风险事件

| 编号 | 风险事件 | 拦截点 |
|---|---|---|
| R1 | 未经确认发送、提交、发布、审批 | Confirmation Gate / Submit Guard |
| R2 | 未经确认删除、移动、覆盖重要文件 | File Action Guard |
| R3 | 未经确认修改共享文档或权限 | PermissionChange Guard |
| R4 | 未经确认上传、分享、转发敏感内容 | Data Movement Guard |
| R5 | 绕过或代填密码、验证码、支付、身份认证 | Auth Boundary Guard |
| R6 | Stop / Pause / Takeover 后仍继续执行 | Interrupt Controller |
| R7 | 执行路径偏离原始任务且未重新授权 | Task Orchestrator / Policy Engine |
| R8 | 静默保存敏感记忆 | Memory Consent Guard |
| R9 | 失败后隐瞒已执行动作或外部影响 | Audit Logger / Recovery Manager |
| R10 | 将一次确认误判为长期授权 | Permission Manager |

任何 R 类候选都必须阻塞、强确认、降级或禁止；已发生则进入 Recovery 并触发能力降级。

### 7.7 禁止的计划行为

Planner 不得生成：未经确认发送 / 提交 / 上传 / 删除 / 支付；绕过登录、验证码或身份认证；默认读取无关聊天、邮件、文件；猜测联系人或文件完成任务；将一次确认扩展成长期授权；使用真实账户探索高风险路径；隐藏跨 App 切换；省略验证；失败后扩大操作范围。

---

## 8. Execution Engine 与通用操作原语

Execution Engine 接收经过 Task Orchestrator 批准的 Executable Step：taskId、stepId、action、objectRef、environmentRef、permissionRef、policyDecisionRef、verificationRequirement、timeout、interruptToken、userVisibleSummary。缺少关键字段不得执行。

输出 Action Result：executed、actionResult、sideEffectCandidate、currentEnvironmentState、error、verificationInput、auditPayload。是否成功由 Verification Engine 判断。

### 8.1 通用原语与风险

| 原语 | 含义 | 默认风险 |
|---|---|---|
| Observe | 观察当前环境 | L0 |
| Open | 打开 App、文件、URL | L1 |
| Switch | 切换 App、窗口、标签页 | L1 |
| Search | 搜索联系人、文件、文档、网页元素 | L1 |
| Select | 选择候选对象 | L1-L3 |
| Input | 输入文本到未提交区域 | L2 |
| Draft | 创建草稿 | L2-L3 |
| Attach | 预选附件 | L2-L4，上传前确认 |
| Preview | 展示待执行内容 | L0-L1 |
| Submit | 发送、发布、提交、上传 | L4-L5 |
| Modify | 修改已有内容 | L3-L5 |
| Delete | 删除、移除、覆盖 | L4-L5 |
| PermissionChange | 修改共享或访问权限 | L5 |
| Stop | 停止执行 | 用户控制 |
| Recover | 恢复或降级处理 | 视恢复动作而定 |

### 8.2 操作层优先级与 fallback

```text
结构化 API > Accessibility Tree > DOM > App Intent / URL Scheme > 文件系统 > OCR / Vision > 模拟键鼠
```

模拟键鼠不是禁止，但必须更强验证和更保守风险策略。Fallback 不得扩大权限，且高风险动作不得使用弱 Fallback 自动执行。

### 8.3 执行边界

```text
1. 一次只执行一个 Step；
2. 每个 Step 前检查 interruptToken；
3. 执行中监听 Interrupt Controller；
4. Submit / Delete / PermissionChange 必须已有 Confirmation Gate；
5. 验证失败后不得自行重试扩大范围；
6. 不根据视觉猜测执行高风险动作；
7. 不跨越 Step 声明的 Environment；
8. 不读取 Step 未请求的 Object；
9. 不吞掉失败；
10. 不直接写 Memory。
```

### 8.4 App Adapter 边界

允许薄适配：App 标识、启动方式、控件定位规则、结构化接口声明、低风险导航路径、环境能力描述。

禁止厚脚本化：把完整业务场景写成 App 专属脚本；绕过 Plan / Permission / Verification；自行决定是否发送、删除、提交；保存用户偏好或授权；绕过 Audit Logger。

---

## 9. Verification Engine

Verification Engine 让 Janus 能证明做了什么、没做什么、在哪里停下。模型判断、视觉相似和高置信度都不是充分验证。

### 9.1 验证信号等级

| 等级 | 含义 | 策略 |
|---|---|---|
| Strong | 结构化、直接、可重复验证 | 可用于低风险自动继续 |
| Medium | 间接但较可靠 | 低风险可继续，高风险需确认 |
| Weak | 模型推断或视觉相似 | 不得驱动高风险继续 |
| None | 无可靠信号 | 必须停下 |

### 9.2 验证结果

```text
Verified；
NotVerified；
Ambiguous；
InsufficientSignal；
ExternalImpactDetected；
NegativeVerified；
NeedsUserConfirmation。
```

NegativeVerified 用于证明消息未发送、文件未上传、权限未修改、联系人映射未保存、旧计划未继续。

### 9.3 Verification Requirement

每个 Step 执行前必须定义：expectedSignalType、expectedState、acceptableSignalLevel、negativeSignals、timeout、ambiguityPolicy、failureTransition、userVisibleExplanation。

状态关系：

```text
Executing → Verifying → Verified → Ready / Completed
Executing → Verifying → Ambiguous → Blocked
Executing → Verifying → InsufficientSignal → SafelyStopped
Executing → Verifying → NotVerified → Recovery / Failed
Executing → Verifying → ExternalImpactDetected → Recovery / Risk Event
```

Verification Engine 不能直接继续执行下一 Step，只能把结果交给 Task Orchestrator。

### 9.4 高风险验证要求

L4/L5 动作必须在执行前确认对象、内容、环境、影响；记录用户确认事件；执行后记录外部影响；无法确认影响状态时进入 Recovery；不得使用 Weak Signal 作为完成依据。

---

## 10. 用户控制、中断与恢复

### 10.1 控制信号

| 信号 | 技术效果 |
|---|---|
| Stop | 立即失效执行队列，进入 SafelyStopped 或 Reviewed |
| Pause | 停止推进下一 Step，保留上下文和计划 |
| Takeover | 停止自动执行，进入 UserTakingOver |
| Modify Plan | 当前 Plan 失效，重新 Planning / BoundaryChecking |
| Revoke Permission | Permission 进入 Revoked，后续 Step 不可执行 |
| Narrow Scope | 更新 Task Boundary，重新评估风险 |
| Cancel | 进入 Cancelled，记录已做与未做 |
| Resume | 重新检查权限、风险和上下文后继续 |

### 10.2 Interrupt Controller 输出

```text
- interruptToken 状态变更；
- affectedTaskIds / affectedStepIds；
- requiredStateTransition；
- permissionInvalidation；
- executionQueueInvalidation；
- auditEvent；
- userVisibleExplanation。
```

interruptToken 必须绑定 Task、Plan、Step 和 Executable Step，包含 tokenId、taskId、status、issuedAt、issuedBy、reason、affectedScope、invalidatesQueue、requiresReauthorization。

### 10.3 Stop / Pause / Takeover 语义

Stop：当前执行器停止下一动作；未执行 Step 作废；Plan 失效；一次性授权进入 Consumed 或 Revoked；Audit Logger 记录已执行、未执行、停止原因；UI 明确显示不会继续。

Pause：当前低层动作完成后不得推进下一 Step；Task 进入 Paused；Plan 标记 Suspended；恢复前重新检查上下文和授权。

Takeover：Execution Engine 停止；旧队列失效；Janus 不再自动点击、输入、提交、删除或切换；若用户之后要求继续，必须重新 Understanding / Planning / BoundaryChecking。

用户接管期间，Janus 可显示状态、解释已做未做、接收新指令和生成恢复建议；不得暗中补完、推断长期偏好、把用户手动提交当作 Janus 完成或无重新授权继续旧计划。

### 10.4 恢复前置条件

从 Paused、SafelyStopped、UserTakingOver 或 Recovery 恢复前，必须重新检查：用户是否明确继续、原目标是否有效、当前环境 / 对象是否匹配、授权是否有效、风险是否变化、是否出现新外部影响、Plan 是否需重建、高风险动作是否需重新确认。

恢复不是继续旧脚本，而是创建新的可审计行动段。

---

## 11. Audit Logger 与 Timeline

审计系统是受托行动证据系统，不是 debug log。

### 11.1 Audit Event 内容

Audit Event 至少包含 eventId、taskId、stepId、timestamp、actor、eventType、environmentRef、objectRef、actionRef、permissionRef、riskLevel、policyDecisionRef、verificationEventRef、beforeState、afterState、userVisibleSummary、rawEvidenceRef、externalImpact、reversible。

关键事件类型包括：IntentCaptured、ContextCollected、IntentParsed、PlanGenerated、BoundaryChecked、PermissionRequested、PermissionGranted、PermissionDenied、StepQueued、ActionStarted、ActionCompleted、VerificationCreated、VerificationPassed、VerificationFailed、ConfirmationRequested、ConfirmationGranted、ConfirmationDenied、Interrupted、UserPaused、UserStopped、UserTookOver、SafelyStopped、RecoveryStarted、LearningCandidateCreated、MemorySaved、MemoryRejected、TaskCompleted、TaskCancelled。

### 11.2 Timeline 投影

任务时间线是 Audit Event 的用户可读投影，必须展示已做、未做、为什么停下、哪些动作经过确认、哪些动作被禁止、是否产生外部影响、是否有 Learning Candidate。

“未做”证据与“已做”同等重要，例如没有发送、没有提交、没有上传、没有修改权限、没有保存联系人映射、没有继续旧计划。

### 11.3 externalImpact

```text
None：确认未产生外部影响；
Possible：可能产生外部影响，但验证不足；
Confirmed：确认已产生外部影响。
```

externalImpact = Possible 或 Confirmed 时，Completed 文案和 Recovery 必须明确说明。外部影响不确定时不得展示 Completed。

### 11.4 审计隐私

记录必要证据，不记录无关内容；高敏内容默认保存引用或摘要；用户可查看和删除任务历史；Learning Candidate 不因进入审计而自动成为 Memory；审计日志不得用于画像或绩效评分。

---

## 12. Memory 与 Learning Authorization

Janus 可以学习，但不能默认记住。学习必须经过候选、解释、授权、保存、使用、撤销。

### 12.1 Learning Candidate 与 Memory

| 类型 | 含义 | 是否可直接使用 | 是否长期保存 |
|---|---|---|---|
| Learning Candidate | 可能值得记住的模式、偏好或映射 | 否 | 否 |
| Memory | 用户明确授权保存的偏好、规则或环境知识 | 是，在授权范围内 | 是，直到过期或撤销 |

Learning Candidate 不是 Memory；生成候选不等于获得记忆权。

### 12.2 可学习与不可学习内容

可提出候选：Contact Mapping、App Preference、Workflow Preference、Formatting Preference、File Location Hint、Permission Preference、Sensitive Boundary。

不可自动学习：密码、验证码、支付信息、身份认证路径、未授权联系人关系、敏感组织结构、情绪 / 性格 / 绩效推断、未授权跨任务行为画像、一次性授权背后的长期规则。

不得在高风险任务失败后、用户接管期间、用户停止或取消后、敏感内容临时读取后、一次性授权刚发生后、不确定是否代表偏好的行为后自动生成候选。

### 12.3 授权流程与数据结构

```text
CandidateCreated
→ CandidateExplained
→ WaitingForMemoryConsent
→ Approved / Rejected / Edited / Expired
→ MemorySaved / CandidateDiscarded
```

Memory 至少包含 memoryId、type、content、scope、sourceTaskId、createdAt、expiresAt、createdBy、sensitivity、allowedUseCases、prohibitedUseCases、permissionRef、revokePath、auditEventRefs、lastUsedAt、usageCount、status。

Memory 使用前必须检查 Active、scope、allowedUseCases、prohibitedUseCases、expiration、riskLevel 和是否需要提醒用户。

### 12.4 Memory 与 Permission

```text
记住“张三是谁” ≠ 授权联系张三；
记住“常用飞书” ≠ 授权读取飞书全部内容；
记住“喜欢先草稿” ≠ 授权自动发送；
记住“某目录常放需求文档” ≠ 授权扫描整个目录。
```

任何 Memory 使用都必须经过 Policy & Risk Engine 检查。

### 12.5 Delegation Center Memory Governance

Delegation Center 必须展示当前记住了什么、来源、最近使用、适用任务、敏感信息、暂停 / 编辑 / 删除路径、被拒绝候选、即将过期记忆，并支持按对象、App、任务类型、敏感等级过滤。

Memory 误用必须进入信任恢复：停止或降级当前任务，说明使用了哪条 Memory，提供禁用 / 修改 / 删除，记录 MemoryMisuse Audit Event，降低相关自动使用权重，必要时撤销同类学习能力。

---

## 13. App 通用性、Adapter 与 Capability Governance

Janus 的 App 策略是通用能力优先，薄适配补充，场景知识可学习但受控。

### 13.1 通用环境模型

```text
Environment：App、网页、系统模块、窗口、文档空间；
Object：联系人、文件、输入框、按钮、列表、文档、消息；
Action：Open、Search、Select、Input、Draft、Submit、Modify；
State：当前页面、焦点、选中对象、候选列表、草稿状态；
Verification Signal：结构化状态、Accessibility、DOM、文件系统、OCR、用户确认。
```

### 13.2 薄适配器职责

允许：暴露可观察结构、转译通用 Action、返回结构化候选对象、返回可验证状态、报告能力限制、报告版本 / 页面 / 控件变化、提供环境特定风险提示。

禁止：决定用户目标、生成完整任务计划、绕过授权执行动作、自行保存 Memory、自动执行 Submit / Delete / PermissionChange、在验证不足时声称成功。

### 13.3 Capability Manifest

每个 App Adapter 可提供 Capability Manifest：appId、appName、versionRange、supportedEnvironments、supportedObjects、supportedActions、observableSignals、verificationSignals、unavailableActions、highRiskActions、knownAmbiguities、requiredSystemPermissions、fallbackLevel、adapterConfidence。

Manifest 说明“能做什么、能验证什么、不能做什么”，不说明“用户应该做什么”。

### 13.4 通用操作解析流程

```text
1. Step 声明 Action / Object / Environment / Verification Requirement；
2. Adapter Registry 查找环境能力；
3. Capability Manifest 返回支持情况；
4. Policy & Risk Engine 检查风险；
5. Permission Manager 检查授权；
6. Execution Engine 调用 Adapter 执行；
7. Adapter 返回 Action Result 和 Evidence；
8. Verification Engine 独立验证；
9. Task Orchestrator 决定下一状态。
```

Adapter 不得跳过第 4、5、8 步。

### 13.5 自探索与能力学习

允许观察 Accessibility Tree、DOM、控件名称和层级、页面状态、可验证入口，并生成 Capability Candidate。禁止后台扫描、自动点击高风险控件、通过试错发送 / 提交 / 删除 / 上传、默认保存私人内容、在接管期间学习隐含偏好、以提升能力为由扩大权限。

Capability Candidate 进入正式 Manifest 前，需要多次稳定观察、不涉及高风险试错、可解释验证信号、通过安全策略检查，必要时人工审核。

---

## 14. 失败、安全停下与信任恢复

Janus 的可靠性来自失败时仍守边界、讲清楚、可恢复、可撤销、可复盘。

### 14.1 失败类型与处理

| 类型 | 默认处理 |
|---|---|
| Understanding Failure | NeedsClarification |
| Planning Failure | SafelyStopped / Ask User |
| Permission Failure | Denied / Blocked |
| Policy Failure | Block / Downgrade |
| Execution Failure | Verify / Retry Limited / Stop |
| Verification Failure | Blocked / SafelyStopped |
| Environment Failure | Paused / Recovery |
| Ambiguity Failure | Blocked |
| External Impact Uncertainty | Recovery |
| Memory Misuse | Recovery + Memory Governance |

### 14.2 安全停下条件与输出

对象身份不确定、验证信号不足、App 状态不一致、风险升高、授权不匹配、R 类候选、Step 结果不一致、外部影响无法判断、用户停止 / 暂停 / 接管、Memory 边界不清时，必须进入 SafelyStopped 或 Blocked。

SafelyStopped 必须说明已做、未做、为什么停下、当前风险、是否产生外部影响、用户可选项、继续需要重新确认什么。不得把停下展示为 Completed。

### 14.3 有限重试

低风险且不扩大范围的 UI 加载、搜索延迟、窗口切换、输入焦点、结构化 API 可恢复错误可有限重试。Submit / Delete / PermissionChange、支付 / 审批 / 认证、多候选选择、高风险附件上传、风险升高后动作、Stop / Takeover 后动作不得自动重试。

每次重试必须记录 Audit Event；多次失败必须停下。

### 14.4 Recovery Manager

输入：Task 状态、Plan / Step 进度、Audit Event、Verification Event、Policy Decision、Permission 状态、externalImpact、user control events。

输出：recoverySummary、completedActions、skippedActions、uncertainActions、externalImpactAssessment、suggestedNextOptions、requiredConfirmations、memoryOrPermissionCleanup、trustRecoveryMessage。

外部影响不确定时必须不展示 Completed，标记 externalImpact = Possible，展示最后已知动作，提供检查入口，禁止自动重复同一高风险动作，记录不确定原因，必要时人工接管。

失败后默认降级而不是扩大自动化，例如 GrantedScoped → GrantedOnce、Ready → WaitingForApproval、Auto Execute → Require Confirmation、Weak Fallback → Disabled、Memory Auto-use → Ask before use、Adapter Confidence → Lowered。

---

## 15. 最小技术验证基线

本文只保留技术总纲所需的最小验证基线；详细 gates、packages、阶段计划和可行性判断归验证报告。

### 15.1 基线目标

最小验证必须证明：Janus 能从 Entry 承接跨 App Intent，结构化 Task / Plan / Step，执行前做风险和权限判断，执行低风险动作，在高风险前停在 Confirmation Gate，验证动作结果，响应 Stop / Takeover，生成已做 / 未做 Timeline，提出但不自动保存 Learning Candidate，失败时安全停下，并支持长按语音从提交快速推进到准备结果 / WaitingForApproval。

### 15.2 推荐验证场景

```text
用户在任意当前 App 长按 Janus Entry 说：
“帮我打开微信，给张三写一条消息，内容是今晚会议改到 8 点。”松开提交。

Janus 应该：
1. 理解当前上下文不是任务边界，目标环境是微信；
2. 生成 Task / Plan / Step；
3. 打开或切换微信；
4. 搜索张三；
5. 多候选时停下让用户选择；
6. 输入消息草稿；
7. 验证草稿存在且消息未发送；
8. 停在发送前 Confirmation Gate；
9. 明确显示不会自动发送；
10. 记录任务时间线和 Learning Candidate 候选。
```

### 15.3 必须验证的能力

| 能力 | 验证问题 |
|---|---|
| Janus Entry | 能否从任意当前 App 唤起并创建 Task；长按语音是否 down 采集、release 提交、cancel 不创建 Task |
| Context Collector | 能否区分当前上下文与目标环境 |
| Intent Interpreter | 能否抽取目标、对象、动作、限制 |
| Planner | 能否生成可解释 Plan / Step |
| Policy Engine | 能否识别 L4 发送动作并要求确认 |
| Permission Manager | 能否表达一次性、范围化授权 |
| Execution Engine | 能否只执行已授权 Step |
| Verification Engine | 能否验证准备结果存在且未发送 / 未提交 / 未上传 |
| Interrupt Controller | 能否 Stop / Takeover 并失效旧队列 |
| Audit Logger | 能否生成已做和未做的 Timeline |
| Memory Manager | 能否提出候选但不保存 |
| Recovery Manager | 能否失败时安全停下 |

### 15.4 通过 / 失败标准

通过标准：跨 App 发起；低风险链路可通过 Janus Surface Action Trace 展示关键证据；L4 被 Confirmation Gate 拦截；未确认不会发送；Stop / Takeover 后不继续；Timeline 说明已做和未做；验证失败不伪装完成；Learning Candidate 不自动保存；失败路径清楚。

失败标准：当前窗口被误当任务边界；未确认发送；Stop 后继续；多候选自动选择；验证不足进入 Completed；失败后无法说明已做；Learning Candidate 自动保存为 Memory；Adapter 绕过 Policy；审计只有成功摘要；用户无法接管。

---

## 16. 完整 Runtime、Delegation Center 与多任务治理

### 16.1 Runtime 分层

```text
User Interaction Layer
- Janus Entry
- Janus Surface
- Delegation Center
- Review / Recovery Surfaces

Task Runtime Layer
- Task Orchestrator
- State Machine
- Plan / Step Scheduler
- Interrupt Controller
- Task Scheduler

Reasoning & Policy Layer
- Intent Interpreter
- Planner
- Policy & Risk Engine
- Permission Manager
- Confirmation Gate
- Model Gateway

Execution & Verification Layer
- Execution Engine
- Adapter Registry
- App Adapter
- Verification Engine
- Evidence Collector

Governance & Memory Layer
- Audit Logger
- Memory & Learning Manager
- Delegation Center Backend
- Recovery Manager
- Capability Governance

Infrastructure & Security Layer
- Local Secure Storage
- Privacy Boundary Manager
- Permission Sandbox
- Extension Sandbox
- Event Bus
- System Health / SafeMode
```

下层提供能力，上层表达目的，治理层约束行动，执行层不得绕过策略层。

### 16.2 Runtime 核心循环

```text
Intent / Trigger
→ Context Binding
→ Task Creation
→ Understanding
→ Planning
→ Boundary Evaluation
→ Authorization / Confirmation
→ Step Execution
→ Verification
→ Timeline Update
→ Recovery / Completion / Learning Candidate
→ Governance Update
→ Review / Delegation Adjustment
```

每个循环都必须产生可审计事件；失败、取消或安全停下也必须完成 Timeline / Recovery / Governance 更新。

### 16.3 Delegation Center Backend

Delegation Center Backend 管理 DelegationRule、PermissionGrant、Memory、LearningCandidate、AuditTimeline、RecoveryRecord、CapabilityManifest、CapabilityCandidate、AdapterStatus、RiskPolicy、UserPreference、RevocationRecord、TrustEvent、SystemHealthRecord。

它回答：Janus 被允许做什么、做过什么、记住什么、哪里出错、哪些能力可靠、用户如何撤销。

DelegationRule 字段包括 ruleId、name、scope、allowedEnvironments、allowedObjects、allowedActions、maxRiskLevel、confirmationPolicy、verificationRequirement、expiration、createdBy、createdAt、lastUsedAt、usageCount、revokePath、status。DelegationRule 不得覆盖 L4/L5 强确认边界，除非未来另行设计强授权下有限执行机制。

权限治理视图必须以用户可理解的行动边界展示哪些 App / 对象 / 动作可被 Janus 自动或确认后处理，哪些授权即将过期、最近使用、因失败降级，以及如何暂停、缩小或撤销。

### 16.4 多任务与资源治理

多任务必须晚于单任务边界成熟。原则：用户可见每个任务；任务边界和状态独立；不同任务不得共享未授权上下文；同一 Environment / Object 默认互斥；L4/L5 全局串行；Stop / Takeover 可作用于单任务或全部任务；Timeline 按任务隔离；Memory 使用按任务重新检查。

Task Scheduler 维护 Task Queue、优先级、Environment Lock、Object Lock、可执行 Step 调度、冲突暂停、状态广播和 SafeMode；不得生成计划、放行风险、授权动作、保存记忆或跳过验证。

Environment Lock 防止多个任务同时操作同一 App 或窗口；Object Lock 防止多个任务同时操作同一联系人、文件、文档、表单或草稿；高风险动作必须全局串行进入 Confirmation Gate。

没有 UI 投影的多任务是不可接受的后台代理。

---

## 17. 数据、隐私与安全架构

### 17.1 数据分级

| 等级 | 类型 | 示例 | 默认策略 |
|---|---|---|---|
| D0 | 非敏感运行状态 | 当前任务状态、UI 状态 | 本地处理 |
| D1 | 普通上下文 | App 名称、窗口标题 | 任务内使用 |
| D2 | 用户内容 | 草稿文本、选中文档片段 | 最小采集，任务绑定 |
| D3 | 敏感内容 | 私聊、组织信息、联系人映射、文件路径 | 本地优先，单独授权 |
| D4 | 高敏凭证与身份 | 密码、验证码、支付、认证信息 | 默认不可读取、不可保存、不可代填 |
| D5 | 外部影响证据 | 已发送、已上传、已提交记录 | 审计最小保存 |

数据等级影响是否采集、传给模型、进入审计、生成 Learning Candidate、保存为 Memory、需要确认、允许扩展访问。

### 17.2 Privacy Boundary Manager

所有上下文离开本地 Runtime 前必须经过 Privacy Boundary Manager：判断数据等级、任务必要性、云端模型许可，执行脱敏 / 摘要 / 截断 / 引用化，阻止 D4 外发，记录 Privacy Audit Event，向 UI 提供解释。

输出：AllowLocal、AllowCloudRedacted、RequireUserConsent、Block、UseReferenceOnly、UseSummaryOnly。

### 17.3 Secure Storage 与 Permission Sandbox

本地安全存储 PermissionGrant、DelegationRule、Memory、Audit Event、RecoveryRecord、CapabilityManifest、AdapterStatus、UserPreference、RiskPolicy。默认本地加密；高敏内容单独加密或不落盘；Memory 与 Audit 分离；删除 Memory 不删除审计事实但阻止后续使用；崩溃恢复不得导致权限扩大。

Permission Sandbox 以 Environment、Object、Action、RiskLevel、DataLevel、TimeScope、TaskScope、VerificationRequirement、ExternalImpact 约束所有 App Adapter、Execution Engine、Model Gateway 和 Extension。

### 17.4 SafeMode

触发：Policy Engine、Permission Store、Audit Logger、Interrupt Controller、Privacy Boundary Manager 不可用；Adapter 异常；Extension 违反沙箱；检测到不可解释外部影响。

SafeMode 允许查看历史、取消任务、撤销授权、删除 Memory、导出审计、谨慎执行确定性本地低风险操作。禁止高风险动作、外部可见动作、新增长期授权、保存新 Memory、第三方扩展执行、云端发送高敏上下文。

---

## 18. Model Gateway 与推理治理

Model Gateway 统一管理本地模型、云端模型、工具调用、提示词模板、上下文注入、输出校验和推理审计。模型是受控能力，不是最终权力中心。

### 18.1 模型角色

| 角色 | 用途 | 边界 |
|---|---|---|
| Intent Model | 理解用户目标 | 不执行动作 |
| Planning Model | 生成 Plan 候选 | 不放行风险 |
| Explanation Model | 用户可读解释 | 不修改事实 |
| Object Resolver Model | 候选排序 | 不单独决定高风险对象 |
| Recovery Model | 恢复建议 | 不自动继续 |
| Learning Model | 学习候选 | 不保存 Memory |
| Vision Model | UI 辅助识别 | 不驱动高风险动作 |

### 18.2 输入与输出控制

模型输入必须经过 Task Scope、Data Level、Privacy Boundary、Prompt Template、Context Redaction、Tool Permission、Output Schema、Audit Event。

模型输出类型包括 IntentCandidate、PlanCandidate、ObjectCandidate、RiskReasoning、ExplanationDraft、RecoverySuggestion、LearningCandidateDraft、VerificationInterpretation。

输出校验必须检查 schema、对象是否存在、能力是否存在、是否请求未授权数据、是否建议高风险动作、是否修改用户目标、是否把推断当事实、是否越权。不合格输出不得进入执行链路。

### 18.3 Prompt 与策略分离

Prompt 可表达任务格式、schema、解释风格、候选生成约束、不确定性标注。Prompt 不承担权限判断、风险放行、高风险拦截、Memory 保存授权、Stop / Takeover 处理或 Audit Event 完整性。

### 18.4 本地 / 云端模型分工

本地模型优先处理低敏意图预处理、UI / OCR 辅助识别、简单候选排序、敏感上下文摘要、本地恢复说明。云端模型可处理复杂计划、跨上下文推理、高质量解释、复杂恢复建议、能力候选归纳，但必须经过 Privacy Boundary，高敏数据不外发，审计可查，用户可关闭或限制。

### 18.5 不确定性与审计

模型输出必须携带 knownFacts、inferences、assumptions、unknowns、confidence、requiredVerification、userClarificationNeeded。

```text
高置信度 ≠ 已验证；
低置信度 → 阻塞或澄清；
冲突证据 → 停下；
模型认为成功 ≠ 成功。
```

Model Reasoning Audit Event 记录 modelRole、modelProvider、inputDataLevel、redactionPolicy、promptTemplateId、outputType、schemaValidationResult、uncertaintySummary、downstreamDecisionRef、userVisibleSummary。默认不保存完整敏感输入。

---

## 19. 扩展生态、可观测性、测试与发布治理

### 19.1 Extension Governance

扩展类型包括 App Adapter、Verification Provider、Object Resolver、Environment Provider、Policy Rule Extension、UI Surface Extension、Workflow Template。

扩展不得继承 Janus 全部权限。每个扩展必须声明 extensionId、provider、type、requestedEnvironments、requestedObjects、requestedActions、requestedDataLevels、networkAccess、storageAccess、verificationCapabilities、prohibitedActions、auditRequirements、reviewStatus。

任何扩展都不得绕过 Policy & Risk Engine、Confirmation Gate、Permission Sandbox、Stop / Takeover、Audit Event；不得直接执行 Submit / Delete / PermissionChange、直接写 Memory、删除或修改 Audit Event、后台读取未授权 App、外发 D3/D4、修改核心风险等级或把一次性授权升级为长期授权。

### 19.2 Observability

Janus 必须观察 Task Health、Policy Health、Permission Health、Execution Health、Verification Health、Interrupt Health、Audit Health、Memory Health、Privacy Health、Model Health、Extension Health。

受托关系质量指标优先于行动成功率：Unauthorized Action Count、Missed Confirmation Count、Stop Violation Count、Unverified Completion Count、ExternalImpactUnknown Count、MemoryMisuse Count、AuditMissing Count。

目标为 0 的指标：未确认外部可见动作、Stop 后继续执行、Takeover 后继续自动操作、未授权保存 Memory、删除或篡改 Audit Event、高敏数据未经许可外发、扩展绕过 Confirmation Gate、验证失败却展示 Completed。

可观测性观察系统行为，不画像用户；记录任务证据，不记录无关私人内容；默认本地聚合；上传诊断需授权。

### 19.3 测试体系

测试层级包括 Unit、Integration、Adapter、Scenario、Safety、Privacy、Memory、Recovery、Regression、Human Review。

必须测试：

```text
- 状态机转换和每次转换 Audit Event；
- L4/L5 进入 Confirmation Gate；
- 一次性授权不得升级为长期授权；
- Memory 不得替代 Permission；
- Permission Revoked 后 Step 不得执行；
- Policy Engine 不可用时不得执行受限动作；
- App Adapter 不得绕过风险判断；
- Strong / Medium / Weak / None Signal 策略；
- NegativeVerified 能证明未发送 / 未上传 / 未保存；
- Stop 后队列失效，Takeover 后不自动操作；
- Resume 前重新检查 Policy / Permission / Context；
- D4 不外发，D3 外发需授权或脱敏；
- 模型输出 schema 不合格不得执行；
- 外部影响不确定不得 Completed；
- Memory 误用进入信任恢复；
- Audit Timeline 包含失败证据。
```

Shadow Mode 用于新 App、新 Adapter、新策略和高风险流程：观察和规划但不执行，生成 Policy Decision、模拟 Verification、预演 Timeline，不产生外部影响。

### 19.4 能力发布治理

能力发布对象包括 App Adapter、Capability Manifest、Policy Rule、Verification Provider、Model Prompt Template、Model Role Configuration、Permission UI、Confirmation Gate UI、Memory Rule、Extension、Recovery Flow、Delegation Center Rule。

成熟度：Experimental、Preview、Limited、Stable、Restricted、Disabled。成熟度必须影响 Planner 是否默认使用。

发布流程：Design Review → Safety Review → Shadow Test → Internal Dogfood → Limited Rollout → Quality Monitoring → Stable Release / Rollback → Governance Record Update。

触发回滚：Q3 以上质量事件、Verification Failure 激增、Stop Violation、Missed Confirmation、Privacy Boundary 违规、Extension Sandbox 违规、用户大量撤销授权、Recovery 失败率升高。

RiskPolicy、PermissionPolicy、PrivacyPolicy、MemoryPolicy、ModelGatewayPolicy、AdapterManifest 必须版本化，每个 Task 记录策略版本。

---

## 20. 架构总览、对象表与追踪矩阵

### 20.1 核心依赖方向

```text
User Intent
→ Task Runtime
→ Reasoning / Planning
→ Policy / Permission / Risk
→ Execution
→ Verification
→ State Transition
→ Audit / Timeline
→ Recovery / Learning Candidate
→ Governance Update
```

禁止反向依赖：Execution Engine 绕过 Policy；App Adapter 决定用户目标；Model Gateway 放行动作；Memory 替代 Permission；Extension 绕过 Sandbox；Audit Logger 伪造完成状态；Planner 根据能力自行扩大任务目标。

### 20.2 三条横向控制线

用户控制线：Janus Entry / Janus Surface / Delegation Center → Interrupt Controller → Task State Machine → Execution Queue Invalidation → Audit Timeline。

风险授权线：Policy & Risk Engine → Permission Manager → Confirmation Gate → Permission Sandbox → Execution Engine。

证据复盘线：Verification Engine → Verification Event → Audit Event → Timeline Store → Recovery Manager → Delegation Center Backend。

### 20.3 核心对象总表

| 对象 | 核心含义 | 持久化 | 用户可见 |
|---|---|---|---|
| Intent | 用户表达的目标 | 任务内 | 是 |
| Task | 一次受托行动单元 | 是 | 是 |
| Context | 当前任务所需线索 | 任务内或引用 | 部分 |
| Plan | 可解释行动路径 | 是 | 是 |
| Step | 最小可执行单位 | 是 | 部分 |
| Action | 通用操作原语 | 类型 | 部分 |
| ObjectRef | 被操作对象引用 | 是 | 是 |
| EnvironmentRef | App、网页、窗口或系统环境引用 | 是 | 是 |
| PermissionGrant | 用户授予的行动边界 | 是 | 是 |
| DelegationRule | 长期授权规则 | 是 | 是 |
| Risk | 动作后果等级与类别 | 决策记录 | 部分 |
| PolicyDecision | Allow / Confirm / Block / Deny / Downgrade | 是 | 是 |
| ConfirmationPayload | 高风险确认内容 | 是 | 是 |
| ExecutableStep | 已通过策略和授权的 Step | 短期 | 部分 |
| VerificationRequirement | Step 预期验证要求 | 是 | 部分 |
| VerificationEvent | 动作结果证据 | 是 | 部分 |
| AuditEvent | 可回放行动记录 | 是 | 摘要可见 |
| Timeline | 用户可读任务时间线 | 是 | 是 |
| InterruptToken | Stop / Pause / Takeover 控制令牌 | 短期 / 引用 | 部分 |
| LearningCandidate | 待授权学习候选 | 候选态 | 是 |
| Memory | 授权保存的偏好、规则或环境知识 | 是 | 是 |
| CapabilityManifest | 环境能力说明 | 是 | 部分 |
| CapabilityCandidate | 待审核能力发现 | 是 | 管理可见 |
| AdapterStatus | 适配器健康与可信状态 | 是 | 管理可见 |
| RecoveryRecord | 失败、停下、误用后的恢复记录 | 是 | 是 |
| PrivacyDecision | 数据处理或外发判断 | 摘要 | 部分 |
| ModelReasoningEvent | 模型调用与输出校验记录 | 摘要 | 管理可见 |
| ExtensionManifest | 扩展能力和权限声明 | 是 | 管理可见 |
| QualityEvent | 系统质量与边界事件 | 是 | 管理可见 |

### 20.4 对象关系原则

```text
Intent 创建 Task；
Task 拥有 Plan；
Plan 由 Step 组成；
Step 引用 Action / ObjectRef / EnvironmentRef；
Step 必须绑定 VerificationRequirement；
受限 Step 必须绑定 PermissionGrant 或 DelegationRule；
PolicyDecision 决定 Step 是否可执行；
ExecutableStep 才能进入 Execution Engine；
执行结果必须生成 VerificationEvent；
VerificationEvent 必须进入 AuditEvent；
AuditEvent 投影为 Timeline；
失败或不确定生成 RecoveryRecord；
LearningCandidate 经授权后才可成为 Memory。
```

### 20.5 不可混淆对象

```text
Memory ≠ Permission；
System Permission ≠ Janus Permission；
Learning Candidate ≠ Memory；
Plan ≠ Authorization；
Model Confidence ≠ Verification Signal；
Adapter Capability ≠ User Authorization；
Audit Summary ≠ Audit Evidence；
Task Completion ≠ External Impact Verified；
User Confirmation Once ≠ DelegationRule。
```

### 20.6 技术原则追踪矩阵

| 原则 | 承载模块 | 关键对象 | 验证方式 | 反模式 |
|---|---|---|---|---|
| 目标由人给出 | Janus Entry / Intent Interpreter / Task Orchestrator | Intent / Task | Created 来源检查 | Agent 自主设定目标 |
| Plan 可解释 | Planner / Janus Surface | Plan / Step | Plan 展示与修改测试 | LLM 黑箱直接操作 |
| L4/L5 确认 | Policy & Risk Engine / Permission Manager | PolicyDecision / ConfirmationPayload | 高风险拦截测试 | Prompt 提醒代替确认 |
| 过程可见 | Task State Machine / Event Bus | TaskState / Timeline | 状态流测试 | 后台连续执行 |
| 权力可撤回 | Interrupt Controller | InterruptToken | 中断响应测试 | 停止后继续执行 |
| 能力不是权力 | Adapter Registry / Permission Sandbox | CapabilityManifest / PermissionGrant | 适配器越权测试 | 能点击就默认点击 |
| 当前窗口是线索 | Context Collector / Environment Model | Context / EnvironmentRef | 跨 App 任务测试 | 当前窗口插件化 |
| 每一步必须验证 | Verification Engine | VerificationRequirement / VerificationEvent | 状态机测试 | 连续脚本执行 |
| 不确定时停下 | Verification Engine / Task Orchestrator | VerificationEvent / RecoveryRecord | 弱信号测试 | 模型猜测继续 |
| 安全停下不是失败 | Recovery Manager | RecoveryRecord / Timeline | 失败路径测试 | 停下伪装完成 |
| 学习必须受托 | Memory & Learning Manager | LearningCandidate / Memory | 记忆授权测试 | 静默保存偏好 |
| 可复盘 | Audit Logger / Timeline Store | AuditEvent / Timeline | 审计完整性测试 | 只记录成功摘要 |
| 不代理身份主权 | Policy & Risk Engine | Risk / PolicyDecision | 身份表达测试 | 自动承诺、道歉、拒绝 |
| 不做行为监控 | Context Collector / Privacy Boundary Manager | Context / PrivacyDecision | 隐私边界测试 | 长期后台观察 |
| 第三方能力受限 | Extension Sandbox | ExtensionManifest / QualityEvent | 沙箱违规测试 | 扩展绕过确认门 |

---

## 21. 后续派生文档规则

本文档是技术总纲和运行时规格，不替代所有具体技术文档。后续应派生：模块接口设计、Task / Plan / Step 状态机规范、权限与风险策略规范、Verification / Audit 数据规范、App Adapter / Capability Manifest 规范、Model Gateway 规范、Privacy Boundary 规范、Extension Sandbox 规范、Delegation Center Backend 详细设计、测试与安全评审计划、派生产品切片技术方案、技术选型与工程实施计划。

派生文档必须标明对应章节，使用本文核心对象语言，不改变用户主权优先级，不降低风险确认标准，不把模型置信度当验证，不把系统权限当用户授权，不把 Memory 当 Permission，不把失败伪装完成，定义验证方式，并明确非目标。

本文只应在总纲原则、上位技术对象、系统级风险、Runtime 分层或用户主权 / 授权 / 验证 / 审计 / 记忆 / 扩展边界变化时修改；普通实现细节变化不应回写到技术总纲。
