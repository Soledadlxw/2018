# CPU 微架构验证专家学习路线（季度执行版）

> 面向背景：已有 DV/验证经验  
> 目标方向：打穿 ISA + 服务器核微架构，达到能看懂处理器微架构设计并指导验证工作  
> 主 ISA：AArch64（ARM 服务器核）深挖，RISC-V 作对照（若主做 RISC-V，两边对调即可）  
> 周期：4 个季度（约 1 年）

---

## 1. 目标画像与年度验收

### 1.1 能力目标

- 看到流水线 / 乱序 / 缓存 / 一致性 / 分支预测等设计，能判断正确性边界、竞态、异常路径与验证盲区
- 能把微架构行为映射为：覆盖点、断言、激励策略、参考模型、调试方法
- 能指导验证：哪些必须证伪、哪些靠形式化、哪些靠随机 + 定向、如何定义收敛

### 1.2 Y1 结束验收标准

1. 独立交付一份服务器核「微架构风险清单 + 验证总纲」
2. 对 Frontend / Rename-ROB / LSUnit / MMU-虚拟化 / Coherence 任一子系统给出可执行验证方案
3. 讲清服务器核关键 ISA 语义：异常/特权、虚拟化、内存模型、原子、中断
4. 用「覆盖 + 断言 + 差分 + litmus/多核」解释收敛，而不是只报 coverage 数字

### 1.3 能力自评门槛（持续对照）

| 门槛 | 标准 |
|------|------|
| 看图能讲 | 任意公开微架构框图，30 分钟讲清数据流、控制流、恢复路径 |
| 看设计能抓风险 | 指出冲刷 × 访存 × 中断等高危交互 |
| 能写验证策略 | 对 LSUnit / Rename / Coherence 给出可执行方案 |
| 能定收敛 | 解释为何覆盖率高仍可能漏致命 bug |
| 能指导他人 | 拆分模块级/核级任务并设优先级 |

---

## 2. 总路线图

```text
Q1  ISA 打穿 + 服务器核语义底座
      ↓
Q2  乱序微架构读图 + 模块验证策略
      ↓
Q3  LSUnit / 内存序 / 一致性 / 虚拟化（主战场）
      ↓
Q4  核级·系统级收敛 + 验证总纲（可指导团队）
```

### 每周固定节奏（四季度通用）

| 时间盒 | 内容 |
|--------|------|
| 6–8h | 手册 / 书 / 论文精读（必须输出笔记） |
| 6–10h | 动手：场景、断言、TB、差分、litmus、开源核解剖 |
| 2h | 维护风险清单 / 覆盖模型 / bug taxonomy |
| 1h | 复盘：本周新理解如何改变验证策略 |

> 已有 DV 经验：少刷 UVM 教程，多刷 **ISA × 微架构 × 检查点映射**。

---

## 3. 书单与规范总表

### 3.1 P0｜主轴书籍（建议购置）

| # | 书名 | 作者 | 用途 |
|---|------|------|------|
| 1 | *Computer Architecture: A Quantitative Approach*（第 6 版） | John L. Hennessy, David A. Patterson | 服务器核微架构、存储层次、多处理器、虚拟化总纲 |
| 2 | *Modern Processor Design: Fundamentals of Superscalar Processors* | John Paul Shen, Mikko H. Lipasti | 乱序超标量（Rename / ROB / Issue / LSU）精读 |
| 3 | *A Primer on Memory Consistency and Cache Coherence*（第 2 版） | Vijay Nagarajan, Daniel J. Sorin, Mark D. Hill, David A. Wood | 内存模型 + 一致性协议，服务器核必读 |
| 4 | *SystemVerilog for Verification*（第 3 版） | Chris Spear, Greg Tumbush | DV 查漏补缺；当手册用，不从头通读 |

### 3.2 P0｜规范手册（当“法律”读）

| # | 文档 | 用途 |
|---|------|------|
| 5 | *Arm Architecture Reference Manual for A-profile architecture*（ARM ARM / DDI0487，跟目标版本） | AArch64 ISA、异常、VMSA、虚拟化、系统指令 |
| 6 | ARM Memory Model 相关章 + Arm 官方 memory model 说明 | 弱序、barrier、原子、可观察性 |
| 7 | *RISC-V Unprivileged ISA Spec* + *RISC-V Privileged Spec* | 对照：特权、虚拟内存、hypervisor |
| 8 | *Arm Generic Interrupt Controller Architecture Specification*（GICv3 / GICv4） | 服务器中断 / 虚拟中断 |

### 3.3 P1｜强烈建议

| # | 书名 / 材料 | 作者 / 来源 | 用途 |
|---|-------------|-------------|------|
| 9 | *Computer Organization and Design*（RISC-V Edition 或 ARM Edition） | Patterson, Hennessy | Q1 补齐流水线 / 异常 / VM 直觉 |
| 10 | *Digital Design and Computer Architecture*（RISC-V Edition） | Sarah Harris, David Harris | 数字逻辑 / 简单流水偏弱时补 |
| 11 | Arm Neoverse N1 / V1 / N2 等 *Technical Overview* / Whitepaper | Arm 官网 PDF | 服务器核真实微架构读图 |
| 12 | Hot Chips Neoverse / AMD / Intel server core 演讲 | Hot Chips 等 | Q2–Q4 读图与风险迁移 |

### 3.4 P2｜加分

| # | 书名 / 材料 | 作者 | 用途 |
|---|-------------|------|------|
| 13 | *Modern Operating Systems*（第 4/5 版） | Andrew S. Tanenbaum, Herbert Bos | 虚拟内存、中断、多核同步（软件如何折磨 CPU） |
| 14 | *Operating Systems: Three Easy Pieces*（OSTEP） | Remzi & Andrea Arpaci-Dusseau | 页表、并发、锁；系统级场景设计 |
| 15 | Formal Verification 实用入门（如 *Finding Your Way Through Formal Verification* 或团队 cookbook） | — | Q4 属性划分 |
| 16 | *Writing Testbenches using SystemVerilog* | Janick Bergeron | 与 Spear 二选一深翻 |
| 17 | *The Art of Verification with SystemVerilog Assertions* | Eduard Cerny 等 | 微架构断言写法 |

### 3.5 最小采购清单（8 样跑完全年）

1. H&P《Computer Architecture: A Quantitative Approach》(6th)
2. Shen & Lipasti《Modern Processor Design》
3. 《A Primer on Memory Consistency and Cache Coherence》(2nd)
4. Spear《SystemVerilog for Verification》(3rd)
5. Arm ARM（A-profile）PDF
6. RISC-V Privileged Spec PDF
7. GICv3/v4 Architecture Spec PDF
8. Arm Neoverse 一代 Technical Overview + 对应 Hot Chips slides

### 3.6 全年阅读顺序（防乱读）

```text
Q1: ARM ARM(异常/VM/内存模型导论) → H&P(存储/多处理器导论) → RISC-V 特权对照
Q2: Shen/Lipasti 主线 → H&P(ILP/投机) → Neoverse 白皮书读图 → Spear/SVA 按需
Q3: Primer on MCC 全书 → ARM ARM 二刷(VMSA/virt/atomics) → GIC → litmus 论文组
Q4: 拆验证计划范文 → formal 选读 → 回看 H&P/Shen 做总纲
```

---

## 4. 季度执行计划

---

### Q1｜ISA 打穿 + 服务器核语义底座

**季度目标：** 把 ISA 从“会查手册”提升到“能定义检查点与非法行为”；补齐服务器核特有语义。

#### 学什么

- AArch64 用户态与系统态：异常级别、异常入口/返回、寄存器与系统寄存器模型
- 虚拟化：EL2、Stage-2 翻译、陷阱转发、虚拟中断基础
- 内存模型：observability、barriers、atomics、acquire/release
- 服务器相关：GIC 概念、多核启动、TLB/ASID/VMID、缓存维护指令语义
- 对照：RISC-V 特权级、Sv39/Sv48、hypervisor 扩展、RVWMO 概念

#### 读什么

| 优先级 | 材料 | 读法 |
|--------|------|------|
| 必须 | ARM ARM | Exception / EL、VMSA、Virtualization 导论、Memory model / barriers / atomics |
| 必须 | H&P 6th | 存储层次、虚拟内存、多处理器基础 |
| 必须 | RISC-V Privileged Spec | Machine/Supervisor/Hypervisor、虚拟内存（对照笔记） |
| 选读 | COD（RISC-V/ARM Edition） | 异常、流水线、VM 三章 |
| 选读 | OSTEP | Virtual memory + Concurrency |
| 暂缓 | Shen/Lipasti | 可翻目录，精读放 Q2 |
| 预读 | Primer on MCC | 仅第 1–3 章 |

#### 练什么

- 每周做 **ISA 语义卡片**：选 5 条服务器核高危指令/系统行为（如 `ERET`、`TLBI`、`DC CIVAC`、原子对、`AT`），写出架构状态变化、副作用、微架构交互点、验证检查点
- 用 Spike / QEMU / Fast Model 做：异常嵌套、页表 walk 失败、barrier 前后可观察序小实验
- 写 ≥20 个 **ISA-level checker 伪代码**（提交点比对哪些架构状态）

#### 产出什么

| 产出物 | 说明 |
|--------|------|
| `ISA_Server_Notes.md` | 异常 / VM / 内存模型 / 原子 / 缓存维护 五章精炼笔记 |
| `ISA_Checker_Checklist_v1` | ≥50 条架构检查点 |
| `Server_ISA_Risk_Top20` | 从 ISA 语义出发的验证风险一页纸 |

#### Q1 出门标准

给一句系统寄存器 / 异常 / 页错误场景，15 分钟内说出：架构期望、常见微架构坑、如何在 TB 里抓。

---

### Q2｜乱序微架构读图 + 模块验证策略

**季度目标：** 从“会搭验证环境”升级到“看懂服务器核主干微架构，并能写模块级验证策略”。

#### 学什么

- 乱序主干：Fetch/BP、Decode、Rename、Dispatch、Issue、EX、WB、Commit
- ROB / 物理寄存器 / 恢复、精确异常与冲刷
- 服务器核放大点：深窗口、SMT（若有）、大 TLB、多级缓存、复杂前端
- 将已有 UVM 经验迁移到：微架构不变量、flush 安全性、资源泄漏、tag 一致性

#### 读什么

| 优先级 | 材料 | 读法 |
|--------|------|------|
| 必须 | Shen & Lipasti《Modern Processor Design》 | 超标量、Rename/ROB/精确中断、执行核心、内存相关章 |
| 必须 | H&P 6th | ILP、投机执行、存储系统 |
| 必须 | Arm Neoverse Whitepaper / Hot Chips | 至少吃透一代结构图 |
| 选读 | Harris《Digital Design and Computer Architecture》 | 经典流水线冒险章（过渡用） |
| 选读 | BOOM 文档 或 香山设计文档 | 读结构练不变量，不要求同 ISA |
| 按需 | Spear《SV for Verification》 | coverage / stimulus / scoreboard |
| 按需 | Cerny《…SystemVerilog Assertions》或常用 SVA 书 | 协议/不变量断言模式 |

#### 练什么

- 选一个开源乱序核做 **结构解剖**：画出 fetch→commit 数据/控制流；为 Rename、ROB、RS、BP 各写 10 条不变量
- 用现有 DV 技能完成 **至少 2 个模块级迷你验证**（建议：Rename+ROB 恢复；Issue 或简单 LSU 片段），含随机 + 定向冲刷 + 断言 + 覆盖
- 每周一次 **RTL/框图评审演练**：以验证负责人视角写出 10 个 review 问题

#### 产出什么

| 产出物 | 说明 |
|--------|------|
| `UO_Pipeline_Map` | 乱序主干图 + 每级接口契约 |
| `Module_VP_RenameROB.md` | Rename/ROB 模块验证计划 |
| `Module_VP_IssueOrLSU.md` | Issue 或 LSU 模块验证计划 |
| `Assertion_Catalog_v1` | ≥30 条微架构断言（协议/资源/恢复/排序） |

#### Q2 出门标准

拿到一张服务器核公开框图，30 分钟讲清：提交点、恢复路径、LSU 与 ROB 握手、验证应卡在哪些接口。

---

### Q3｜LSU、内存序、一致性、虚拟化

**季度目标：** 拿下服务器核验证主战场，形成可指导团队的专项方案。

#### 学什么

- LSUnit：load/store buffer、转发、推测 load、replay/squash、设备内存 vs 可缓存内存
- 内存模型验证：单核序、多核 litmus、barrier/原子、弱序陷阱
- Cache coherence：MESI/MOESI 或目录；MSHR、RFO、分享降级、原子与排他
- MMU + 虚拟化：多级页表、S1/S2、TLBI 广播语义、权限/属性、并发改页表
- RAS（按需）：poison、错误上报对架构状态的影响（功能视角）

#### 读什么

| 优先级 | 材料 | 读法 |
|--------|------|------|
| 必须 | 《A Primer on Memory Consistency and Cache Coherence》(2nd) | **全书** |
| 必须 | ARM ARM 二刷 | Memory model、Atomics、VMSA、TLBI/cache maintenance、Virtualization+S2 |
| 必须 | GIC Architecture Spec（v3/v4） | distributor/redistributor、vINT、LPI 基础（按产品裁剪） |
| 必须 | H&P 6th | multiprocessors & coherence 再读一轮，对照 Primer |
| 对照 | RISC-V RVWMO 章节 | 弱序对照 |
| 可选对照 | Intel SDM Vol.3 memory ordering | 仅内存序章，不展开全手册 |

#### 论文 / 工具材料（精读 6–8 篇即可）

| 主题 | 推荐 |
|------|------|
| Litmus / 内存模型测试 | Alglave et al., *Herding Cats*；diy/herd 工具文档 |
| 弱序与 litmus | Alglave, Maranget, Tautschnig 等相关工作 |
| 一致性验证 | coherence verification 案例（ISCA/MICRO/FMCAD 检索） |
| 微架构内存 | store buffer、load speculation、memory disambiguation 经典论文 2–3 篇 |

#### 练什么

- 建立 **litmus + 定向场景库（≥40）**：MP、SB、LB、原子、错误配对 barrier、别名、I-cache 一致性等
- 设计并尽量实现 **提交点差分**：RTL/核 vs ISS（GPRs、关键系统寄存器、fault syndrome）
- 落地 LSU 验证方案至少一种：转发 scoreboard / store-load 顺序检查 / fake coherence BFM
- 做 2 次故障注入：错误转发、漏 squash、TLBI 未同步，检验检测网是否报警

#### 产出什么

| 产出物 | 说明 |
|--------|------|
| `Server_Memsys_Verification_Playbook.md` | LSU / MMU / Coherence / Virtualization 四分册 |
| `Litmus_Catalog_v1` | ≥40 场景 + 覆盖模型草案 |
| `Bug_Taxonomy_Memsys` | ≥40 条按根因分类的 bug 库 |

#### Q3 出门标准

能主持 LSUnit 或一致性验证评审：指出 Top 风险、缺的覆盖、该补的断言/场景，并给出优先级。

---

### Q4｜核级 / 系统级收敛 + 可指导验证

**季度目标：** 从专项专家变成能写总纲、定 signoff、带人拆任务的验证负责人视角。

#### 学什么

- 核级验证架构：ISS 差分、trace、repro、覆盖闭环、回归分层
- 系统级：多核启动、OS bring-up 激励、中断风暴、设备内存、锁与同步密集负载
- 形式化补强：关键控制 FSM、ROB 恢复、通道协议的属性划分
- 验证管理：风险驱动计划、人力切分、周收敛指标、escape 分析

#### 读什么

| 优先级 | 材料 | 读法 |
|--------|------|------|
| 必须 | 汇总 Q1–Q3 笔记 | 重读汇编，不再追求新大部头 |
| 必须 | Spear《SV for Verification》 | regression、coverage closure、可重用 TB |
| 必须 | 2 份 CPU Verification Plan / DVCon 论文 | 当范文精拆 |
| 选读 | Formal 入门 + 1–2 篇 CPU control/coherence formal 案例 | 属性划分 |
| 选读 | Tanenbaum《Modern Operating Systems》相关章 | 系统级场景 |
| 选读 | Hot Chips / DVCon server CPU verification 分享 3–5 场 | 工业现实 |

#### 练什么

- 完整撰写 **服务器核验证总纲（Verification Plan）**
- 「虚拟带队」：拆成 6–8 个模块/场景 owner 任务，含入口/出口标准
- 选一个子系统补齐到签核草案：覆盖目标、必过断言、必过定向、随机 saturate、遗留风险
- 至少 1 次端到端演练：随机包 + 差分失败 → root cause 到微架构机制（写复盘）；全年累计 ≥3 个复盘案例

#### 产出什么

| 产出物 | 说明 |
|--------|------|
| `Server_Core_VP_v1.md` | 15–25 页：微架构摘要、风险热图、分层、激励、覆盖、工具、signoff |
| `Coverage_Model_Server_Core_v1` | 架构覆盖 + 微架构覆盖 |
| `Signoff_Gate_Checklist` | 带出/签核门槛 |
| 作品集打包 | Q1–Q4 全部文档（“能指导验证”的证据） |

#### Q4 出门标准

面对新服务器核，1–2 周内产出可评审验证总纲与 Top 风险，并回答：为何这些场景过了可带出，哪些还必须补形式化/硅后。

---

## 5. 季度产出总览

| 季度 | 关键产出 | 能力跃迁 |
|------|----------|----------|
| Q1 | ISA 检查点表 + Server ISA Top20 风险 | 架构语义可验证化 |
| Q2 | 乱序图解 + 2 份模块 VP + 断言目录 | 微架构可读 + 模块策略 |
| Q3 | Memsys/VM/Coherence Playbook + litmus 库 | 服务器核主战场可指导 |
| Q4 | 核级验证总纲 + 覆盖模型 + signoff gate | 验证负责人视角 |

---

## 6. 验证方法论锚点（全年贯穿）

### 6.1 分层验证地图

1. 单元/模块级：重命名、RS、ROB、LSU、TLB、分支预测
2. 流水线集成：冲刷、stall、回压、多拍交互
3. 核级功能：自比对、参考模型、随机指令流
4. 系统级：OS boot、多核、设备、中断风暴
5. 形式化：关键控制 FSM、协议、不变量
6. 硅前专项：内存模型、安全/侧信道相关功能约束（按产品）

### 6.2 技术栈（在已有 DV 经验上补齐）

- UVM（或团队等价框架）+ 受限随机
- 功能覆盖 / 交叉覆盖 / **微架构覆盖**
- SVA：协议、不变量、非法排除
- 参考模型：ISS（Spike/QEMU/自研）差分
- 形式化：BMC/证明关键控制属性
- 调试：波形方法论、trace、commit log、重放

### 6.3 指导验证时的检查清单

对任意新核，争取较快产出：

- [ ] 微架构风险清单（Top 20 bug 类）
- [ ] 验证分层与所有权切分
- [ ] 覆盖模型初稿（含微架构事件）
- [ ] 激励策略：随机 / 定向 / 压力比例
- [ ] 收敛判据与 signoff gate

---

## 7. 风险与纠偏

| 现象 | 纠偏 |
|------|------|
| Q1 仍在“抄手册” | 强制「每条语义 → 检查点/伪代码」 |
| Q2 沉迷开源核细节 | 只服务不变量与验证策略，不变成 RTL 开发 |
| Q3 只跑通 litmus | 必须补微架构机制级覆盖（转发、squash、TLBI、MSHR） |
| Q4 文档全但无失败复盘 | 补 ≥3 个端到端 debug 案例 |
| 同时深挖 x86+ARM+RISC-V | 先一条 ISA 打穿，再迁移 |
| 覆盖率数字游戏 | 用风险清单与 bug taxonomy 校正优先级 |

---

## 8. 启动包（第 1 周）

1. 选定主 ISA（AArch64 或 RISC-V）并固定手册版本号
2. 精读：异常级别/特权 + 页表 + 内存模型导论
3. 产出：10 条 ISA checker + Top10 服务器核 ISA 风险
4. 预约 Q2 用的开源核仓库与文档（不提前深潜）
5. 备齐第 3.5 节最小采购清单中的 PDF/书籍

---

## 9. 附录：常见弯路

- 只学 UVM 语法，不学微架构 → 只能写 TB，不能指导
- 只看架构课，不动手差分/断言 → 理解浮在 PPT
- 过早钻性能调优/PPA，弱化功能正确性与恢复路径
- 忽略内存模型与多核竞态
- 收藏大量资料却无每周交付物

---

## 文档修订

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-09-11 | 首版：季度计划 + 具体书单 + 产出与验收 |

---

*本文档用于个人/团队培养路径规划；执行时按实际产品 ISA 与团队框架裁剪，但不要砍掉 Q1 ISA 检查点化与 Q3 内存子系统主战场。*
