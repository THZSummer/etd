# SDDU Workspace: ETD 项目

## 目录简介

本目录是 **ETD (Expert Tree Design)** 项目的 SDDU 工作空间。ETD 是一套领域无关的方法论，为任意领域的复杂任务提供动态构建专业协作树的机制，实现 AI 执行 + 人类看护、谋定而后动。

**核心定位**：
- **E**xpert - 专家分工
- **T**ree - 动态协作树
- **D**esign - 设计先行

**产品形态**：三层架构（L1 SDK / L2 CLI / L3 SaaS）

---

## 目录结构

```
.sddu/
├── README.md                              # 本文件 - 工作空间导航
└── specs-tree-root/                       # ETD 规范树根目录
    ├── README.md                          # 规范树目录导航
    ├── spec.md                            # 产品规范 (状态: specified)
    ├── discovery.md                       # 需求挖掘报告 (状态: discovered)
    ├── plan.md                            # 技术规划 (状态: planned)
    ├── spec.json                          # 规范元数据
    ├── state.json                         # 状态文件 (specified)
    ├── specs-tree-expert-node-registry/   # 专家节点注册子目录
    ├── specs-tree-expert-workbench/       # 专家工作台子目录
    └── specs-tree-task-dispatcher/        # 任务分发器子目录
```

---

## 文件说明

| 文件/目录 | 说明 | 状态 |
|-----------|------|------|
| `specs-tree-root/` | ETD 规范树根目录 | ✅ 存在 |
| `specs-tree-root/spec.md` | ETD 产品规范文档 - 详细的规范定义 | ✅ specified |
| `specs-tree-root/discovery.md` | 需求挖掘报告 - 核心原则、目标、逻辑 | ✅ discovered |
| `specs-tree-root/plan.md` | 技术规划 - 架构、技术栈、模块设计 | ✅ planned |
| `specs-tree-root/spec.json` | 规范元数据 - JSON 格式的规范摘要 | ✅ 存在 |
| `specs-tree-root/state.json` | 状态文件 - 当前规范状态追踪 | ✅ specified |

---

## 子目录说明

| 目录 | 说明 | 状态 |
|------|------|------|
| `specs-tree-root/specs-tree-expert-node-registry/` | 专家节点注册 - 管理专家节点的注册、发现、匹配 | ⏳ drafting |
| `specs-tree-root/specs-tree-expert-workbench/` | 专家工作台 - 每个专家节点的工作界面 | ⏳ drafting |
| `specs-tree-root/specs-tree-task-dispatcher/` | 任务分发器 - 递归分配逻辑与双树并行 | ⏳ drafting |

---

## 项目状态

- **Feature ID**: ETD-001
- **当前状态**: `specified` (规范已完成)
- **创建日期**: 2026-04-18
- **最后更新**: 2026-04-30
- **估算工作量**: Large (3-4 人月)
- **优先级**: P0

---

## 核心原则

1. **专业分工** - 专业的事交给专业的 AI/人去做
2. **设计先行** - 谋定而后动，思考规划先于行动
3. **双树并行** - AI 执行树 + 人类看护树，节点一一对应
4. **领域无关** - 方法论而非具体解决方案，不绑定特定领域

---

## 下一步

根据 SDDU 工作流，ETD 项目当前处于 **specified** 状态：

1. ✅ Discovery 完成 - 需求挖掘已完成
2. ✅ Spec 完成 - 产品规范已编写
3. ✅ Plan 完成 - 技术规划已完成
4. 👉 **下一步**: 运行 `@sddu-tasks etd` 开始任务分解

---

## 相关链接

- [ETD 规范树根目录](./specs-tree-root/README.md)
- [产品规范文档](./specs-tree-root/spec.md)
- [需求挖掘报告](./specs-tree-root/discovery.md)
- [技术规划文档](./specs-tree-root/plan.md)