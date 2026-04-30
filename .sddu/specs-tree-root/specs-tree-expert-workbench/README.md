# 目录: 专家工作台

## 目录简介

本目录是 **Expert Workbench** (专家工作台) 的规范树目录，负责为每个专家节点提供工作界面，支持任务接收、处理、提交。

**Feature ID**: ETD-FR-WORKBENCH-001  
**父 Feature**: ETD-001 (Expert Tree Design)

---

## 目录结构

```
.sddu/specs-tree-root/specs-tree-expert-workbench/
├── README.md          # 本文件 - 目录导航
└── state.json         # 状态文件
```

---

## 文件说明

| 文件 | 说明 | 状态 |
|------|------|------|
| `state.json` | 状态文件 - 追踪当前 Feature 状态、元数据、关键能力 | ⏳ drafting |

---

## Feature 概述

| 属性 | 值 |
|------|-----|
| **Feature 名称** | Expert Workbench |
| **Feature ID** | ETD-FR-WORKBENCH-001 |
| **中文名称** | 专家工作台 |
| **当前状态** | ⏳ drafting (草稿阶段) |
| **优先级** | P1 |
| **工作量估算** | Medium |
| **父 Feature** | ETD-001 |
| **创建日期** | 2026-04-18 |

---

## 关键能力

- **任务接收** - 接收分配的任务
- **专家委托** - 继续向下找专家或自己处理
- **结果提交** - 提交处理结果
- **多终端支持** - CLI + Web 界面

---

## 功能需求覆盖

根据父 Feature 规范，本目录主要覆盖：

| 需求 ID | 名称 | 优先级 |
|---------|------|--------|
| FR-009 | 专家工作台 | P1 |

---

## 状态追踪

| 阶段 | 状态 | 说明 |
|------|------|------|
| Discovery | ⏳ 待启动 | 需求挖掘阶段 |
| Spec | ⏳ 待启动 | 规范编写阶段 |
| Plan | ⏳ 待启动 | 技术规划阶段 |
| Tasks | ⏳ 待启动 | 任务分解阶段 |
| Build | ⏳ 待启动 | 实现阶段 |
| Validate | ⏳ 待启动 | 验证阶段 |

---

## 上级目录

- [返回 ETD 规范树根目录](../README.md)
- [返回 SDDU 工作空间根目录](../../README.md)

---

## 相关链接

- [ETD 产品规范](../spec.md) - 查看完整的 ETD 规范定义
- [ETD 需求挖掘](../discovery.md) - 查看 ETD 需求分析
- [ETD 技术规划](../plan.md) - 查看 ETD 技术规划