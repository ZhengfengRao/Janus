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

本文**不拥有**运行时架构对象的完整定义、字段 schema、状态机细节、接口协议或实现方案。Runtime 对象、模块职责、状态转换和架构细节以 `janus-technical-architecture-design.md` 为源头文档。本文只在验证需要时概念性引用 Task、Step、Permission、Risk、Verification、Audit、Memory、Delegation Center、Janus Entry 和 Janus Surface。

本文的核心问题是：Janus 作为**通用受托行动层**，能否在真实桌面环境中，把用户目标转化为低风险、跨 App、可检查、可编辑、可确认的结果，并在外部影响边界前硬停。

---

## 1. 结论摘要

### 1.1 总体判断

Janus 在技术上是可论证、可验证、可分阶段推进的，但它不应被理解为：

```text
LLM + 键鼠模拟 + 若干 App 脚本。
```

更准确的技术判断是：

```text
Janus 是以 Task 为中心、以权限和风险为边界、以单步执行和证据验证为运行纪律、以用户可中断和可复盘为信任基础的 macOS 桌面受托行动运行时。
```

研发难点不是“Agent 能不能点按钮”，而是：

```text
能否在跨 App、跨上下文、存在外部影响的桌面环境中，快速完成有用的低风险机械路径，并持续保证目标来自用户、路径可解释、动作可授权、结果可验证、过程可接管、事后可复盘。
```

Janus 的早期成功标准必须是价值优先：

```text
快速把用户带到有用、低风险、可检查、可编辑、可确认的结果；
一旦接近发送、提交、上传、删除、授权变更等外部影响边界，必须硬停并等待用户确认。
```

### 1.2 技术可行性等级

| 类别 | 技术能力 | 可行性判断 | 研发含义 |
|---|---|---|---|
| A 类：基础可行 | Janus Entry、本地运行时、任务状态机、本地事件流、审计时间线、安全存储、风险规则引擎 | 高 | 应作为最早工程底座 |
| B 类：可行但需严格约束 | Accessibility 控制、屏幕/OCR 辅助、跨 App 执行、验证信号系统、薄 App Adapter | 中-高 | 必须单步执行、强验证、低风险先行 |
| C 类：可行但产品边界敏感 | 模型规划、对象解析、上下文理解、记忆学习、长期授权 | 中 | 必须通过策略层、授权层和审计层约束 |
| D 类：长期复杂能力 | 第三方扩展生态、多任务并发、通用 App 自探索、自我能力提升、企业组织治理 | 中-低到中 | 不应进入早期核心闭环 |

### 1.3 核心技术路线判断

Janus 后续研发应采用：

```text
Evidence-first Native Runtime
证据优先的本地桌面运行时路线
```

顺序是：

1. 先建立 Task / Step / Permission / Risk / Verification / Audit 的不可变运行纪律；
2. 再接入 macOS 控制能力；
3. 再接入模型理解和规划能力；
4. 最后扩展 App 能力、记忆能力、并发能力和第三方扩展能力。

不能反过来先做“聪明的模型自动操作”，再补权限、审计和验证。证据优先也不等于慢执行：它的目标是让低风险机械路径更快、更可靠地完成，并把硬停集中在外部影响和用户责任边界。

---

## 2. 论证方法与通过标准

### 2.1 总纲追踪

每项核心技术都必须追溯到至少一类约束：

| 来源 | 核心约束 |
|---|---|
| 产品总纲 | Janus 是人的受托数字行动入口；能力不是权力；目标由人给出；关键动作由人确认；权力必须可撤回 |
| 产品体验总纲 | Janus Entry、Janus Surface、Delegation Center；当前上下文是线索不是边界；任务必须可见、可接管、可复盘 |
| 技术总纲 | Task 中心、Step 最小执行单位、Planner/Policy 分离、Permission Sandbox、Verification Event、Audit Event、Memory 授权 |

如果某项技术无法追溯到这些约束，或会破坏更高优先级约束，应推迟、降级或拒绝。

### 2.2 技术成熟度判断

| 等级 | 含义 |
|---|---|
| T0 | 概念成立，但未验证工程路径 |
| T1 | 单点技术可验证 |
| T2 | 可在受控任务链路中验证 |
| T3 | 可支撑低风险真实任务 |
| T4 | 可支撑长期产品化治理 |

任何进入真实外部影响链路的能力，必须至少达到 T2，并受风险策略约束。

### 2.3 风险优先级

```text
越权风险 > 外部影响风险 > 隐私风险 > 错误执行风险 > 可用性风险 > 效率风险
```

验证不能只证明“没有做错事”，还必须证明 Janus 能快速完成有用的低风险机械路径。一个执行成功率更高但更难审计、更难中断、更容易越权的方案，不应优先于一个执行范围较窄但可验证、可接管、可复盘的方案；在边界同样清晰的情况下，更少界面劳动、更快到达可检查结果，是更优方案。

### 2.4 证据优先于实现

研发通过不以“模块已实现”为准，而以证据为准：

1. 能证明该模块减少界面劳动并推进低风险机械路径；
2. 能证明该模块没有越过自己的权力边界；
3. 能证明它与 Task / Step / Permission / Risk / Verification / Audit 正确连接；
4. 能证明失败、不确定、用户中断时会安全停下；
5. 能证明其行为可被用户理解、接管和复盘。

模型置信度不是验证，视觉相似不是充分验证。验证必须来自可解释证据。

---

## 3. 不可变技术约束

### 3.1 产品哲学约束到技术约束

| 哲学约束 | 技术约束 |
|---|---|
| 目标由人给出 | Task 必须由用户 Intent 创建，Agent 不得自建目标 |
| 路径由 Agent 处理 | Planner 可生成候选 Plan，但必须可解释、可修改 |
| 关键动作由人确认 | L4/L5 和 R 类风险事件必须进入 Confirmation Gate |
| 过程必须可见 | Task State、Step、Verification、Audit 必须投影到 Janus Surface / Action Trace |
| 权力必须可撤回 | Interrupt Controller 必须高于 Execution Queue |

### 3.2 三个产品表面

| 产品层 | 技术含义 | 边界 |
|---|---|---|
| Janus Entry | 任意 macOS 工作现场可唤起；可从快捷键、选中对象、胶囊入口、按住说话创建 Intent | 只做入口、状态锚点和最小控制，不承载完整任务解释或托管治理 |
| Janus Surface | 当前受托任务的理解、计划、风险、执行、验证、中断、确认、复盘可见；低风险清晰任务可把理解和 Plan 折叠进 Action Trace | 不替代 Delegation Center 的长期治理 |
| Delegation Center | 跨任务治理托管、权限、记忆、历史、审计、信任恢复和能力降级证据 | 不成为当前任务确认流 |

目标 App 现场是用户直接操作的最高优先级场域。用户直接编辑、发送、关闭或接管时，Janus 应暂停或失效旧执行队列，而不是要求用户先“申请接管”。

### 3.3 技术硬规则

```text
1. 没有 Task，不得执行 Action；
2. 没有 Permission，不得执行受限 Action；
3. 没有 Verification Requirement，不得自动执行 Step；
4. L4/L5 动作必须经过 Confirmation Gate；
5. R 类风险事件候选必须被拦截或进入人工确认；
6. Executing 后必须进入 Verifying；
7. 用户 Stop / Takeover 后旧执行队列必须失效；
8. Learning Candidate 不能自动保存为 Memory；
9. Audit Event 必须记录已做和未做；
10. Recovery 不能伪装为 Completed。
```

这些规则是后续模块接口、测试用例和代码审查的通过标准。

---

## 4. 核心技术地图与压缩论证

Janus 依赖的核心技术按权力链路组织，而不是按功能清单组织：

```text
Intent
→ Task
→ Plan
→ Policy / Permission / Risk
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
| 3 | 意图理解与任务建模 | 把自然语言目标转化为结构化 Intent / Task，并保存边界、对象、限制和不确定性 | T2 / 中-高 | 自然语言歧义、重名对象、隐含授权误判、目标过宽 | 每个 Task 可追溯到用户表达；“发送前确认 / 不要发 / 只创建草稿”等限制结构化保存；歧义时澄清或阻塞 | 模型可参与理解，但 Task 创建必须经过 schema 校验和策略预检查 |
| 4 | Plan 生成与策略分离 | Planner 生成可解释、可修改、可验证的 Plan；Policy 独立判断是否允许 | T1-T2 / 中 | LLM 计划漂移、虚构能力、步骤粒度过粗、用 prompt 替代 Policy | Plan 可解释；Step 可单独执行 / 验证 / 中断 / 审计；Planner 高风险建议被 Policy 拦截；用户修改后重评估 | 早期只要求结构化 Plan、暴露不确定性、可被 Policy 拒绝或降级 |
| 5 | 权限、风险与确认门 | 保证能力不会自动转化为权力 | T2-T3 / 高 | 风险分类覆盖、语义风险、长期授权误用；用系统权限替代 Janus 权限 | L4/L5 和 R 类硬停；Permission 绑定 Environment、Object、Action、Risk、Duration、Scope；撤销后旧 Step 不可继续 | 权限与风险引擎早于真实执行器完成 |
| 6 | 跨 App 执行与通用操作原语 | 用通用原语控制 App、网页、文件系统和系统环境，避免逐 App 重脚本 | T1-T3 / 中 | App 可访问性差异、UI 更新、焦点漂移、模拟键鼠不稳定；退化为 RPA | 操作映射到 Observe / Open / Switch / Search / Select / Input / Draft / Attach / Preview / Submit 等通用 Action；一次一步；fallback 不驱动高风险动作 | 优先结构化 API、Accessibility、DOM；OCR / Vision / 键鼠只作低可信 fallback |
| 7 | 验证信号与证据系统 | 证明每一步做了什么、没做什么、在哪里停下、是否产生外部影响 | T1-T2 / 中-高 | 很多外部影响难完全证明未发生；视觉证据不稳定；用模型置信度替代验证 | 每个 Step 执行前有 VerificationRequirement；执行后进入 Verifying；Weak / None 不允许继续高风险动作；NegativeVerified 能证明未发送 / 未上传 / 未保存 | Verification Engine 与 Execution Engine 同步建设 |
| 8 | 用户中断与接管 | 保证 Stop / Pause / Takeover / Revoke 始终高于模型计划和执行队列 | T2-T3 / 高 | 某些底层动作不可瞬时取消；UI 显示停止但后台继续执行 | Stop 后旧队列失效；用户接管后不自动恢复；权限撤销后相关 Step 不可执行；Timeline 记录已做 / 未做 / 为何停止 | Interrupt Controller 早于复杂执行器完成 |
| 9 | 审计、时间线与信任恢复 | 记录 Janus 做了什么、没做什么、为什么停下、是否产生外部影响 | T2-T3 / 高 | 证据保存与隐私最小化冲突；只记录成功摘要 | Intent、Plan、Policy、Execution、Verification、State Transition 完整记录；未发送 / 未上传 / 未保存记忆明确记录；Recovery 不伪装为 Completed | 第一条任务链路就必须有审计 |
| 10 | 记忆、学习与长期授权 | 在用户授权下沉淀偏好、联系人映射、环境知识和流程规则 | T1-T2 / 中 | 偏好推断过度；一次性授权被误判为长期规则；Memory 替代 Permission | Learning Candidate 不自动保存；授权卡说明记住什么、为什么、用于哪里、如何撤销；记住偏好不跳过发送确认 | 早期先做候选、解释、拒绝、过期、删除，不急于自动 Memory |
| 11 | 数据、隐私、安全与沙箱 | 在采集、模型调用、执行、审计、记忆、扩展之间建立统一安全边界 | T2-T3 / 高 | macOS 系统权限粒度较粗；隐私与可审计存在张力 | D0-D5 影响采集、模型调用、审计、记忆、扩展访问；D4 默认不可读 / 不保存 / 不代填；SafeMode 禁止高风险动作 | 最小底座包含 DataLevel、PrivacyDecision、Permission Sandbox、SafeMode |
| 12 | 模型网关、适配器治理与可观测性 | 让模型、Adapter 和扩展作为能力组件受 Runtime 治理，而不是成为权力中心 | T1-T2 / 中 | 模型不稳定、适配器质量差异、第三方扩展不可控、指标诱导错误优化 | 模型输出经 Schema、Policy、Permission、Verification、用户可见状态；Adapter 只声明能力，不决定授权 / 风险 / 完成状态 | Model Gateway 应在 Planner 之前完成最小版本；业务代码不得直接执行模型建议 |

### 4.2 边界分离必须保留

```text
System Permission ≠ Janus Permission
Capability ≠ Authorization
Current Context ≠ Task Boundary
User Confirmation Once ≠ DelegationRule
Learning Candidate ≠ Memory
Memory ≠ Permission
Model Confidence ≠ Verification Evidence
Adapter Capability ≠ Action Legitimacy
```

---

## 5. 核心技术依赖矩阵

| 技术能力 | 支撑原则 | 关键对象 | 成熟度 | 主要风险 | 必须验证 |
|---|---|---|---|---|---|
| Janus Entry | 目标由人给出 | Intent / Task | T2-T3 | 后台监控化 | 空闲不采集；任意 App 可唤起；按住说话 release 可追溯 |
| Context | 当前上下文是线索 | Context / PrivacyDecision | T1-T3 | 过度采集 | 最小采集；敏感降级；目标 App 可独立建模 |
| Task | 没有 Task 不行动 | Intent / Task | T2 | 目标误解 | Intent 来源、边界、不确定性 |
| Plan | 路径由 Agent 处理 | Plan / Step | T1-T2 | 黑箱计划 | 可解释、可修改、可拒绝、可折叠进 Action Trace |
| Policy & Risk | 能力不是权力 | Risk / PolicyDecision | T2-T3 | prompt 替代策略 | L4/L5/R 类拦截 |
| Permission | 授权边界 | PermissionGrant / DelegationRule | T2 | 一次确认变长期授权 | 状态机、撤销、过期、范围绑定 |
| Execution | 单步受控执行 | ExecutableStep / ActionResult | T1-T2 | 连续脚本化 | 一次一步；中断检查 |
| Adapter | 通用能力优先 | CapabilityManifest | T1-T2 | 厚脚本化 | 只能描述能力，不决策 |
| Verification | 每一步必须验证 | VerificationRequirement / Event | T1-T2 | 弱信号继续 | NegativeVerified；弱信号停下 |
| Interrupt | 权力可撤回 | InterruptToken | T2-T3 | 停止后继续 | 队列失效；审计记录 |
| Audit | 可复盘 | AuditEvent / Timeline | T2-T3 | 只记成功 | 已做与未做完整记录 |
| Memory | 学习必须受托 | LearningCandidate / Memory | T1-T2 | 静默保存 | 候选、授权、撤销 |
| Privacy | 不做行为监控 | DataLevel / PrivacyDecision | T2 | 数据外发 | D4 阻断；云端脱敏 |
| Model Gateway | 模型不是权力中心 | ModelReasoningEvent | T1-T2 | 模型越权 | Schema、Policy、Audit |
| SafeMode | 不确定时停下 | RecoveryRecord / QualityEvent | T2 | 带病执行 | 安全能力故障时降级 |

---

## 6. 关键技术路线选择

### 6.1 原生 macOS Runtime 优先，而不是跨平台壳优先

Janus 早期技术验证应优先采用原生 macOS Runtime。原因是 Janus 深度依赖 macOS 桌面环境、系统权限、窗口状态、Accessibility、全局快捷键、本地安全存储和执行控制。

边界：这不意味着所有 UI 永久原生，也不排除未来 Web 管理后台或跨平台组件。判断仅针对早期 Runtime 底座。

### 6.2 结构化信号优先，而不是视觉优先

Janus 应优先使用结构化 API、Accessibility Tree、DOM、文件系统状态等强信号。OCR / Vision / 截图只作为补充，模拟键鼠作为最后 fallback。

边界：视觉能力在封闭 App 中仍有价值，但必须与风险策略绑定，不能直接成为高风险执行依据。

### 6.3 通用原语 + 薄适配，而不是逐 App 重脚本

Janus 应将 App 能力抽象为 Environment / Object / Action / State / Verification Signal，而不是为每个 App、每个场景、每个按钮写完整脚本。

薄适配可以提供启动方式、常见控件定位规则、可用结构化接口声明、低风险导航路径和能力描述，但不能成为业务决策中心。

### 6.4 Policy / Permission / Verification 先于复杂模型能力

模型越强，越容易生成看似合理但越权、不可验证或不可审计的路径。模型可以很早参与 Intent 和 Plan 候选生成，但不能绕过：

```text
Schema Validation
→ Policy Check
→ Permission Check
→ Verification Requirement Binding
→ User-visible State
→ Audit Event
```

### 6.5 审计和验证从第一天存在，而不是后补

Verification Event 和 Audit Event 必须在第一条任务链路中存在。后补审计只能记录“系统声称做了什么”，很难证明“实际做了什么、没做什么”。

---

## 7. 研发验证路线

### 7.1 阶段门总览

建议把研发分为六个阶段门，而不是按 App 数量推进：

```text
Gate 0：总纲一致性门
Gate 1：Runtime 骨架门
Gate 2：低风险单步执行门
Gate 3：跨 App 草稿闭环门
Gate 4：信任治理门
Gate 5：能力扩展门
```

每个阶段门都必须用证据通过，而不是用功能演示通过。

### 7.2 Gate 0：总纲一致性门

目标：确保任何研发方案没有偏离 Janus 定位。

必须产出：

- 模块所属架构层说明；
- 触碰核心对象说明；
- 权限、风险、数据、外部影响说明；
- 测试方式说明；
- 反模式检查。

通过标准：每个方案都能回答它服务哪条总纲原则、依赖哪些技术原则、触碰哪些核心对象、是否新增权限 / 风险 / 数据 / 外部影响、如何被测试、可能造成哪种反模式。

### 7.3 Gate 1：Runtime 骨架门

目标：建立不依赖真实 App 自动化的 Janus 核心运行纪律。

| 能力 | 通过标准 |
|---|---|
| Task 创建 | Intent 能创建 Task |
| Plan / Step | Plan 可展示或折叠为 Action Trace，Step 有风险和验证要求 |
| PolicyDecision | Allow / Confirm / Block / Deny / Downgrade 可表达 |
| Task State Machine | Created → Planning → BoundaryChecking → Executing → Verifying 等状态可流转 |
| InterruptToken | Stop / Takeover 可失效队列 |
| AuditEvent | 已做、未做、状态变化可记录 |

不应做：真实发送、上传、删除；长期记忆；多任务并发；第三方扩展。

### 7.4 Gate 2：低风险单步执行门

目标：验证 Janus 可以执行单个低风险 Step，并产生验证证据。

推荐任务：

```text
打开一个 App；
切换窗口；
读取当前窗口标题；
在本地测试输入框中输入草稿；
验证输入框内容；
停止并记录未继续执行。
```

| 能力 | 通过标准 |
|---|---|
| ExecutableStep | 缺少权限、风险或验证要求时不执行 |
| Execution | 一次只执行一个 Step |
| Verification | 执行后生成 VerificationEvent |
| Interrupt | 执行前后均可 Stop |
| Audit | Timeline 能复盘已做与未做 |

### 7.5 Gate 3：跨 App 草稿闭环门

目标：验证 Janus 的本质链路：从当前上下文出发，快速完成跨 App 低风险准备工作，并在高风险或外部影响动作前停下。

微信消息草稿只是代表性验证样本，用来验证入口、跨 App 执行、草稿准备、确认缝和复盘闭环；它不能反向限定 Janus 的产品结构。Janus 的通用对象是目标、环境、对象、动作、风险、验证和结果。

推荐验证链路：

```text
用户正在 Terminal；
长按 Janus Entry 说出“打开微信给张三写条消息，说我十分钟后到，发送前确认”；
松开提交 Intent；
Janus 形成理解和 Plan；低风险清晰任务可在执行态 Action Trace 中展示，而不是要求用户先批准；
Janus Surface 展开并直接推进机械路径；
Janus 打开或切换微信；
Janus 搜索联系人；
Janus 创建消息草稿；
Janus 验证草稿存在；
Janus 验证消息未发送；
Janus 停在确认前 / 等待确认状态；
Janus 生成 Timeline。
```

通过标准：

1. 当前 Terminal 不是任务边界；
2. 目标 App 能作为 EnvironmentRef；
3. 搜索、选择、输入、草稿均映射为通用原语；
4. 长按语音路径能按下采集、松开提交，并创建可追溯 Intent；
5. 低风险准备路径能自动推进到准备结果 / 等待确认状态；
6. Submit 被识别为 L4；
7. 没有用户确认时不会发送；
8. NegativeVerified 能证明未发送；
9. Stop / Takeover 可中断；
10. Audit Timeline 记录已做、未做、理解、Plan、验证和确认边界；
11. LearningCandidate 不自动保存为 Memory。

失败标准：

```text
未经确认发送；
找错联系人仍继续；
无法验证草稿却继续；
长按语音 release 后没有形成可追溯 Intent；
快速路径跳过理解、Plan、风险判断、验证或确认边界记录；
用户停止后继续；
把发送前确认当作长期授权；
把联系人映射静默保存为 Memory；
Timeline 只显示成功，不显示未做动作。
```

### 7.6 Gate 4：信任治理门

目标：验证 Delegation Center 的托管、权限、记忆三类治理视图，以及长期授权、记忆候选、审计历史和信任恢复证据能治理 Janus 行动权。

| 能力 | 通过标准 |
|---|---|
| 授权边界 | 可查看、撤销、过期、缩小授权 |
| 记忆候选 | 候选可解释、拒绝、编辑、过期 |
| 历史复盘 | 成功、失败、安全停下、接管均可查看 |
| 信任恢复 | 风险事件后能力降级并说明原因 |
| 数据删除 | Memory 可删除，后续不可使用 |

### 7.7 Gate 5：能力扩展门

目标：在核心信任闭环成立后，扩展更多 App、更多任务类型、更复杂模型、更强适配器和可能的第三方能力。

只有 Gate 1-4 稳定后，才应扩展多任务并发、第三方 Adapter、长期 DelegationRule、App 自探索、更复杂模型规划、企业组织策略和扩展生态。

扩展不得破坏三条控制线：

```text
用户控制线；
风险授权线；
证据复盘线。
```

---

## 8. 推荐研发顺序

| 优先级 | 研发主题 | 范围 | 原因 |
|---|---|---|---|
| 1 | 可信 Runtime 底座 | Task / Intent / Plan / Step、Task State Machine、Permission / Risk / PolicyDecision、VerificationRequirement / Event、AuditEvent / Timeline、InterruptToken、Local Event Bus、SafeMode | 这些对象定义 Janus 是否仍然是 Janus |
| 2 | 低风险本地执行器 | Execution Engine、通用操作原语、低风险 App 打开 / 切换 / 输入、验证信号读取、受限 Adapter、失败与中断处理 | 先证明低风险机械路径可快速、可验证地完成 |
| 3 | 模型网关与规划能力 | Intent Model、Planning Model、Object Resolver、Output Schema Validation、Prompt / Policy 分离、ModelReasoningEvent | 模型必须进入 Runtime，而不是绕过 Runtime |
| 4 | 跨 App 受托任务闭环 | 创建消息草稿、查找并打开文档、整理网页到草稿、本地文件选择与预览、低风险信息搬运 | 重点不是覆盖场景，而是验证链路 |
| 5 | 长期治理能力 | Delegation Center、Memory & Learning、Recovery Manager、Capability Governance、Observability、版本发布治理 | 只有核心信任闭环成立后才有长期治理基础 |

---

## 9. 后续必须派生的研发文档

根据本报告，建议按以下顺序派生研发文档。它们是具体实现前的研发包，不是本文需要展开的运行时定义。

| 顺序 | 文档 / 任务包 | 主要内容 | 用途 |
|---|---|---|---|
| 1 | Janus Runtime 对象与状态机规范 | Intent、Task、Plan、Step、ExecutableStep、Task State Machine、Authorization State Machine、InterruptToken、状态转换事件 | 作为所有模块共同协议 |
| 2 | 权限、风险与确认门策略规范 | L0-L5、R 类风险事件、PolicyDecision、ConfirmationPayload、PermissionGrant、DelegationRule、撤销、过期、降级 | 防止能力滑坡为权力 |
| 3 | Verification Event 与 Audit Event 数据规范 | VerificationRequirement、VerificationEvent、NegativeVerified、AuditEvent、Timeline、EvidenceRef、隐私最小化保存 | 建立证据链和复盘链 |
| 4 | macOS Runtime 技术验证方案 | Janus Entry、全局快捷键、胶囊状态、当前 App / Window、Accessibility、Screen / OCR、本地权限、中断控制 | 验证 macOS 桌面行动层是否成立 |
| 5 | App Adapter / Capability Manifest 规范 | Environment、Object、Action、State、Verification Signal、AdapterStatus、CapabilityCandidate、能力降级 | 防止逐 App 重脚本化 |
| 6 | Model Gateway 与模型治理规范 | 模型角色、输入数据边界、输出 schema、工具权限、模型审计、失败降级、本地 / 云端模型边界 | 防止模型成为权力中心 |
| 7 | Privacy Boundary 与安全沙箱规范 | D0-D5 数据分级、PrivacyDecision、Permission Sandbox、Secure Storage、SafeMode、Extension Sandbox | 建立长期可信的数据和执行边界 |
| 8 | 派生产品切片技术方案 | 选择一个跨 App、低到中风险、含确认和验证闭环的任务；明确范围、非目标、验证标准、失败标准 | 进入实际开发前的技术切片设计 |

---

## 10. 技术风险清单

### 10.1 一票否决风险

以下风险一旦出现，应停止相关能力继续扩展：

```text
未经确认发送、上传、提交、删除；
用户 Stop / Takeover 后继续执行；
无法验证结果却标记 Completed；
静默保存联系人映射、文件路径、组织信息；
用一次确认生成长期授权；
Adapter 绕过 Policy 或 Audit；
模型输出直接驱动高风险动作；
第三方扩展绕过 Sandbox；
高敏数据未经边界检查发送到云端。
```

### 10.2 高概率工程风险

| 风险 | 影响 | 应对 |
|---|---|---|
| Accessibility 信号不稳定 | 执行和验证失败 | 结构化优先，多信号验证，失败停下 |
| UI 焦点漂移 | 点错对象 | 每步前后验证焦点和对象 |
| OCR 误识别 | 错误判断状态 | OCR 只作 Medium / Weak 信号 |
| LLM 虚构能力 | 生成不可执行计划 | CapabilityManifest + Schema Validation |
| 用户目标歧义 | 做错任务 | NeedsClarification / Blocked |
| 权限粒度过粗 | 系统权限过大 | 内部 Permission Sandbox |
| 审计保存过多 | 隐私风险 | EvidenceRef + 最小证据 |
| 适配器膨胀 | 退化为脚本集合 | 通用原语 + 薄适配职责 |

### 10.3 指标反模式风险

后续研发不应使用以下指标作为最高目标：

```text
自动完成率；
任务覆盖 App 数量；
平均执行速度；
模型一次性成功率；
用户少点了几次确认。
```

更高优先级指标应是：

```text
越权事件数；
未确认外部影响数；
验证缺失率；
Stop 后继续执行率；
审计完整率；
高风险动作拦截率；
Memory 静默保存数；
SafeStop 正确率；
低风险路径到达可检查结果的成功率；
外部影响边界前硬停成功率。
```

---

## 11. 后续研发流程建议

### 11.1 每个研发任务必须经过六问

任何模块、能力、切片进入开发前，必须回答：

```text
1. 它服务哪条 Janus 总纲原则？
2. 它触碰哪些核心对象？
3. 它新增了什么权限、风险、数据或外部影响？
4. 它的验证信号是什么？
5. 用户如何停止、接管、撤销和复盘？
6. 如果它失败，如何安全停下？
```

### 11.2 每个 PRD 必须附技术边界表

| 项 | 内容 |
|---|---|
| Task 类型 | 该需求创建什么 Task |
| Action 原语 | 涉及哪些通用 Action |
| Risk Level | 最高风险等级 |
| Permission | 需要什么授权 |
| Confirmation | 哪些动作必须确认 |
| Verification | 每步如何验证 |
| Audit | 记录什么已做 / 未做 |
| Memory | 是否产生学习候选 |
| Stop / Takeover | 如何中断 |
| Non-goals | 明确不做什么 |

### 11.3 每个技术方案必须附反模式检查

至少检查：

```text
是否变成 App 专属脚本？
是否让模型直接操作 UI？
是否把系统权限当作用户授权？
是否把一次确认当作长期授权？
是否用 OCR / 视觉相似驱动高风险动作？
是否忽略 Stop / Takeover？
是否只记录成功不记录失败？
是否静默保存 Memory？
```

### 11.4 每个测试计划必须包含失败路径

Janus 的测试不能只测成功路径。必须包含对象歧义、权限不足、验证不足、用户停止、用户接管、UI 变化、App 不可用、模型输出非法、Adapter 能力失效、审计不可用、Privacy Boundary 阻断和 SafeMode 触发。

---

## 12. 推荐的近期研发任务包

| Package | 目标 | 范围 | 验收 |
|---|---|---|---|
| A：Runtime Kernel | 建立 Janus 最小可信内核 | Intent / Task / Plan / Step、Task State Machine、Permission / Risk / PolicyDecision、InterruptToken、VerificationRequirement、AuditEvent、Event Bus | 不接真实 App，也能完整模拟 Task 生命周期、风险拦截、中断和审计 |
| B：macOS Shell | 建立 Janus Entry 和用户控制锚点 | Janus Entry、快捷键、胶囊、Janus Surface 状态投影、当前 App / Window 识别、Stop / Pause / Takeover UI | 任意 App 中可唤起 Janus；空闲不采集；执行中可停止 |
| C：Evidence Engine | 建立验证和审计证据链 | VerificationEvent、EvidenceRef、NegativeVerified、Timeline、AuditEvent、FailureTransition | 能证明 Step 成功、失败、不确定、未产生外部影响 |
| D：Low-risk Executor | 执行低风险通用原语 | Open、Switch、Observe、Input、Draft、Preview、结构化信号读取、受限 fallback、ExecutableStep | 一次一步、每步验证、可中断、可审计 |
| E：Policy & Permission | 建立真实可执行边界 | L0-L5、R 类风险、PermissionGrant、ConfirmationPayload、Permission Sandbox、Revoke / Expire / Consume | 未经确认的 L4/L5 动作不能执行 |
| F：Model Gateway | 让模型进入受控 Runtime | IntentCandidate、PlanCandidate、ObjectCandidate、RiskReasoning、Output Schema Validation、Privacy Boundary、ModelReasoningEvent | 模型不能直接产生 ExecutableStep；非法输出被拦截 |

---

## 13. 研发决策建议

### 13.1 应立即坚持的决策

```text
1. 使用 Task 而不是 Chat 作为核心运行对象；
2. 使用 Step 而不是脚本作为最小执行单位；
3. 使用 Policy / Permission / Verification 约束模型和执行器；
4. 使用通用操作原语，而不是逐 App 场景脚本；
5. 使用 Verification Event 和 Audit Event 建立证据链；
6. 使用 Permission Sandbox 弥补系统权限粒度过粗；
7. 使用 SafeMode 处理核心安全模块不可用；
8. 使用 Delegation Center 治理托管任务、权限和记忆等长期状态。
```

### 13.2 暂不应过早决定的事项

完整商业化架构、全部 App 覆盖范围、第三方扩展生态开放方式、企业组织后台完整策略、多任务并发开放时间、通用 App 自探索方案、所有模型供应商和最终模型组合，都应在核心信任闭环验证后再决策。

### 13.3 可以并行探索但不得阻塞主线的事项

不同 App 的 Accessibility 质量调研、Browser Extension 与 DOM 控制方案、本地模型和云端模型能力对比、OCR / Vision 可靠性、安全存储和审计最小证据策略、用户确认卡和 Janus Surface 交互细节，可以并行探索，但不得替代主线阶段门。

---

## 14. 最终结论

Janus 的核心依赖技术不是某一个单点能力，而是一组必须共同成立的运行纪律：

```text
Task 化目标；
Step 化执行；
Policy 化边界；
Permission 化授权；
Verification 化结果；
Interrupt 化控制；
Audit 化复盘；
Memory 化学习授权；
Sandbox 化能力治理；
Gateway 化模型使用。
```

从技术论证看，Janus 可以进入后续研发流程，但研发起点不应是“做一个能操作电脑的模型助手”，而应是：

```text
先实现一个小而硬的 Janus Runtime Kernel，
证明它能在低风险、跨 App、可中断、可验证、可复盘的任务中快速完成有用准备工作，
并且不会在外部影响边界前越权。
```

只有当这个内核成立，Janus 才有资格继续扩展 App 覆盖、模型能力、长期记忆、主动性、多任务并发和第三方生态。

最终研发原则应保持：

```text
少做一点，但每一步都可授权；
快做低风险机械路径，但每一步都可验证；
窄做一点，但每一次能力扩张都不牺牲用户主权。
```

这才符合三份总纲共同定义的 Janus：

```text
人的受托数字行动入口。
```

---

## 附录 A：外部技术验证入口

以下外部技术方向应在进入具体实现文档时逐项查阅官方文档并实证验证。当前报告只做架构级论证，不把这些链接视为最终 API 设计结论。

- [Apple Accessibility / AXUIElement](https://developer.apple.com/documentation/applicationservices/axuielement)：用于评估 macOS UI 结构读取与可访问性操作边界。
- [Apple ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit)：用于评估窗口 / 屏幕采集、验证信号与隐私权限边界。
- [Apple Vision Text Recognition](https://developer.apple.com/documentation/vision/recognizing-text-in-images)：用于评估 OCR 作为中低等级验证信号的可用性。
- [Apple App Sandbox](https://developer.apple.com/documentation/security/app_sandbox) / [Hardened Runtime](https://developer.apple.com/documentation/security/hardened-runtime)：用于评估本地安全边界、权限和分发约束。
- [Apple Keychain](https://developer.apple.com/documentation/security/keychain_services) / [CryptoKit](https://developer.apple.com/documentation/cryptokit)：用于评估本地敏感状态、授权和记忆的安全存储。

---

## 附录 B：报告使用方式

后续团队使用本文时，不应把它当成实现任务列表，而应当成研发阶段门说明。

每当一个具体技术方案被提出，应检查：

```text
它是否仍然服务“受托数字行动入口”？
它是否把能力误当成权力？
它是否有验证信号？
它是否能被用户停止？
它是否能被审计复盘？
它是否会静默学习或长期观察？
它是否能在失败时安全停下？
```

如果答案不清楚，说明该方案还不能进入实现。
