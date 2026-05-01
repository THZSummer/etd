# ETD 技术规划文档

## 版本信息

- **版本**: 2.0.0-plan
- **对应 Spec**: 2.4.0-spec
- **最后更新**: 2026-05-01

---

## 目录

1. [架构概述](#1-架构概述)
2. [技术栈选择](#2-技术栈选择)
3. [模块设计](#3-模块设计)
4. [数据模型设计](#4-数据模型设计)
5. [API 设计](#5-api-设计)
6. [算法设计](#6-算法设计)
7. [实现计划](#7-实现计划)
8. [依赖分析](#8-依赖分析)
9. [风险评估](#9-风险评估)
10. [ADR 记录](#10-adr-记录)

---

## 1. 架构概述

### 1.1 六层分层架构

基于 spec §1.3.3 的平台无关架构要求，ETD 采用六层分层架构：

```
┌─────────────────────────────────────────────────────────────┐
│  L6 表示层 (Presentation)                                    │
│  ├── Web Dashboard (React)                                  │
│  ├── CLI 工具 (Node.js)                                     │
│  └── API Gateway                                            │
├─────────────────────────────────────────────────────────────┤
│  L5 应用层 (Application)                                     │
│  ├── RESTful API (Express/Fastify)                          │
│  ├── GraphQL Endpoint                                       │
│  └── WebSocket 实时通信                                       │
├─────────────────────────────────────────────────────────────┤
│  L4 领域层 (Domain)                                          │
│  ├── 领域服务 (Domain Services)                               │
│  ├── 领域事件 (Domain Events)                                 │
│  └── 业务规则引擎                                             │
├─────────────────────────────────────────────────────────────┤
│  L3 核心层 (Core)                                            │
│  ├── ETD Engine (Expert Tree Designer)                      │
│  │   ├── 领域模板引擎                                         │
│  │   ├── 动态树生成器                                         │
│  │   ├── 演进管理器                                           │
│  │   └── 双树并行引擎                                         │
│  ├── Adapter 抽象层                                           │
│  │   ├── IExecutionAdapter                                    │
│  │   ├── IStorageAdapter                                      │
│  │   ├── INotificationAdapter                                 │
│  │   └── IAuthAdapter                                         │
│  └── 通用工具库                                               │
├─────────────────────────────────────────────────────────────┤
│  L2 基础设施层 (Infrastructure)                              │
│  ├── 持久化层                                                │
│  │   ├── PostgreSQL (主要数据)                                │
│  │   ├── Redis (缓存/队列)                                    │
│  │   └── MinIO/S3 (对象存储)                                  │
│  ├── 消息队列                                                │
│  │   ├── RabbitMQ / Apache Kafka                              │
│  │   └── 内置事件总线                                         │
│  ├── 外部服务客户端                                           │
│  │   ├── LLM Provider Clients                                │
│  │   └── Third-party API Clients                              │
│  └── 平台适配器实现                                           │
│      ├── OpenCode Adapter (优先)                              │
│      ├── OpenClaw Adapter (计划)                              │
│      └── Custom Adapter (完全支持)                            │
├─────────────────────────────────────────────────────────────┤
│  L1 集成层 (Integration)                                     │
│  ├── LLM 集成                                                │
│  │   ├── OpenAI GPT-4/4o                                      │
│  │   ├── Anthropic Claude                                     │
│  │   ├── Google Gemini                                        │
│  │   └── 本地模型 (Ollama/LM Studio)                           │
│  ├── 中间工具集成                                             │
│  │   ├── OpenCode (执行/存储/通知)                             │
│  │   ├── OpenClaw (执行/存储/通知)                             │
│  │   └── 自定义中间工具                                        │
│  └── 通知服务                                                 │
│      ├── Email (SendGrid/AWS SES)                               │
│      ├── Webhook                                              │
│      └── Slack/Discord                                        │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 双树并行架构

基于 spec §1.3.3 和 FR-007，ETD 采用**双树并行架构**：

```
┌─────────────────────────────────────────────────────────────┐
│                    专家工作台 (Expert Workbench)               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │  设计树      │    │  任务中心    │    │  审视工具    │   │
│  │  (Design)    │◄──►│  (Tasks)     │◄──►│  (Review)    │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    双树并行引擎 (Dual-Tree Engine)             │
│                                                              │
│  ┌───────────────────────────┐  ┌───────────────────────────┐│
│  │      设计树              │  │      执行树              ││
│  │    (Design Tree)         │  │  (Execution Tree)         ││
│  │  ┌─────┐ ┌─────┐        │  │  ┌─────┐ ┌─────┐         ││
│  │  │设计1│ │设计2│ ...    │  │  │任务1│ │任务2│ ...     ││
│  │  └──┬──┘ └──┬──┘        │  │  └──┬──┘ └──┬──┘         ││
│  │     │       │          │  │     │       │             ││
│  │     ▼       ▼          │  │     ▼       ▼             ││
│  │   [谋定后动]           │  │   [执行+看护]             ││
│  │   [质量约束]           │  │   [AI执行+人类看护]       ││
│  └───────────────────────────┘  └───────────────────────────┘│
│                              │                               │
│                              ▼                               │
│                    ┌─────────────────┐                        │
│                    │   演进管理器    │                        │
│                    │ (Evolution Mgr) │                        │
│                    └─────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Adapter 抽象层                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ IExecution  │  │ IStorage    │  │ INotification│         │
│  │  Adapter    │  │  Adapter    │  │  Adapter    │          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
│         │                │                │                   │
│         ▼                ▼                ▼                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  OpenCode   │  │  OpenClaw   │  │   Custom    │          │
│  │  Adapter    │  │  (Planned)  │  │  Adapter    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 平台无关架构

基于 spec §1.3.3，ETD 通过 **Adapter 抽象层** 实现平台无关性：

```typescript
// 核心抽象接口
interface IExecutionAdapter {
  execute(task: Task): Promise<ExecutionResult>;
  cancel(executionId: string): Promise<void>;
  getStatus(executionId: string): Promise<ExecutionStatus>;
}

interface IStorageAdapter {
  read<T>(key: string): Promise<T | null>;
  write<T>(key: string, value: T): Promise<void>;
  delete(key: string): Promise<void>;
  list(prefix: string): Promise<string[]>;
}

interface INotificationAdapter {
  send(notification: Notification): Promise<void>;
  subscribe(event: string, handler: EventHandler): void;
  unsubscribe(event: string, handler: EventHandler): void;
}

interface IAuthAdapter {
  authenticate(credentials: Credentials): Promise<AuthResult>;
  authorize(user: User, resource: Resource): Promise<boolean>;
  refresh(token: string): Promise<AuthResult>;
}
```

---

## 2. 技术栈选择

### 2.1 技术选型总览

| 层级 | 技术选型 | 版本 | 说明 |
|------|---------|------|------|
| **L6 表示层** | React 18 | ^18.2.0 | 主界面框架 |
| | Next.js 14 | ^14.0.0 | SSR/SSG |
| | TailwindCSS | ^3.4.0 | 样式系统 |
| | React Query | ^5.0.0 | 数据获取 |
| | Zustand | ^4.4.0 | 状态管理 |
| **L5 应用层** | Fastify | ^4.25.0 | API 框架 |
| | tRPC | ^11.0.0 | 类型安全 API |
| | Zod | ^3.22.0 | 校验/验证 |
| | Socket.io | ^4.7.0 | 实时通信 |
| **L4 领域层** | DDD Toolkit | Custom | 领域驱动设计 |
| | EventEmitter3 | ^5.0.0 | 领域事件 |
| **L3 核心层** | TypeScript | ^5.3.0 | 主语言 |
| | Babel | ^7.23.0 | 转译 |
| | SWC | ^1.3.0 | 快速编译 |
| **L2 基础设施层** | PostgreSQL | 15+ | 主数据库 |
| | Redis | 7+ | 缓存/队列 |
| | MinIO / S3 | Latest | 对象存储 |
| | RabbitMQ / Kafka | Latest | 消息队列 |
| | Docker | 24+ | 容器化 |
| | Kubernetes | 1.28+ | 编排 |
| **L1 集成层** | OpenAI SDK | ^4.20.0 | GPT 集成 |
| | Anthropic SDK | ^0.8.0 | Claude 集成 |
| | Google AI SDK | ^0.2.0 | Gemini 集成 |
| | LangChain | ^0.1.0 | LLM 编排 |
| | OpenCode SDK | ^1.0.0 | 执行集成 |

### 2.2 技术选型理由

#### 为什么使用 TypeScript？

- 类型安全：减少运行时错误
- 开发体验：智能提示、重构支持
- 团队协作：接口即文档
- 生态丰富：Node.js + 浏览器统一语言

#### 为什么使用 Fastify？

- 性能：比 Express 快 20%+
- 插件系统：高度可扩展
- JSON Schema：内置验证
- 低开销：适合高并发场景

#### 为什么使用 PostgreSQL？

- 关系型数据：符合领域模型
- JSON 支持：灵活存储
- 成熟稳定：生产验证
- 扩展丰富：PostGIS, pgvector 等

#### 为什么使用 DDD？

- 业务对齐：代码反映业务
- 边界清晰：模块高内聚
- 易于测试：领域逻辑独立
- 演进友好：支持业务变化

---

## 3. 模块设计

### 3.1 模块总览

基于 spec §3.2 的功能需求 (FR-001 到 FR-010)，设计以下核心模块：

```
┌─────────────────────────────────────────────────────────────┐
│                    ETD 模块架构                              │
├─────────────────────────────────────────────────────────────┤
│  Core Domain Layer                                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │  Domain     │ │  Expert     │ │  Task       │             │
│  │  Model      │ │  Node       │ │  Dispatch   │             │
│  │  (FR-001)   │ │  (FR-002)   │ │  (FR-005)   │             │
│  └─────────────┘ └─────────────┘ └─────────────┘             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │  Dynamic    │ │  Constraint │ │  Evolution  │             │
│  │  Tree Gen   │ │  Engine     │ │  Manager    │             │
│  │  (FR-004)   │ │  (FR-006)   │ │  (FR-008)   │             │
│  └─────────────┘ └─────────────┘ └─────────────┘             │
│  ┌─────────────┐ ┌─────────────┐                             │
│  │  Dual Tree  │ │  Multi      │                             │
│  │  Engine     │ │  Domain     │                             │
│  │  (FR-007)   │ │  (FR-010)   │                             │
│  └─────────────┘ └─────────────┘                             │
├─────────────────────────────────────────────────────────────┤
│  Application Services Layer                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │  Expert     │ │  Template   │ │  Review     │             │
│  │  Workbench  │ │  System     │ │  Engine     │             │
│  │  (FR-009)   │ │  (§1.5)     │ │  (§1.4)     │             │
│  └─────────────┘ └─────────────┘ └─────────────┘             │
│  ┌─────────────┐ ┌─────────────┐                             │
│  │  Team       │ │  Auth       │                             │
│  │  Config     │ │  Service    │                             │
│  │  (FR-003)   │ │  (L2)       │                             │
│  └─────────────┘ └─────────────┘                             │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure Layer                                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │  Database   │ │  Cache      │ │  Message    │             │
│  │  Adapter    │ │  Layer      │ │  Queue      │             │
│  │  (L2)       │ │  (L2)       │ │  (L2)       │             │
│  └─────────────┘ └─────────────┘ └─────────────┘             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │  LLM        │ │  Platform   │ │  Storage    │             │
│  │  Clients    │ │  Adapters   │ │  Adapter    │             │
│  │  (L1)       │ │  (L2)       │ │  (L2)       │             │
│  └─────────────┘ └─────────────┘ └─────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 核心模块详情

#### 3.2.1 领域模型管理模块 (FR-001)

**职责**: 管理领域定义、领域特性和领域关系

**主要组件**:
```typescript
// 领域实体
interface Domain {
  id: string;
  name: string;
  description: string;
  characteristics: DomainCharacteristic[];
  templates: Template[];
  parentId?: string;
  createdAt: Date;
  updatedAt: Date;
}

// 领域服务
class DomainService {
  createDomain(dto: CreateDomainDTO): Promise<Domain>;
  updateDomain(id: string, dto: UpdateDomainDTO): Promise<Domain>;
  deleteDomain(id: string): Promise<void>;
  getDomainTree(rootId: string): Promise<DomainTree>;
  cloneDomain(sourceId: string, targetName: string): Promise<Domain>;
}
```

**数据流**:
1. 用户定义领域 → DomainService 验证 → 存储到 Domain Repository
2. 领域模板生成 → 触发 TemplateEngine → 存储到 Template Store
3. 领域变更 → 发布 DomainChangedEvent → 通知相关模块

---

#### 3.2.2 专家节点注册模块 (FR-002)

**职责**: 管理专家节点定义、能力模型和协作关系

**主要组件**:
```typescript
// 专家节点实体
interface ExpertNode {
  id: string;
  name: string;
  domainId: string;
  capabilities: Capability[];
  personality?: ExpertPersonality;
  constraints: ExpertConstraint[];
  collaborators: string[]; // ExpertNode IDs
  version: number;
  createdAt: Date;
  updatedAt: Date;
}

// 专家服务
class ExpertNodeService {
  registerExpert(dto: RegisterExpertDTO): Promise<ExpertNode>;
  updateExpert(id: string, dto: UpdateExpertDTO): Promise<ExpertNode>;
  getExpertCapabilities(id: string): Promise<Capability[]>;
  findExpertsByCapability(capability: string): Promise<ExpertNode[]>;
  calculateCollaborationScore(expertA: string, expertB: string): Promise<number>;
}
```

---

#### 3.2.3 团队结构配置模块 (FR-003)

**职责**: 管理现实世界的团队结构映射

**主要组件**:
```typescript
// 团队实体
interface Team {
  id: string;
  name: string;
  domainId: string;
  members: TeamMember[];
  hierarchy: HierarchyNode;
  projects: Project[];
  createdAt: Date;
  updatedAt: Date;
}

// 团队服务
class TeamService {
  createTeam(dto: CreateTeamDTO): Promise<Team>;
  updateTeamStructure(teamId: string, hierarchy: HierarchyNode): Promise<Team>;
  mapMemberToExpert(memberId: string, expertNodeId: string): Promise<void>;
  getTeamExpertiseProfile(teamId: string): Promise<ExpertiseProfile>;
}
```

---

#### 3.2.4 动态树生成模块 (FR-004)

**职责**: 根据领域、团队和项目动态生成专业协作树

**主要组件**:
```typescript
// 动态树生成器
class DynamicTreeGenerator {
  async generateTree(config: TreeGenerationConfig): Promise<ExpertTree> {
    // 1. 加载领域模板
    const domainTemplate = await this.templateEngine.loadDomainTemplate(
      config.domainId
    );
    
    // 2. 分析团队结构
    const teamProfile = await this.teamAnalyzer.analyze(config.teamId);
    
    // 3. 匹配专家节点
    const expertAssignments = await this.expertMatcher.match(
      domainTemplate.requiredCapabilities,
      teamProfile
    );
    
    // 4. 生成树结构
    const tree = await this.treeBuilder.build({
      domainTemplate,
      expertAssignments,
      projectContext: config.projectContext
    });
    
    // 5. 应用约束和优化
    return await this.constraintOptimizer.optimize(tree);
  }
}
```

---

#### 3.2.5 任务分配与调度模块 (FR-005)

**职责**: 智能任务分配和调度管理

**主要组件**:
```typescript
// 调度器
class TaskDispatcher {
  async dispatch(task: Task): Promise<DispatchResult> {
    // 1. 解析任务依赖
    const dependencies = await this.dependencyResolver.resolve(task);
    if (!dependencies.satisfied) {
      return { status: 'PENDING', reason: 'DEPENDENCIES_NOT_MET' };
    }
    
    // 2. 匹配专家
    const candidates = await this.expertMatcher.findCandidates(task);
    
    // 3. 选择最佳专家
    const selectedExpert = await this.selectionStrategy.select(candidates, task);
    
    // 4. 分配任务
    task.assignedTo = selectedExpert.id;
    await this.taskRepository.save(task);
    
    return { status: 'DISPATCHED', expert: selectedExpert };
  }
}
```

---

#### 3.2.6 设计先行约束模块 (FR-006)

**职责**: 强制执行「设计先行、谋定后动」的质量约束

**主要组件**:
```typescript
// 约束引擎
class ConstraintEngine {
  async validate(entity: ValidatableEntity): Promise<ValidationResult> {
    const applicableConstraints = this.constraints.filter(c => c.appliesTo(entity));
    
    const results = await Promise.all(
      applicableConstraints.map(constraint => constraint.validate(entity))
    );
    
    const violations = results.filter(r => !r.valid);
    
    return {
      valid: violations.length === 0,
      violations
    };
  }
}

// 设计先行约束实现
class DesignFirstConstraint implements Constraint {
  async validate(entity: ValidatableEntity): Promise<ConstraintResult> {
    const task = entity as ExecutionTask;
    
    // 检查是否有对应的设计节点
    const designNode = await this.treeService.findDesignNode(task.nodeId);
    if (!designNode) {
      return {
        valid: false,
        message: '执行节点必须先完成设计节点',
        severity: 'BLOCKING'
      };
    }
    
    // 检查设计节点是否通过审视
    const reviewResult = await this.reviewService.getResult(designNode.id);
    if (!reviewResult || !reviewResult.approved) {
      return {
        valid: false,
        message: '设计节点必须通过审视才能进入执行',
        severity: 'BLOCKING'
      };
    }
    
    return { valid: true };
  }
}
```

---

#### 3.2.7 双树并行引擎 (FR-007)

**职责**: 同时管理设计树和执行树，支持并行演进

**主要组件**:
```typescript
// 双树引擎
class DualTreeEngine {
  async createDualTree(config: DualTreeConfig): Promise<DualTree> {
    // 创建设计树
    const designTree = await this.designTreeManager.create({
      ...config,
      treeType: 'DESIGN'
    });
    
    // 创建执行树
    const executionTree = await this.executionTreeManager.create({
      ...config,
      treeType: 'EXECUTION',
      parentTreeId: designTree.id
    });
    
    // 建立树间关联
    await this.syncCoordinator.linkTrees(designTree.id, executionTree.id);
    
    return {
      designTreeId: designTree.id,
      executionTreeId: executionTree.id
    };
  }
  
  async syncTreeChanges(sourceTreeId: string, changes: TreeChange[]): Promise<void> {
    const targetTreeId = await this.syncCoordinator.getPairedTree(sourceTreeId);
    
    for (const change of changes) {
      await this.applyChange(targetTreeId, change);
    }
  }
}
```

---

#### 3.2.8 演进管理模块 (FR-008)

**职责**: 管理树的动态演进，包括生长、凋落和重构

**主要组件**:
```typescript
// 演进管理器
class EvolutionManager {
  async detectRealWorldChanges(treeId: string): Promise<ChangeReport> {
    const tree = await this.treeRepo.findById(treeId);
    
    // 检测团队变化
    const teamChanges = await this.changeDetector.detectTeamChanges(tree.teamId);
    
    // 检测项目变化
    const projectChanges = await this.changeDetector.detectProjectChanges(tree.projectId);
    
    // 检测领域变化
    const domainChanges = await this.changeDetector.detectDomainChanges(tree.domainId);
    
    return {
      treeId,
      detectedAt: new Date(),
      changes: [...teamChanges, ...projectChanges, ...domainChanges],
      impact: this.assessImpact([...teamChanges, ...projectChanges, ...domainChanges])
    };
  }
  
  async growTree(treeId: string, growthPlan: GrowthPlan): Promise<GrowthResult> {
    const tree = await this.treeRepo.findById(treeId);
    const result = await this.growthEngine.execute(tree, growthPlan);
    await this.treeRepo.update(treeId, result.updatedTree);
    return result;
  }
  
  async witherTree(treeId: string, witheringPlan: WitheringPlan): Promise<WitheringResult> {
    const tree = await this.treeRepo.findById(treeId);
    const result = await this.witheringEngine.execute(tree, witheringPlan);
    await this.treeRepo.update(treeId, result.updatedTree);
    return result;
  }
  
  async refactorTree(treeId: string, refactoringPlan: RefactoringPlan): Promise<RefactoringResult> {
    const tree = await this.treeRepo.findById(treeId);
    const result = await this.refactoringEngine.execute(tree, refactoringPlan);
    await this.treeRepo.update(treeId, result.updatedTree);
    return result;
  }
}
```

---

#### 3.2.9 专家工作台模块 (FR-009)

**职责**: 为专家提供交互式工作台，支持设计和执行工作

**主要组件**:
```typescript
// 专家工作台
class ExpertWorkbench {
  async getWorkbenchOverview(expertId: string): Promise<WorkbenchOverview> {
    const [assignedTrees, pendingTasks, recentActivity] = await Promise.all([
      this.treeService.getTreesByExpert(expertId),
      this.taskService.getPendingTasks(expertId),
      this.getRecentActivity(expertId)
    ]);
    
    return {
      expertId,
      summary: {
        activeTrees: assignedTrees.filter(t => t.status === 'ACTIVE').length,
        pendingTasks: pendingTasks.length,
        completedTasksThisWeek: recentActivity.completedTasks.length
      },
      trees: assignedTrees,
      pendingTasks,
      recentActivity
    };
  }
  
  async openDesignStudio(params: DesignStudioParams): Promise<DesignSession> {
    const session = await this.designStudio.createSession({
      expertId: params.expertId,
      treeId: params.treeId,
      nodeId: params.nodeId,
      mode: params.mode
    });
    
    await session.loadContext({
      tree: await this.treeService.getTree(params.treeId),
      templates: await this.getApplicableTemplates(params.domainId)
    });
    
    return session;
  }
  
  async openExecutionCenter(params: ExecutionCenterParams): Promise<ExecutionSession> {
    const session = await this.executionCenter.createSession({
      expertId: params.expertId,
      treeId: params.treeId,
      taskId: params.taskId
    });
    
    await session.loadContext({
      task: await this.taskService.getTask(params.taskId),
      executionHistory: await this.getExecutionHistory(params.taskId)
    });
    
    return session;
  }
}
```

---

#### 3.2.10 多领域适配模块 (FR-010)

**职责**: 支持多个领域的树生成和管理

**主要组件**:
```typescript
// 领域适配器
interface DomainAdapter {
  domainId: string;
  getCapabilities(): Capability[];
  getTemplates(): DomainTemplate[];
  getConstraints(): DomainConstraint[];
  getQualityChecks(): QualityCheck[];
  getEvolutionRules(): EvolutionRule[];
}

// 多领域管理器
class MultiDomainManager {
  private adapters: Map<string, DomainAdapter> = new Map();
  
  registerAdapter(adapter: DomainAdapter): void {
    this.adapters.set(adapter.domainId, adapter);
  }
  
  async getAdapter(domainId: string): Promise<DomainAdapter> {
    const adapter = this.adapters.get(domainId);
    if (!adapter) {
      return await this.loadAdapterDynamically(domainId);
    }
    return adapter;
  }
  
  async generateCrossDomainTree(
    primaryDomainId: string,
    secondaryDomains: string[],
    config: CrossDomainConfig
  ): Promise<ExpertTree> {
    const adapters = await Promise.all([
      this.getAdapter(primaryDomainId),
      ...secondaryDomains.map(id => this.getAdapter(id))
    ]);
    
    const mergedCapabilities = this.mergeCapabilities(
      adapters.map(a => a.getCapabilities())
    );
    
    return await this.crossDomainGenerator.generate({
      primaryDomain: primaryDomainId,
      secondaryDomains,
      capabilities: mergedCapabilities,
      config
    });
  }
}
```

---

## 4. 数据模型设计

### 4.1 五元模型映射

根据 spec 定义的五元关系，设计数据模型：

```sql
-- 领域表
CREATE TABLE domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    parent_id UUID REFERENCES domains(id),
    characteristics JSONB DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 专家节点表
CREATE TABLE expert_nodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id UUID NOT NULL REFERENCES domains(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    capabilities JSONB DEFAULT '[]',
    personality JSONB DEFAULT '{}',
    constraints JSONB DEFAULT '[]',
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 专业协作树表
CREATE TABLE expert_trees (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id UUID NOT NULL REFERENCES domains(id),
    team_id UUID NOT NULL,
    project_id UUID,
    root_node_id UUID,
    tree_type VARCHAR(50) NOT NULL CHECK (tree_type IN ('DESIGN', 'EXECUTION')),
    paired_tree_id UUID REFERENCES expert_trees(id),
    status VARCHAR(50) DEFAULT 'ACTIVE',
    version INTEGER DEFAULT 1,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 树节点表
CREATE TABLE tree_nodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tree_id UUID NOT NULL REFERENCES expert_trees(id) ON DELETE CASCADE,
    expert_node_id UUID REFERENCES expert_nodes(id),
    parent_id UUID REFERENCES tree_nodes(id),
    type VARCHAR(50) NOT NULL CHECK (type IN ('EXPERT', 'TASK', 'MILESTONE', 'DECISION')),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'PENDING',
    position JSONB DEFAULT '{"x": 0, "y": 0}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 任务表
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tree_id UUID NOT NULL REFERENCES expert_trees(id),
    node_id UUID NOT NULL REFERENCES tree_nodes(id),
    type VARCHAR(50) NOT NULL,
    priority VARCHAR(20) DEFAULT 'MEDIUM',
    status VARCHAR(50) DEFAULT 'PENDING',
    assigned_to UUID REFERENCES expert_nodes(id),
    scheduled_at TIMESTAMP WITH TIME ZONE,
    deadline TIMESTAMP WITH TIME ZONE,
    dependencies UUID[] DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

---

## 5. API 设计

### 5.1 RESTful API 概览

```yaml
openapi: 3.0.0
info:
  title: ETD API
  version: 1.0.0
  description: Expert Tree Designer API

servers:
  - url: /api/v1

paths:
  /domains:
    get:
      summary: 获取领域列表
      parameters:
        - name: parent_id
          in: query
          schema:
            type: string
      responses:
        200:
          description: 领域列表
          
    post:
      summary: 创建领域
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                description:
                  type: string
      responses:
        201:
          description: 创建成功

  /expert-nodes:
    get:
      summary: 获取专家节点列表
      parameters:
        - name: domain_id
          in: query
          schema:
            type: string
      responses:
        200:
          description: 专家节点列表
          
    post:
      summary: 注册专家节点
      responses:
        201:
          description: 注册成功

  /trees:
    get:
      summary: 获取协作树列表
      responses:
        200:
          description: 协作树列表
          
    post:
      summary: 创建协作树
      responses:
        201:
          description: 创建成功

  /tasks:
    get:
      summary: 获取任务列表
      responses:
        200:
          description: 任务列表
          
    post:
      summary: 创建任务
      responses:
        201:
          description: 创建成功
```

---

## 6. 算法设计

### 6.1 动态树生成算法

```typescript
/**
 * 动态树生成算法
 * 基于领域、团队和项目上下文生成专业协作树
 */
class DynamicTreeGenerationAlgorithm {
  
  async generate(config: TreeGenerationConfig): Promise<ExpertTree> {
    // Phase 1: 领域分析
    const domainAnalysis = await this.analyzeDomain(config.domainId);
    
    // Phase 2: 团队能力映射
    const teamCapabilities = await this.mapTeamCapabilities(config.teamId);
    
    // Phase 3: 专家匹配
    const expertAssignments = await this.matchExperts(
      domainAnalysis.requiredCapabilities,
      teamCapabilities
    );
    
    // Phase 4: 树结构生成
    const treeStructure = await this.generateTreeStructure({
      domainAnalysis,
      expertAssignments,
      projectContext: config.projectContext
    });
    
    // Phase 5: 约束应用
    const constrainedTree = await this.applyConstraints(treeStructure);
    
    // Phase 6: 优化
    return await this.optimizeTree(constrainedTree);
  }
  
  private async matchExperts(
    requiredCapabilities: Capability[],
    teamCapabilities: TeamCapability[]
  ): Promise<ExpertAssignment[]> {
    const assignments: ExpertAssignment[] = [];
    
    for (const capability of requiredCapabilities) {
      const candidates = teamCapabilities.filter(
        tc => this.capabilityMatcher.match(tc.capabilities, capability)
      );
      
      if (candidates.length === 0) {
        assignments.push({
          capability,
          expert: null,
          confidence: 0,
          gap: true
        });
      } else {
        const bestMatch = this.selectBestMatch(candidates, capability);
        assignments.push({
          capability,
          expert: bestMatch.expert,
          confidence: bestMatch.confidence,
          gap: false
        });
      }
    }
    
    return assignments;
  }
}
```

---

## 7. 实现计划

### 7.1 开发阶段

基于 spec §7.2 的 Feature 拆分，制定以下实现计划：

```
Phase 1: 核心引擎 (Weeks 1-4)
├── Week 1-2: 基础架构搭建
│   ├── 项目结构初始化
│   ├── 数据库设计实现
│   ├── 基础框架搭建
│   └── CI/CD 配置
├── Week 3-4: 核心引擎开发
│   ├── 领域模型管理 (FR-001)
│   ├── 专家节点注册 (FR-002)
│   ├── 团队结构配置 (FR-003)
│   └── 动态树生成核心 (FR-004)

Phase 2: 双树引擎 (Weeks 5-7)
├── Week 5: 设计树实现
│   ├── 设计工作室 (FR-009 部分)
│   ├── 设计先行约束 (FR-006)
│   └── 审视引擎集成
├── Week 6: 执行树实现
│   ├── 任务分配调度 (FR-005)
│   ├── 执行中心 (FR-009 部分)
│   └── 质量保障集成
├── Week 7: 双树并行引擎
│   ├── 双树同步算法 (FR-007)
│   ├── 树间映射管理
│   └── 约束传播机制

Phase 3: 演进系统 (Weeks 8-9)
├── Week 8: 演进管理
│   ├── 变化检测引擎 (FR-008)
│   ├── 生长算法
│   ├── 凋落处理
│   └── 重构引擎
├── Week 9: 模板系统
│   ├── 领域模板 (§1.5)
│   ├── 专家模板
│   ├── 任务模板
│   └── 审视模板

Phase 4: 平台适配 (Week 10)
├── Adapter 抽象层
│   ├── IExecutionAdapter
│   ├── IStorageAdapter
│   ├── INotificationAdapter
│   └── IAuthAdapter
├── OpenCode Adapter 实现 (优先)
├── OpenClaw Adapter 规划
└── Custom Adapter 支持

Phase 5: 集成与优化 (Weeks 11-12)
├── Week 11: 多领域支持 (FR-010)
│   ├── 领域适配器框架
│   ├── 跨领域树生成
│   └── 领域特定扩展点
├── Week 12: 性能优化与测试
│   ├── 性能测试与优化
│   ├── 边界情况处理 (§6)
│   ├── 集成测试
│   └── 文档完善
```

### 7.2 里程碑

| 里程碑 | 日期 | 交付物 | 验收标准 |
|--------|------|--------|----------|
| **M1: 核心引擎完成** | Week 4 | • 领域模型管理<br>• 专家节点系统<br>• 基础树生成 | • 可创建和管理领域<br>• 可注册和匹配专家<br>• 可生成简单树结构 |
| **M2: 双树引擎完成** | Week 7 | • 设计树/执行树<br>• 双树同步<br>• 约束引擎 | • 可创建双树<br>• 设计先行约束生效<br>• 任务可分配调度 |
| **M3: 演进系统完成** | Week 9 | • 变化检测<br>• 树演进算法<br>• 模板系统 | • 可检测现实变化<br>• 树可生长/凋落/重构<br>• 模板可应用 |
| **M4: 平台适配完成** | Week 10 | • Adapter 抽象层<br>• OpenCode Adapter<br>• 文档 | • 可在 OpenCode 环境运行<br>• Adapter 接口稳定 |
| **M5: Beta 发布** | Week 12 | • 完整系统<br>• 多领域支持<br>• 文档和示例 | • 通过完整测试套件<br>• 性能达标<br>• 可演示完整流程 |

---

## 8. 依赖分析

### 8.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                     模块依赖关系图                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                            │
│  │  Web UI      │                                            │
│  └──────┬───────┘                                            │
│         │ 依赖                                               │
│         ▼                                                    │
│  ┌──────────────┐     ┌──────────────┐                      │
│  │  API Layer   │◄────►│  Expert      │                      │
│  └──────┬───────┘     │  Workbench   │                      │
│         │             └──────────────┘                      │
│         │ 依赖                                               │
│         ▼                                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Application Services                    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │   │
│  │  │ Domain   │ │ Team     │ │ Template │            │   │
│  │  │ Service  │ │ Service  │ │ Service  │            │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │   │
│  └───────┼───────────┼───────────┼────────────────────┘   │
│          │           │           │                          │
│          ▼           ▼           ▼                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Core Domain Layer                       │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │   │
│  │  │ Dynamic  │ │ Dual     │ │ Constraint│            │   │
│  │  │ Tree Gen │ │ Tree Eng │ │ Engine   │            │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │   │
│  │       │           │           │                    │   │
│  │  ┌────┴───────────┴───────────┴────┐              │   │
│  │  │     Evolution Manager          │              │   │
│  │  └───────────────────────────────┘              │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Infrastructure Layer                    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │   │
│  │  │ Database │ │ Cache    │ │ Message  │            │   │
│  │  │ Adapter  │ │ Layer    │ │ Queue    │            │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │   │
│  │       │           │           │                    │   │
│  │  ┌────┴───────────┴───────────┴────┐              │   │
│  │  │     Platform Adapters          │              │   │
│  │  │  (OpenCode/OpenClaw/Custom)    │              │   │
│  │  └───────────────────────────────┘              │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 外部依赖

| 依赖类型 | 名称 | 版本 | 用途 | 风险等级 |
|----------|------|------|------|----------|
| **数据库** | PostgreSQL | 15+ | 主数据存储 | 低 |
| **缓存** | Redis | 7+ | 缓存/会话/队列 | 低 |
| **消息队列** | RabbitMQ | 3.12+ | 异步消息 | 中 |
| **对象存储** | MinIO | Latest | 文件存储 | 低 |
| **LLM** | OpenAI API | v1 | GPT-4/4o 集成 | 中 |
| **LLM** | Anthropic API | v1 | Claude 集成 | 中 |
| **中间工具** | OpenCode | ^1.0 | 执行环境 | 高 |
| **容器** | Docker | 24+ | 容器化 | 低 |
| **编排** | Kubernetes | 1.28+ | 容器编排 | 中 |

---

## 9. 风险评估

### 9.1 风险矩阵

| 风险 | 可能性 | 影响 | 风险等级 | 缓解策略 | 负责人 |
|------|--------|------|----------|----------|--------|
| **EC-001: 专家节点匹配失败** | 中 | 高 | 🔴 高 | 实现专家推荐算法，支持部分匹配 | 算法团队 |
| **EC-002: 循环依赖检测** | 中 | 高 | 🔴 高 | 构建依赖图，实施拓扑排序验证 | 架构团队 |
| **EC-003: 并发任务冲突** | 高 | 中 | 🟡 中 | 实现分布式锁和乐观并发控制 | 后端团队 |
| **EC-004: 网络分区容错** | 低 | 高 | 🟡 中 | 设计分区感知算法，支持脑裂恢复 | SRE 团队 |
| **EC-005: 长时间运行任务处理** | 高 | 中 | 🟡 中 | 实现任务检查点和断点续传 | 后端团队 |
| **LLM API 不稳定** | 中 | 高 | 🔴 高 | 多供应商路由 + 本地模型降级 | 集成团队 |
| **性能瓶颈** | 中 | 中 | 🟡 中 | 早期性能测试 + 可扩展架构 | 性能团队 |
| **需求变更** | 高 | 中 | 🟡 中 | 敏捷迭代 + 模块化设计 | 产品团队 |
| **团队技能差距** | 中 | 中 | 🟡 中 | 早期培训 + 外部咨询 | 工程经理 |

---

## 10. ADR 记录

### ADR-001: 六层分层架构

**状态**: ACCEPTED

**背景**: ETD 需要清晰的架构分层来支持复杂的业务逻辑和技术实现。

**决策**: 采用六层分层架构：
1. L6 表示层 - 用户界面
2. L5 应用层 - API 和协议
3. L4 领域层 - 业务逻辑
4. L3 核心层 - ETD 引擎
5. L2 基础设施层 - 数据持久化
6. L1 集成层 - 外部服务

**后果**:
- ✅ 职责清晰，易于维护
- ✅ 支持独立测试各层
- ⚠️ 可能引入一定的性能开销
- ⚠️ 需要严格的接口设计

---

### ADR-002: 双树并行架构

**状态**: ACCEPTED

**背景**: 需要同时支持「设计先行」和「执行并行」的工作模式，确保质量约束的同时提高效率。

**决策**: 采用双树并行架构：
- 设计树 (Design Tree) - 负责「谋定后动」
- 执行树 (Execution Tree) - 负责「执行+看护」
- 双树同步引擎 - 保持两树一致性

**后果**:
- ✅ 严格保证「设计先行」原则
- ✅ 支持设计到执行的平滑过渡
- ✅ 允许并行工作提高效率
- ⚠️ 增加了系统复杂性
- ⚠️ 需要解决树间同步冲突

---

### ADR-003: 平台无关架构

**状态**: ACCEPTED

**背景**: ETD 需要在不同的执行环境中运行（OpenCode、OpenClaw、自定义环境），需要抽象底层差异。

**决策**: 采用 Adapter 抽象层实现平台无关性：
- IExecutionAdapter - 执行操作抽象
- IStorageAdapter - 存储操作抽象
- INotificationAdapter - 通知操作抽象
- IAuthAdapter - 认证操作抽象

**后果**:
- ✅ 可在多种平台运行
- ✅ 易于添加新平台支持
- ✅ 测试时可以 Mock
- ⚠️ 可能限制平台特定功能的使用
- ⚠️ Adapter 实现需要维护

---

### ADR-004: TypeScript 作为主要语言

**状态**: ACCEPTED

**背景**: 需要选择一种能够支持复杂领域模型、提供良好开发体验并且生态丰富的编程语言。

**决策**: 采用 TypeScript 作为主要开发语言：
- 类型安全减少运行时错误
- 优秀的 IDE 支持和开发体验
- 丰富的 npm 生态系统
- 支持前后端统一语言

**后果**:
- ✅ 提高代码质量和可维护性
- ✅ 增强团队协作和代码可读性
- ✅ 丰富的类型定义和工具支持
- ⚠️ 需要编译步骤
- ⚠️ 类型系统学习曲线

---

### ADR-005: PostgreSQL 作为主数据库

**状态**: ACCEPTED

**背景**: 需要选择一种能够支持复杂关系型数据、提供良好一致性保证并且生态成熟的数据库。

**决策**: 采用 PostgreSQL 作为主数据库：
- 成熟稳定的关系型数据库
- 支持 JSON 数据类型实现灵活存储
- 强大的事务和并发控制
- 丰富的扩展生态系统

**后果**:
- ✅ 保证数据一致性和完整性
- ✅ 支持复杂查询和事务
- ✅ 灵活的 JSON 存储能力
- ⚠️ 水平扩展相对复杂
- ⚠️ 需要专业的 DBA 维护

---

## 版本历史

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|----------|------|
| 2.0.0-plan | 2026-05-01 | 基于 spec v2.4.0 全面更新：<br>• 添加平台无关架构和 Adapter 层<br>• 完善双树并行架构设计<br>• 增加演进管理模块详细设计<br>• 更新实现计划为 5 个 Phase<br>• 添加新的 ADR (003-006)<br>• 增加边界情况处理代码示例 | SDDU Planner |
| 1.0.0-plan | 2026-04-15 | 初始版本，基于 spec v1.0.0 | SDDU Planner |

---

*文档结束*
