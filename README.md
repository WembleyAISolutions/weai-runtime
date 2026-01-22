# weai-runtime

> Execution runtime for AI agents in the real economy.

This repository implements the runtime layer responsible for **resolving, executing, metering, and settling** skill-based actions registered in the WeAI platform. It enforces policy, billing, trust, and audit guarantees for real-world agent execution.

---

## 1️⃣ What the WeAI Runtime Does

The WeAI Runtime is responsible for executing skill-based actions registered in the platform.

**It handles:**
- Skill resolution and execution
- Policy and permission enforcement
- Metering and billing hooks
- Settlement and audit guarantees

**The runtime does NOT define:**
- Business logic
- Pricing rules
- UI behavior
- Industry-specific workflows

> 💡 **Core Principle**: "Skills are stable assets. Agents, models, and applications are replaceable."

---

## 2️⃣ Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      AGENT EXECUTION FLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. Agent Request                                              │
│      ↓                                                          │
│   2. Skill Resolution (from Registry)                           │
│      ↓                                                          │
│   3. Policy & Permission Check                                  │
│      ↓                                                          │
│   4. Execution (or Dry-Run Simulation)                          │
│      ↓                                                          │
│   5. Metering & Billing Hooks                                   │
│      ↓                                                          │
│   6. Settlement & Audit Records                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Detailed Flow

| Step | Component | Responsibility |
|------|-----------|----------------|
| 1 | **Resolver** | Locate skill definition from registry |
| 2 | **Policy Engine** | Validate permissions, quotas, jurisdiction |
| 3 | **Executor** | Invoke skill adapter with context |
| 4 | **Meter** | Record usage metrics (calls, tokens, etc.) |
| 5 | **Settler** | Calculate charges, update balances |
| 6 | **Auditor** | Produce immutable execution records |

---

## 3️⃣ Relationship to Core and Registry

```
┌─────────────────────────────────────────────────────────────────┐
│                     WeAI PLATFORM HIERARCHY                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   weai-core (宪法)                                              │
│   ├── Type definitions                                          │
│   ├── Canonical schemas                                         │
│   └── Platform constants                                        │
│          ↓ governs                                              │
│   weai-registry (登记簿)                                        │
│   ├── Skill manifests                                           │
│   ├── Provider registry                                         │
│   └── Discovery index                                           │
│          ↓ provides to                                          │
│   weai-runtime (执行器) ← YOU ARE HERE                          │
│   ├── Execution engine                                          │
│   ├── Billing hooks                                             │
│   └── Audit trail                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Authority Rules

- Skill definitions and schemas are governed by `weai-core`
- Only Skills registered in `weai-registry` may be executed
- In case of conflict, `weai-core` specifications take precedence

> **The runtime enforces — but does not redefine — platform rules.**

---

## 4️⃣ Public and Internal Implementations

This repository represents the **public execution model and interfaces**.

| Layer | Repository | Visibility |
|-------|------------|------------|
| Interfaces & Specs | `weai-runtime` | **Public** |
| Production Runtime | `weai-runtime-internal` | Private |

### Why Separate?

Production implementations may include:
- Jurisdiction-specific logic
- Proprietary optimizations
- Commercial integrations that are intentionally not public

> 👉 This separation ensures: **"public repo 没代码 = 没能力" is false.** The public repo defines the standard; private repos implement it.

---

## 5️⃣ Architecture & Interfaces

### Component Responsibilities

```typescript
// 1. Resolver - 技能解析
interface SkillResolver {
  resolve(skillId: string): Promise<SkillDefinition>;
  validate(skill: SkillDefinition): ValidationResult;
}

// 2. Executor - 执行引擎
interface SkillExecutor {
  execute<I, O>(request: ExecutionRequest<I>): Promise<ExecutionResult<O>>;
  dryRun<I, O>(request: ExecutionRequest<I>): Promise<SimulationResult<O>>;
}

// 3. Meter - 计量器
interface UsageMeter {
  record(execution: ExecutionRecord): Promise<void>;
  getUsage(orgId: string, period: Period): Promise<UsageReport>;
}

// 4. Settler - 结算器
interface SettlementEngine {
  calculate(usage: UsageReport): Promise<SettlementPreview>;
  execute(settlement: SettlementPreview): Promise<SettlementResult>;
}
```

### Skill Execution Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│                   SKILL EXECUTION LIFECYCLE                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PENDING → VALIDATING → EXECUTING → METERING → SETTLED      │
│     │          │            │           │          │         │
│     │          │            │           │          └─ Done   │
│     │          │            │           └─ Billing recorded  │
│     │          │            └─ Adapter invoked               │
│     │          └─ Permissions checked                        │
│     └─ Request received                                      │
│                                                              │
│  Error States: REJECTED | FAILED | TIMEOUT | ROLLBACK        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6️⃣ Interface Protocols

### Execution Request / Response

```typescript
// Request
interface ExecutionRequest<T = unknown> {
  skillId: string;
  version?: string;
  input: T;
  context: ExecutionContext;
  options?: ExecutionOptions;
}

interface ExecutionContext {
  actor: 'HUMAN' | 'AI_AGENT' | 'SYSTEM' | 'WEBHOOK';
  orgId: string;
  userId?: string;
  permissions: string[];
  jurisdiction: Jurisdiction;
  correlationId: string;
  timestamp: Date;
}

// Response
interface ExecutionResult<T = unknown> {
  success: boolean;
  skillId: string;
  output?: T;
  error?: ExecutionError;
  metrics: ExecutionMetrics;
  audit: AuditRecord;
}
```

### Error Mapping

| Error Code | Description | Retryable |
|------------|-------------|-----------|
| `SKILL_NOT_FOUND` | Skill not in registry | No |
| `PERMISSION_DENIED` | Insufficient permissions | No |
| `QUOTA_EXCEEDED` | Usage limit reached | Yes (after reset) |
| `VALIDATION_FAILED` | Invalid input | No |
| `EXECUTION_FAILED` | Adapter error | Maybe |
| `TIMEOUT` | Execution timeout | Yes |
| `SETTLEMENT_FAILED` | Billing error | Yes |

### Billing Hooks

```typescript
interface BillingHook {
  // Called before execution (for quota check)
  preExecute(context: ExecutionContext, skill: SkillDefinition): Promise<BillingDecision>;
  
  // Called after execution (for metering)
  postExecute(result: ExecutionResult, metrics: ExecutionMetrics): Promise<void>;
  
  // Called on settlement cycle
  settle(usage: UsageReport): Promise<SettlementResult>;
}
```

> ⚠️ **Note**: Billing hooks exist, but pricing/rates are NOT defined here. They come from `weai-registry` or platform configuration.

---

## 7️⃣ Examples / Stub

### Mock Executor

```typescript
import { SkillExecutor, ExecutionRequest, ExecutionResult } from './interfaces';

export class MockExecutor implements SkillExecutor {
  async execute<I, O>(request: ExecutionRequest<I>): Promise<ExecutionResult<O>> {
    console.log(`[MockExecutor] Executing skill: ${request.skillId}`);
    
    return {
      success: true,
      skillId: request.skillId,
      output: { message: 'Mock execution completed' } as O,
      metrics: {
        startTime: new Date(),
        endTime: new Date(),
        durationMs: 100,
        tokensUsed: 0,
      },
      audit: {
        id: `audit_${Date.now()}`,
        timestamp: new Date(),
        actor: request.context.actor,
        action: 'EXECUTE',
        skillId: request.skillId,
      },
    };
  }

  async dryRun<I, O>(request: ExecutionRequest<I>): Promise<ExecutionResult<O>> {
    console.log(`[MockExecutor] Dry-run skill: ${request.skillId}`);
    return this.execute(request);
  }
}
```

### Dry-Run Mode

```typescript
// Dry-run mode simulates execution without side effects
const result = await executor.dryRun({
  skillId: 'order.create',
  input: { items: [...], customer: {...} },
  context: { actor: 'AI_AGENT', orgId: 'org_123', ... },
  options: { dryRun: true }
});

// result.output contains simulated response
// No actual order is created
// No billing is charged
```

### Test Skill

```typescript
// Test skill for runtime validation
const testSkill: SkillDefinition = {
  id: 'test.echo',
  name: 'Echo Test',
  version: '1.0.0',
  description: 'Returns input as output for testing',
  inputs: { message: 'string' },
  outputs: { echo: 'string', timestamp: 'string' },
  permissions: ['test:execute'],
};

// Execute test
const result = await executor.execute({
  skillId: 'test.echo',
  input: { message: 'Hello, WeAI!' },
  context: testContext,
});
// result.output = { echo: 'Hello, WeAI!', timestamp: '2026-01-23T...' }
```

---

## 8️⃣ Repository Structure

```
weai-runtime/
├── README.md                    # 架构说明（最重要）
├── docs/
│   ├── execution-flow.md        # 详细执行流程
│   ├── billing-hooks.md         # 计费钩子说明
│   └── security-model.md        # 安全模型
├── interfaces/
│   ├── executor.ts              # 执行器接口
│   ├── resolver.ts              # 解析器接口
│   ├── meter.ts                 # 计量器接口
│   └── settler.ts               # 结算器接口
├── examples/
│   └── demo-skill-execution.ts  # 示例执行
└── src/
    └── stubs/                   # 非生产实现
        ├── mock-executor.ts
        ├── mock-resolver.ts
        └── mock-meter.ts
```

---

## 9️⃣ Platform Products Using This Runtime

| Product | Repository | Description |
|---------|------------|-------------|
| **DryCleanOne** | `drycleanone` | Service Commerce OS for Clothing Care |
| **LaundryBoxOne** | (planned) | Laundry Service Platform |
| **ShoesCleanOne** | (planned) | Shoe Care Platform |
| **AlterationOne** | (planned) | Alteration & Tailoring Platform |
| **CleanSupply** | (planned) | Cleaning Supply Marketplace |

All products share:
- Same Skill execution model
- Same billing hooks
- Same audit trail
- Same trust guarantees

---

## 🔟 Investor Perspective

When investors see:
- `weai-core` (Public)
- `weai-registry` (Public)
- `weai-runtime` (Public)

They recognize:

> **"This is not a team shipping prompt demos.**
> **This is building the Stripe + Kubernetes for the Agent era."**

### Value Proposition

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   WeAI Platform = Infrastructure for Agent Economy              │
│                                                                 │
│   weai-core      → Standards (like W3C specs)                   │
│   weai-registry  → Discovery (like npm registry)                │
│   weai-runtime   → Execution (like Kubernetes runtime)          │
│                                                                 │
│   + Billing + Trust + Audit = Production-grade Agent Infra      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> ⚠️ **Disclaimer**: This positioning reflects architectural intent, not product parity. We are building toward this vision.

---

## License

© Wembley AI Solutions.

This repository is publicly readable for ecosystem transparency.
Production implementations, connectors, and commercial deployments
are proprietary and maintained in private repositories.

---

## Related Repositories

| Repository | Description | Visibility |
|------------|-------------|------------|
| `weai-core` | Platform type definitions and schemas | Public |
| `weai-registry` | Skill registration and discovery | Public |
| `weai-runtime` | Execution runtime (this repo) | Public |
| `weai-runtime-internal` | Production implementation | Private |
| `drycleanone-skills` | DryCleanOne skill definitions | Private |
| `drycleanone` | DryCleanOne main platform | Private |
