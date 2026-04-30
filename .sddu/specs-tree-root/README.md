# ETD 规范树根目录

## 目录简介

本目录是 **ETD (Expert Tree Design)** 项目的规范树根目录，包含完整的规范定义、需求挖掘、技术规划文档，以及三个子 Feature 的规范树。

**项目定位**：
- ETD 是一套领域无关的方法论
- 动态构建专业协作树
- AI 执行 + 人类看护，谋定而后动

---

## 目录结构

```
.sddu/specs-tree-root/
├── README.md                              # 本文件 - 规范树目录导航
├── spec.md                                 # 产品规范文档
├── discovery.md                            # 需求挖掘报告
├── plan.md                                 # 技术规划文档
├── spec.json                               # 规范元数据
├── state.json                              # 状态文件
├── specs-tree-expert-node-registry/        # 专家节点注册子目录
│   ├── state.json                          # 状态: drafting
│   └── README.md                           # 目录导航
├── specs-tree-expert-workbench/            # 专家工作台子目录
│   ├── state.json                          # 状态: drafting
│   └── README.md                           # 目录导航
└── specs-tree-task-dispatcher/             # 任务分发器子目录
│   ├── state.json                          # 状态: drafting
│   └── README.md                           # 目录导航
```

---

## 文件说明

### 核心规范文件

| 文件 | 说明 | 状态 |
|------|------|------|
| `spec.md` | **产品规范文档** - 包含概述、用户故事、功能需求、非功能需求、技术设计、边界情况、拆分建议等完整规范 | ✅ specified |
| `discovery.md` | **需求挖掘报告** - 核心原则推导、树的构建逻辑、演进逻辑、示例场景等需求分析 | ✅ discovered |
| `plan.md` | **技术规划文档** - 分层架构、技术栈选择、模块设计、数据模型、API 设计、算法设计、实现计划、风险评估 | ✅ planned |

### 元数据文件

| 文件 | 说明 | 状态 |
|------|------|------|
| `spec.json` | 规范元数据 - JSON 格式的规范摘要，包含 Feature ID、状态、核心原则、目标、拆分建议等 | ✅ 存在 |
| `state.json` | 状态文件 - 追踪当前规范状态，包含产品信息、状态、创建日期、子 Feature 列表等 | ✅ specified |

---

## 子目录说明

### specs-tree-expert-node-registry

**专家节点注册** - 管理 AI Agent 和人类角色的注册、发现、匹配

| 属性 | 值 |
|------|-----|
| Feature ID | ETD-FR-REGISTRY-001 |
| 状态 | ⏳ drafting |
| 优先级 | P0 |
| 工作量估算 | Medium |
| 父 Feature | ETD-001 |

**关键能力**：
- 专家节点注册（AI Agent / 人类角色）
- 按生命周期 × 技术栈 × 细分领域匹配专家
- 专家节点层级管理（L0/L1/L2/L3）
- 专家能力声明与认证

---

### specs-tree-expert-workbench

**专家工作台** - 每个专家节点的工作界面

| 属性 | 值 |
|------|-----|
| Feature ID | ETD-FR-WORKBENCH-001 |
| 状态 | ⏳ drafting |
| 优先级 | P1 |
| 工作量估算 | Medium |
| 父 Feature | ETD-001 |

**关键能力**：
- 接收分配的任务
- 继续向下找专家 / 自己处理
- 提交结果
- CLI + Web 界面

---

### specs-tree-task-dispatcher

**任务分发器** - 实现先找专家，找不到才自己做的递归分配逻辑

| 属性 | 值 |
|------|-----|
| Feature ID | ETD-FR-DISPATCHER-001 |
| 状态 | ⏳ drafting |
| 优先级 | P0 |
| 工作量估算 | Medium |
| 父 Feature | ETD-001 |

**关键能力**：
- 任务解析与专家匹配
- 递归向下分配
- 结果向上汇聚
- 支持 AI 执行树 + 人类看护树双树

---

## 项目状态

| 维度 | 值 |
|------|-----|
| **Feature ID** | ETD-001 |
| **当前状态** | specified (规范已完成) |
| **创建日期** | 2026-04-18 |
| **最后更新** | 2026-04-30 |
| **估算工作量** | Large (3-4 人月) |
| **优先级** | P0 |
| **目标用户** | Solo 开发者、小型团队、中型团队 |
| **首要用户** | 团队 |

---

## 核心原则

| 原则 | 说明 |
|------|------|
| **专业分工** | 专业的事交给专业的 AI/人去做 |
| **设计先行** | 谋定而后动，思考规划先于行动 |
| **双树并行** | AI 执行树 + 人类看护树，节点一一对应 |
| **领域无关** | 方法论而非具体解决方案，不绑定特定领域 |

---

## 核心目标

| 目标 | 说明 |
|------|------|
| **动态建树** | 根据现实世界的团队、项目、领域，生成专业协作树 |
| **演进** | 树是活的，随现实变化生长凋落 |
| **质量保障** | 每个节点谋定后动，AI 执行 + 人类看护双线并行 |

---

## 拆分建议

Spec 文档提出了 **4 个垂直 Feature** 拆分方案：

| Feature | 名称 | 包含的需求 |
|---------|------|-----------|
| **ETD-A** | 领域建模 (Domain Modeling) | FR-001, FR-002, FR-003 |
| **ETD-B** | 树生成 (Tree Generation) | FR-004 |
| **ETD-C** | 任务执行 (Task Execution) | FR-005, FR-006, FR-007, FR-009 |
| **ETD-D** | 树演进 (Tree Evolution) | FR-008 |

> 注：当前目录下的三个子 Feature 是 Discovery 阶段的拆分，Spec 阶段提出了新的拆分建议。

---

## 下一步

根据 SDDU 工作流，ETD 项目当前处于 **specified** 状态：

1. ✅ Discovery 完成 (需求挖掘)
2. ✅ Spec 完成 (产品规范)
3. ✅ Plan 完成 (技术规划)
4. 👉 **下一步**: 运行 `@sddu-tasks etd` 开始任务分解

---

## 上级目录

- [返回 SDDU 工作空间根目录](../README.md)

---

## 相关链接

- [产品规范文档](./spec.md)
- [需求挖掘报告](./discovery.md)
- [技术规划文档](./plan.md)
- [专家节点注册子目录](./specs-tree-expert-node-registry/README.md)
- [专家工作台子目录](./specs-tree-expert-workbench/README.md)
- [任务分发器子目录](./specs-tree-task-dispatcher/README.md)