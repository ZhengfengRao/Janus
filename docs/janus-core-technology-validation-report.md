# Janus 核心依赖技术论证与验证报告

## 0. 本文定位 / 与其他文档关系

本文是 Janus 的**可行性与验证门报告**，负责把产品总纲、体验总纲和技术总架构转化为研发前必须通过的技术成熟度判断、路线选择、阶段门和风险边界。

本文拥有以下判断：

- Janus 是否具备可分阶段验证的技术可行性；
- 关键技术能力的成熟度、限制、风险和验证方式；
- 研发路线选择和先后顺序；
- 阶段门、证据标准和失败标准；
- 推荐研发任务包；
- 一票否决风险和高概率工程风险。

本文**不拥有**运行时架构对象的完整定义、字段 schema、状态机细节、接口协议或实现方案。Runtime 对象、模块职责、状态转换和架构细节以 `janus-technical-architecture-design.md` 为源头文档。本文只在验证需要时概念性引用 Task、Step、AutonomyPolicy、Permission、StandingAuthorization、RiskBudget、Verification、Audit、Memory、Delegation Center、Janus Entry 和 Janus Surface。

本文的核心问题是：Janus 能否成为 **full-capability entrusted Agent / 全能受托 Agent**——通过分阶段验证 macOS runtime、自主策略、授权执行、风险预算、恢复和审计，逐步从受控任务扩展到授权范围内的复杂跨 App 目标完成。

---

## 1. 结论摘要

### 1.1 总体判断

Janus 在技术上是可论证、可验证、可分阶段推进的，但它不应被理解为：

```text
LLM + 键鼠模拟 + 若干 App 脚本。
```

更准确的技术判断是：

```text
Janus 是以 Task 为中心、以自主策略 / 授权 / 风险预算为边界、以单步执行和证据验证为运行纪律、以用户可中断和可复盘为信任基础的 macOS 全能受托 Agent Runtime。
```

研发难点不是“Agent 能不能点按钮”，而是：

```text
能否在跨 App、跨上下文、存在外部影响的桌面环境中，完成用户授权的数字目标，并持续保证目标来自用户、路径可解释、动作可授权、风险预算可治理、结果可验证、过程可接管、事后可复盘。
```

Janus 的早期成功标准必须区分长期愿景与验证顺序：

```text
长期愿景：全能受托 Agent，在授权范围和风险预算内完成电脑任务。
早期门槛：先用受控、可撤销、低风险任务验证 runtime、自主策略、恢复和审计，再逐步扩大自主性与外部影响范围。
```

因此，高风险动作约束是验证要求，不是产品身份；低风险 / 草稿 / 准备任务是第一成熟度切片，不是最终定位。

### 1.2 技术可行性等级

| 类别 | 技术能力 | 可行性判断 | 研发含义 |
|---|---|---|---|
| A 类：基础可行 | Janus Entry、本地运行时、任务状态机、本地事件流、审计时间线、安全存储、风险规则引擎 | 高 | 应作为最早工程底座 |
| B 类：可行但需严格约束 | Accessibility 控制、屏幕/OCR 辅助、跨 App 执行、验证信号系统、薄 App Adapter | 中-高 | 必须单步执行、强验证、受控任务先行 |
| C 类：可行但产品边界敏感 | 模型规划、对象解析、上下文理解、记忆学习、standing authorization、risk budget | 中 | 必须通过策略层、授权层、预算层和审计层约束 |
| D 类：长期复杂能力 | 第三方扩展生态、多任务并发、通用 App 自探索、自我能力提升、企业组织治理 | 中-低到中 | 不应进入早期核心闭环 |

### 1.3 核心技术路线判断

Janus 后续研发应采用：

```text
Evidence-first Native Runtime
证据优先的本地桌面运行时路线
```

顺序是：

1. 先建立 Task / Step / AutonomyPolicy / Permission / RiskBudget / Verification / Audit 的不可变运行纪律；
2. 再接入 macOS 控制能力；
3. 再接入模型理解和规划能力；
4. 再验证 standing authorization 覆盖的重复 / 可撤销任务；
5. 最后扩展 App 能力、记忆能力、外部影响执行、复杂多步目标、并发能力和第三方扩展能力。

不能反过来先做“聪明的模型自动操作”，再补权限、审计和验证。证据优先也不等于慢执行：它的目标是让授权内任务更快、更可靠地完成，并把升级集中在授权、风险预算、验证和用户责任边界。

---

## 2. 论证方法与通过标准

### 2.1 总纲追踪

每项核心技术都必须追溯到至少一类约束：

| 来源 | 核心约束 |
|---|---|
| 产品总纲 | Janus 是全能受托 Agent；能力不是权力；目标由人给出；授权内自主完成；超界升级；权力必须可撤回 |
| 产品体验总纲 | Janus Entry、Janus Surface、Delegation Center；当前上下文是线索不是边界；任务必须可见、可接管、可复盘 |
| 技术总纲 | Task 中心、Step 最小执行单位、Planner/Policy 分离、Autonomy Policy Engine、Risk Budget Manager、Verification Event、Audit Event、Memory 授权 |

如果某项技术无法追溯到这些约束，或会破坏更高优先级约束，应推迟、降级或拒绝。

### 2.2 技术成熟度判断

| 等级 | 含义 |
|---|---|
| T0 | 概念成立，但未验证工程路径 |
| T1 | 单点技术可验证 |
| T2 | 可在受控任务链路中验证 |
| T3 | 可支撑低风险真实任务 |
| T4 | 可支撑授权重复 / 可撤销任务 |
| T5 | 可支撑外部影响 standing authorization 与复杂多步目标 |

任何进入真实外部影响链路的能力，必须至少达到 T2，并受明确授权、风险预算、验证和升级策略约束。

### 2.3 风险优先级

```text
越权风险 > 风险预算违规 > 外部影响风险 > 隐私风险 > 错误执行风险 > 可用性风险 > 效率风险
```

验证不能只证明“没有做错事”，还必须证明 Janus 能快速完成有用的授权任务。一个执行成功率更高但更难审计、更难中断、更容易越权的方案，不应优先于一个执行范围较窄但可验证、可接管、可复盘的方案；在边界同样清晰的情况下，更少界面劳动、更高授权内完成率，是更优方案。

### 2.4 证据优先于实现

研发通过不以“模块已实现”为准，而以证据为准：

1. 能证明该模块减少界面劳动并推进授权任务完成；
2. 能证明该模块没有越过自己的权力边界或风险预算；
3. 能证明它与 Task / Step / AutonomyPolicy / Permission / RiskBudget / Verification / Audit 正确连接；
4. 能证明失败、不确定、用户中断时会安全停下或升级；
5. 能证明其行为可被用户理解、接管和复盘。

模型置信度不是验证，视觉相似不是充分验证。验证必须来自可解释证据。

---

## 3. 不可变技术约束

### 3.1 产品哲学约束到技术约束

| 哲学约束 | 技术约束 |
|---|---|
| 目标由人给出 | Task 必须由用户 Intent 创建，Agent 不得自建目标 |
| 路径由 Agent 处理 | Planner 可生成候选 Plan，但必须可解释、可修改 |
| 授权内自主完成 | Autonomy Policy Engine + Permission Manager + Risk Budget Manager 共同放行 Step |
| 超界升级确认 | 未覆盖 L4/L5、R 类风险、预算不足、验证不足必须进入 Confirmation / Escalation Gate |
| 过程必须可见 | Task State、Step、Verification、Audit 必须投影到 Janus Surface / Action Trace |
| 权力必须可撤回 | Interrupt Controller 必须高于 Execution Queue |

### 3.2 三个产品表面

| 产品层 | 技术含义 | 边界 |
|---|---|---|
| Janus Entry | 任意 macOS 工作现场可唤起；可从快捷键、选中对象、胶囊入口、按住说话创建 Intent | 只做入口、状态锚点和最小控制，不承载完整任务解释或托管治理 |
| Janus Surface | 当前受托任务的理解、计划、自主策略、风险预算、执行、验证、中断、确认、复盘可见 | 不替代 Delegation Center 的长期治理 |
| Delegation Center | 跨任务治理托管、授权、standing authorization、风险预算、记忆、历史、审计、信任恢复和能力降级证据 | 不成为当前任务确认流 |

目标 App 现场是用户直接操作的最高优先级场域。用户直接编辑、发送、关闭或接管时，Janus 应暂停或失效旧执行队列，而不是要求用户先“申请接管”。

### 3.3 技术硬规则

```text
1. 没有 Task，不得执行 Action；
2. 没有 Permission，不得执行受限 Action；
3. 没有 RiskBudget，不得执行预算型自主外部影响动作；
4. 没有 Verification Requirement，不得自动执行 Step；
5. L4/L5 动作必须经过 Autonomy Policy / Risk Budget / Confirmation Gate；
6. R 类风险事件候选必须被拦截、升级确认或禁止；
7. Executing 后必须进入 Verifying；
8. 用户 Stop / Takeover 后旧执行队列必须失效；
9. Learning Candidate 不能自动保存为 Memory；
10. Audit Event 必须记录已做、未做、授权依据和风险预算消耗；
11. Recovery 不能伪装为 Completed。
```

这些规则是后续模块接口、测试用例和代码审查的通过标准。

---

## 4. 核心技术地图与压缩论证

Janus 依赖的核心技术按权力链路组织，而不是按功能清单组织：

```text
Intent
→ Task
→ Plan
→ Autonomy Policy
→ Permission / Standing Authorization
→ Risk Budget
→ Executable Step
→ Execution
→ Verification
→ State Transition
→ Audit / Timeline
→ Recovery / Learning Candidate
→ Governance Update
```

### 4.1 十二组核心能力压缩表

| # | 技术能力 | 目标 | 成熟度 / 可行性 | 主要限制与风险 | 必须验证 | 研发建议 |
|---|---|---|---|---|---|---|
| 1 | macOS Janus Entry 与本地运行时 | 任意 App 工作现场唤起 Janus，并将用户表达转化为 Intent | T2-T3 / 高 | 系统权限、焦点、多屏幕、多 Space、全屏 App；入口常驻滑向后台观察者 | 任意 App 唤起；空闲不采集；按住说话 down 采集、release 提交、cancel 不创建 Task；胶囊可停止 / 接管 / 展开 Janus Surface | 先做唤起、Intent 输入、状态锚点和中断控制；不做长期观察或主动推荐 |
| 2 | 上下文采集与环境建模 | 理解当前任务现场、目标环境、选中对象和可操作状态 | T1-T3 / 中-高 | App 可访问性不一致、封闭 App 结构化信号不足、OCR 不稳定、隐私敏感 | 最小采集；结构化优先；敏感降级；当前上下文只作线索；低质量 Context 不驱动高风险 Step | 先做 Immediate / Execution / Verification / Recovery Context Mode；不做全局桌面理解或长期画像 |
| 3 | 意图理解与任务建模 | 把自然语言目标转化为结构化 Intent / Task，并保存边界、对象、限制和不确定性 | T2 / 中-高 | 自然语言歧义、重名对象、隐含授权误判、目标过宽 | 每个 Task 可追溯到用户表达；“客户承诺先问我 / 只按模板发送”等限制结构化保存；歧义时澄清或阻塞 | 模型可参与理解，但 Task 创建必须经过 schema 校验和策略预检查 |
| 4 | Plan 生成与策略分离 | Planner 生成可解释、可修改、可验证的 Plan；Policy 独立判断是否允许 | T1-T2 / 中 | LLM 计划漂移、虚构能力、步骤粒度过粗、用 prompt 替代 Policy | Plan 可解释；Step 可单独执行 / 验证 / 中断 / 审计；Planner 高风险建议被 Policy 拦截；用户修改后重评估 | 早期只要求结构化 Plan、暴露不确定性、可被 Policy 拒绝或降级 |
| 5 | 自主策略、权限、风险预算与确认门 | 保证能力不会自动转化为权力，同时允许授权内自主完成 | T2-T3 / 高 | 风险分类覆盖、语义风险、长期授权误用、预算虚设 | L4/L5 和 R 类未覆盖时升级；Permission 绑定 Environment、Object、Action、Risk、Duration、Scope；RiskBudget 可消耗、可撤销、可审计 | Autonomy Policy Engine、Permission Manager、Risk Budget Manager 早于真实执行器完成 |
| 6 | 跨 App 执行与通用操作原语 | 用通用原语控制 App、网页、文件系统和系统环境，避免逐 App 重脚本 | T1-T3 / 中 | App 可访问性差异、UI 更新、焦点漂移、模拟键鼠不稳定；退化为 RPA | 操作映射到 Observe / Open / Switch / Search / Select / Input / Draft / Attach / Preview / Submit 等通用 Action；一次一步；fallback 不驱动高风险动作 | 优先结构化 API、Accessibility、DOM；OCR / Vision / 键鼠只作低可信 fallback |
| 7 | 验证信号与证据系统 | 证明每一步做了什么、没做什么、在哪里停下、是否产生外部影响 | T1-T2 / 中-高 | 很多外部影响难完全证明未发生；视觉证据不稳定；用模型置信度替代验证 | 每个 Step 执行前有 VerificationRequirement；执行后进入 Verifying；Weak / None 不允许继续高风险动作；NegativeVerified 能证明未发送 / 未上传 / 未保存 | Verification Engine 与 Execution Engine 同步建设 |
| 8 | 用户中断与接管 | 保证 Stop / Pause / Takeover / Revoke 始终高于模型计划和执行队列 | T2-T3 / 高 | 某些底层动作不可瞬时取消；UI 显示停止但后台继续执行 | Stop 后旧队列失效；用户接管后不自动恢复；权限撤销后相关 Step 不可执行；Timeline 记录已做 / 未做 / 为何停止 | Interrupt Controller 早于复杂执行器完成 |
| 9 | 审计、时间线与信任恢复 | 记录 Janus 做了什么、没做什么、基于什么授权、消耗什么预算、为什么停下 | T2-T3 / 高 | 证据保存与隐私最小化冲突；只记录成功摘要 | Intent、Plan、Policy、Permission、RiskBudget、Execution、Verification、State Transition 完整记录；Recovery 不伪装为 Completed | 第一条任务链路就必须有审计 |
| 10 | 记忆、学习与长期授权 | 在用户授权下沉淀偏好、联系人映射、环境知识和流程规则 | T1-T2 / 中 | 偏好推断过度；一次性授权被误判为长期规则；Memory 替代 Permission | Learning Candidate 不自动保存；授权卡说明记住什么、为什么、用于哪里、如何撤销；记住偏好不跳过发送确认 | 早期先做候选、解释、拒绝、过期、删除，不急于自动 Memory |
| 11 | 数据、隐私、安全与沙箱 | 在采集、模型调用、执行、审计、记忆、扩展之间建立统一安全边界 | T2-T3 / 高 | macOS 系统权限粒度较粗；隐私与可审计存在张力 | D0-D5 影响采集、模型调用、审计、记忆、扩展访问；D4 默认不可读 / 不保存 / 不代填；SafeMode 禁止高风险动作 | 最小底座包含 DataLevel、PrivacyDecision、Permission Sandbox、SafeMode |
| 12 | 模型网关、适配器治理与可观测性 | 让模型、Adapter 和扩展作为能力组件受 Runtime 治理，而不是成为权力中心 | T1-T2 / 中 | 模型不稳定、适配器质量差异、第三方扩展不可控、指标诱导错误优化 | 模型输出经 Schema、Policy、Permission、RiskBudget、Verification、用户可见状态；Adapter 只声明能力，不决定授权 / 风险 / 完成状态 | Model Gateway 应在 Planner 之前完成最小版本；业务代码不得直接执行模型建议 |

### 4.2 边界分离必须保留

```text
System Permission ≠ Janus Permission
Capability ≠ Authorization
Current Context ≠ Task Boundary
User Confirmation Once ≠ DelegationRule
StandingAuthorization ≠ Unlimited Authorization
RiskBudget ≠ Permission To Ignore Verification
Learning Candidate ≠ Memory
Memory ≠ Permission
Model Confidence ≠ Verification Evidence
Adapter Capability ≠ Action Legitimacy
```

---

## 5. 分阶段验证门

Janus 的验证路线必须证明“可以成为全能受托 Agent”，但不能假装 full autonomy 已经被验证。建议按以下 gates 推进：

| Gate | 名称 | 目标 | 通过标准 | 失败标准 |
|---|---|---|---|---|
| Gate 1 | Runtime kernel | 建立 Task / Step / AutonomyPolicy / Permission / RiskBudget / Verification / Audit 骨架 | 任意 Intent 可进入状态机；Action 必须绑定 Task；审计记录 done / not done | 可绕过 Task 或审计执行动作 |
| Gate 2 | Low-risk immediate execution | 验证低风险即时跨 App 执行 | 可完成打开、搜索、整理、起草、预填；每步可验证、可停止、可接管 | 需要连续手动操作；停止后仍执行；验证缺失仍继续 |
| Gate 3 | Authorized recurring / reversible execution | 验证 standing authorization 下的重复 / 可撤销任务 | 固定对象、模板、频率、期限内可无逐步确认完成；撤销立即生效；审计显示授权依据 | 一次确认被泛化；撤销后继续；预算不可见 |
| Gate 4 | External-impact with standing authorization and escalation | 验证外部可见动作的授权执行与超界升级 | 例行发送 / 更新共享文档等可在授权和风险预算内执行；新增对象、客户承诺、权限变化等升级；risk-budget violations = 0 | 未授权外部影响；超预算执行；升级卡缺少对象 / 影响 / 授权依据 |
| Gate 5 | Complex multi-step autonomous goals | 验证复杂跨 App 目标完成 | 多 App、多对象、多约束目标可在授权范围内完成；失败可恢复；用户仍能接管和复盘 | Agent 漂移目标；无法解释授权；把 Blocked / Recovery 伪装 Completed |

---

## 6. 推荐研发任务包

| 包 | 内容 | 先验门 |
|---|---|---|
| P0 Runtime Kernel | Task State Machine、Event Bus、Audit Logger、Interrupt Controller | Gate 1 |
| P1 Entry + Surface Slice | Janus Entry、Intent Capture、Action Trace、Completed / Recovery 状态 | Gate 1-2 |
| P2 Controlled Execution | macOS 控制原语、Verification Engine、Adapter Manifest | Gate 2 |
| P3 Autonomy Policy + Risk Budget | Autonomy Policy Engine、Permission Manager、Risk Budget Manager、Escalation Gate | Gate 2-3 |
| P4 Standing Authorization | Delegation Center 规则、撤销、过期、频率、审计 | Gate 3 |
| P5 External-impact Authorized Execution | 例行发送、共享文档更新、外部影响审计、超界升级 | Gate 4 |
| P6 Complex Goal Orchestration | 多 App 目标规划、恢复、跨任务治理 | Gate 5 |

---

## 7. 一票否决风险

以下风险出现时，不得扩大自主等级：

```text
- 未授权外部影响；
- risk-budget violation；
- Stop / Takeover 后继续执行；
- 一次确认被静默转为 standing authorization；
- Memory 被当作 Permission；
- 模型置信度替代 Verification；
- Recovery / Blocked / SafelyStopped 被展示为 Completed；
- Delegation Center 无法撤销或审计授权；
- Adapter 或扩展绕过 Policy / Permission / RiskBudget。
```

---

## 8. 最终判断

Janus 的技术可行性不是一次性证明“通用 Agent 已经能全自动操作电脑”，而是通过一组不可跳过的成熟度门，证明：

```text
1. 受托行动 runtime 可被构造；
2. 低风险跨 App 执行可被验证；
3. standing authorization 可安全支持重复 / 可撤销任务；
4. 外部影响可在明确授权和风险预算内执行，并在超界时升级；
5. 复杂多步目标可在授权范围内完成、恢复、审计和治理。
```

如果这些门逐步通过，Janus 才能从早期验证切片走向 full-capability entrusted Agent。若任何阶段无法证明授权、风险预算、验证、接管和审计，正确做法不是把产品重新定义为低风险助手，而是暂停扩大自主性，修复 runtime 和治理基础。
