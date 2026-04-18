# ETD 工作空间

> **AI执行 × 人类看护**

## 目录结构

```
.sddu/                           # 当前使用 SDDU 规范开发前期版本
├── README.md                    # 本文件
├── specs-tree-root/             # 规范目录
│   ├── README.md
│   ├── discovery.md             # 需求挖掘报告
│   ├── state.json               # 产品状态
│   ├── specs-tree-expert-node-registry/
│   ├── specs-tree-task-dispatcher/
│   └── specs-tree-expert-workbench/
└── docs/                        # 文档目录
```

## 核心原则

| 原则 | 说明 |
|------|------|
| **专业分工** | 专业的事交给专业的 AI/人去做 |
| **设计先行** | 所有实施必须先有设计 |
| **双树并行** | AI 执行树 + 人类看护树 |

## 开发规范说明

- **当前**: 使用 `.sddu` 规范开发前期版本
- **后期**: 切换到 `.etd` 规范实现自迭代

## 下一步

👉 [查看需求挖掘报告](./specs-tree-root/discovery.md)