# AI Diagram Generation

> Generate mermaid diagrams from code: architecture views, flowcharts, ER diagrams, sequence diagrams, class diagrams, and more -- all using Claude Code skills, agents, and hooks.

---

## Overview

Diagrams are the fastest way to understand a system. AI can generate them by analyzing code structure, database schemas, API flows, and git history. Mermaid is the ideal target format because it is text-based (version-controllable), renders natively on GitHub, and can be edited by both humans and AI.

### Supported Diagram Types

```mermaid
mindmap
  root((AI Diagram<br/>Generation))
    Architecture
      C4 Context
      C4 Container
      C4 Component
      Deployment
    Flow
      Flowcharts
      State Machines
      Activity Diagrams
    Data
      ER Diagrams
      Class Diagrams
      Schema Maps
    Behavior
      Sequence Diagrams
      Interaction Diagrams
      User Journeys
    DevOps
      CI/CD Pipelines
      Deployment Flows
      Dependency Graphs
    Timeline
      Gantt Charts
      Git Graphs
      Release Timelines
```

---

## Working Skill: Diagram Generator

### Skill File: `~/.claude/skills/diagram_creator.md`

```markdown
# Skill: Mermaid Diagram Generator

## When to activate
Activate when the user asks to:
- Create any kind of diagram
- Visualize code structure, data flow, or architecture
- Generate ER diagrams from database schemas
- Create sequence diagrams from API flows
- Visualize CI/CD pipelines
- Map dependencies

## Instructions

### Diagram Type Selection
Based on what the user wants to visualize, choose the right diagram type:

| Need | Diagram Type | Mermaid Syntax |
|------|-------------|----------------|
| System overview | C4 Context | C4Context |
| Service architecture | C4 Container | C4Container |
| Module structure | C4 Component | C4Component |
| Process flow | Flowchart | graph TD/LR |
| API call sequence | Sequence | sequenceDiagram |
| Database schema | ER | erDiagram |
| Class relationships | Class | classDiagram |
| Object states | State | stateDiagram-v2 |
| User experience | User Journey | journey |
| Project timeline | Gantt | gantt |
| Git branching | Git Graph | gitGraph |
| Decision tree | Flowchart | graph TD |

### Generation Rules
1. Always use GitHub-compatible mermaid syntax
2. Keep diagrams readable:
   - Maximum 15-20 nodes per diagram
   - Split large diagrams into multiple focused views
   - Use subgraphs for logical grouping
   - Use consistent naming conventions
3. Add descriptive labels to all relationships
4. Use appropriate direction (TD for hierarchies, LR for flows)
5. Include a title comment above each diagram
6. For C4 diagrams, use the official C4 mermaid syntax

### Code Analysis Patterns

#### For Flowcharts (from function logic):
1. Find the function/method
2. Trace control flow: conditionals, loops, error paths
3. Map each branch to a decision diamond
4. Map each action to a process box
5. Include error/exception paths

#### For Sequence Diagrams (from API code):
1. Identify the entry point (HTTP handler, event listener)
2. Trace all function calls that cross module boundaries
3. Map external service calls (HTTP, database, cache, queue)
4. Include request and response data types
5. Show async operations with par/alt blocks

#### For ER Diagrams (from database schemas):
1. Read migration files, schema definitions, or ORM models
2. Extract entities and their attributes
3. Identify relationships (FK constraints, join tables)
4. Map cardinality (one-to-one, one-to-many, many-to-many)
5. Include attribute types and constraints

#### For Class Diagrams (from source code):
1. Identify classes, interfaces, and abstract classes
2. Map inheritance (extends) and implementation (implements)
3. Identify composition and aggregation relationships
4. Include key methods and properties (not all -- top 5-7)
5. Show visibility modifiers (+, -, #, ~)

#### For State Diagrams (from business logic):
1. Identify the entity with state (order, user, ticket)
2. Extract all possible states
3. Map transitions with trigger events
4. Include guard conditions
5. Mark initial and final states
```

---

## Diagram Examples from Code

### Flowchart from Function Logic

Given this code:

```javascript
async function processOrder(order) {
  if (!order.items.length) throw new Error('Empty order');

  const inventory = await checkInventory(order.items);
  if (!inventory.available) {
    await notifyBackorder(order);
    return { status: 'backordered' };
  }

  const payment = await processPayment(order);
  if (!payment.success) {
    await refundHold(payment.holdId);
    return { status: 'payment_failed' };
  }

  await reserveInventory(order.items);
  await sendConfirmation(order);
  return { status: 'confirmed' };
}
```

Generated diagram:

```mermaid
flowchart TD
    START([processOrder]) --> VALIDATE{Items exist?}
    VALIDATE -->|No| ERROR[Throw: Empty order]
    VALIDATE -->|Yes| INVENTORY[Check inventory]
    INVENTORY --> AVAILABLE{In stock?}
    AVAILABLE -->|No| BACKORDER[Notify backorder]
    BACKORDER --> RETURN_BO([Return: backordered])
    AVAILABLE -->|Yes| PAYMENT[Process payment]
    PAYMENT --> PAID{Payment success?}
    PAID -->|No| REFUND[Refund hold]
    REFUND --> RETURN_FAIL([Return: payment_failed])
    PAID -->|Yes| RESERVE[Reserve inventory]
    RESERVE --> CONFIRM[Send confirmation]
    CONFIRM --> RETURN_OK([Return: confirmed])

    style ERROR fill:#f96,stroke:#333
    style RETURN_BO fill:#ff9,stroke:#333
    style RETURN_FAIL fill:#f96,stroke:#333
    style RETURN_OK fill:#9f9,stroke:#333
```

### Sequence Diagram from API Flow

Given an authentication endpoint, the AI traces the full call chain:

```mermaid
sequenceDiagram
    actor User
    participant Gateway as API Gateway
    participant Auth as Auth Service
    participant DB as PostgreSQL
    participant Cache as Redis
    participant Email as Email Service

    User->>Gateway: POST /auth/login {email, password}
    Gateway->>Auth: authenticate(email, password)
    Auth->>Cache: get(rate_limit:email)
    Cache-->>Auth: attempts: 2

    alt Too many attempts
        Auth-->>Gateway: 429 Too Many Requests
        Gateway-->>User: 429 Rate Limited
    end

    Auth->>DB: SELECT * FROM users WHERE email = ?
    DB-->>Auth: user record

    alt User not found
        Auth-->>Gateway: 401 Invalid credentials
        Gateway-->>User: 401 Unauthorized
    end

    Auth->>Auth: verify password hash

    alt Invalid password
        Auth->>Cache: incr(rate_limit:email)
        Auth-->>Gateway: 401 Invalid credentials
        Gateway-->>User: 401 Unauthorized
    end

    Auth->>Auth: generate JWT + refresh token
    Auth->>Cache: set(session:userId, token, TTL=24h)
    Auth->>DB: INSERT INTO refresh_tokens(...)

    alt 2FA enabled
        Auth->>Email: send 2FA code
        Auth-->>Gateway: 200 {requiresTwoFactor: true}
        Gateway-->>User: 200 Enter 2FA code
    else No 2FA
        Auth-->>Gateway: 200 {token, refreshToken}
        Gateway-->>User: 200 Authenticated
    end
```

### ER Diagram from Database Schema

Given migration files or ORM models:

```mermaid
erDiagram
    USERS {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar name
        enum role "admin | user | moderator"
        timestamp created_at
        timestamp updated_at
    }

    ORDERS {
        uuid id PK
        uuid user_id FK
        enum status "pending | confirmed | shipped | delivered | cancelled"
        decimal total
        jsonb shipping_address
        timestamp created_at
    }

    ORDER_ITEMS {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
    }

    PRODUCTS {
        uuid id PK
        varchar name
        text description
        decimal price
        int stock_quantity
        uuid category_id FK
    }

    CATEGORIES {
        uuid id PK
        varchar name UK
        uuid parent_id FK "self-referential"
    }

    REVIEWS {
        uuid id PK
        uuid user_id FK
        uuid product_id FK
        int rating "1-5"
        text comment
        timestamp created_at
    }

    USERS ||--o{ ORDERS : places
    ORDERS ||--|{ ORDER_ITEMS : contains
    PRODUCTS ||--o{ ORDER_ITEMS : "included in"
    CATEGORIES ||--o{ PRODUCTS : categorizes
    CATEGORIES ||--o{ CATEGORIES : "parent of"
    USERS ||--o{ REVIEWS : writes
    PRODUCTS ||--o{ REVIEWS : "reviewed by"
```

### State Diagram from Business Logic

```mermaid
stateDiagram-v2
    [*] --> Draft: Create order

    Draft --> Pending: Submit order
    Draft --> Cancelled: Cancel

    Pending --> PaymentProcessing: Begin payment
    Pending --> Cancelled: Cancel / Timeout (30min)

    PaymentProcessing --> Confirmed: Payment success
    PaymentProcessing --> PaymentFailed: Payment declined

    PaymentFailed --> PaymentProcessing: Retry payment
    PaymentFailed --> Cancelled: Max retries (3)

    Confirmed --> Shipped: Ship order
    Confirmed --> Refunded: Full refund

    Shipped --> Delivered: Delivery confirmed
    Shipped --> Returned: Return initiated

    Delivered --> Returned: Return window (30 days)
    Delivered --> [*]: Complete

    Returned --> Refunded: Process refund
    Refunded --> [*]: Complete
    Cancelled --> [*]: Complete

    note right of PaymentProcessing
        Retries: max 3
        Timeout: 5 minutes per attempt
    end note

    note right of Delivered
        Return window: 30 days
    end note
```

### Class Diagram from Source Code

```mermaid
classDiagram
    class BaseRepository~T~ {
        <<abstract>>
        #db: Database
        +findById(id: string) T
        +findAll(filter: Filter) T[]
        +create(data: Partial~T~) T
        +update(id: string, data: Partial~T~) T
        +delete(id: string) boolean
    }

    class UserRepository {
        +findByEmail(email: string) User
        +findByRole(role: Role) User[]
        +updatePassword(id: string, hash: string) void
    }

    class OrderRepository {
        +findByUser(userId: string) Order[]
        +findByStatus(status: OrderStatus) Order[]
        +getRevenue(dateRange: DateRange) number
    }

    class UserService {
        -userRepo: UserRepository
        -authService: AuthService
        +register(data: RegisterDTO) User
        +login(credentials: LoginDTO) AuthToken
        +updateProfile(id: string, data: ProfileDTO) User
    }

    class OrderService {
        -orderRepo: OrderRepository
        -paymentService: PaymentService
        -inventoryService: InventoryService
        +createOrder(userId: string, items: CartItem[]) Order
        +cancelOrder(orderId: string) Order
        +processRefund(orderId: string) Refund
    }

    BaseRepository~T~ <|-- UserRepository
    BaseRepository~T~ <|-- OrderRepository
    UserService --> UserRepository : uses
    UserService --> AuthService : uses
    OrderService --> OrderRepository : uses
    OrderService --> PaymentService : uses
    OrderService --> InventoryService : uses
```

### CI/CD Pipeline Visualization

```mermaid
graph LR
    subgraph "Trigger"
        PUSH[Push to main]
        PR[Pull Request]
    end

    subgraph "Build"
        LINT[Lint & Format]
        TYPECHECK[Type Check]
        UNIT[Unit Tests]
        BUILD[Build]
    end

    subgraph "Test"
        INTEGRATION[Integration Tests]
        E2E[E2E Tests]
        SECURITY[Security Scan]
    end

    subgraph "Deploy"
        STAGING[Deploy Staging]
        SMOKE[Smoke Tests]
        APPROVE{Manual Approval}
        PROD[Deploy Production]
        MONITOR[Health Check]
    end

    PUSH --> LINT
    PR --> LINT
    LINT --> TYPECHECK
    TYPECHECK --> UNIT
    UNIT --> BUILD

    BUILD --> INTEGRATION
    INTEGRATION --> E2E
    E2E --> SECURITY

    SECURITY --> STAGING
    STAGING --> SMOKE
    SMOKE --> APPROVE
    APPROVE -->|Approved| PROD
    APPROVE -->|Rejected| ROLLBACK[Rollback]
    PROD --> MONITOR
    MONITOR -->|Unhealthy| ROLLBACK

    style PUSH fill:#4a9eff
    style PR fill:#4a9eff
    style PROD fill:#2ea44f
    style ROLLBACK fill:#f96
```

### Dependency Graph

```mermaid
graph TD
    subgraph "Frontend"
        UI[ui-components]
        WEB[web-app]
        ADMIN[admin-panel]
    end

    subgraph "Backend"
        API[api-gateway]
        AUTH[auth-service]
        ORDERS[order-service]
        INVENTORY[inventory-service]
        NOTIFY[notification-service]
    end

    subgraph "Shared"
        TYPES[shared-types]
        UTILS[shared-utils]
        CONFIG[shared-config]
    end

    subgraph "Infrastructure"
        DB[(PostgreSQL)]
        CACHE[(Redis)]
        QUEUE[(RabbitMQ)]
        S3[(S3)]
    end

    WEB --> UI
    ADMIN --> UI
    WEB --> API
    ADMIN --> API

    API --> AUTH
    API --> ORDERS
    API --> INVENTORY

    ORDERS --> NOTIFY
    INVENTORY --> NOTIFY

    AUTH --> TYPES
    ORDERS --> TYPES
    ORDERS --> UTILS
    INVENTORY --> UTILS
    API --> CONFIG

    AUTH --> DB
    AUTH --> CACHE
    ORDERS --> DB
    ORDERS --> QUEUE
    INVENTORY --> DB
    NOTIFY --> QUEUE
    NOTIFY --> S3
```

---

## Working Agent: Diagram Audit Agent

```markdown
# Agent: Diagram Freshness Auditor

## Role
You are a diagram audit agent. Your job is to compare existing mermaid
diagrams against the current codebase and flag discrepancies.

## Trigger
Run weekly, or when user asks: claude "Audit diagrams for accuracy"

## Process

### 1. Inventory
- Find all .md files containing mermaid code blocks
- Catalog each diagram by type and what it documents
- Build a map: diagram -> source code it represents

### 2. Comparison
For each diagram:
- Re-analyze the source code it represents
- Generate a fresh version of the diagram
- Diff the existing vs fresh diagram
- Classify differences:
  - STALE: Code changed, diagram didn't
  - EXTRA: Diagram shows something not in code
  - MISSING: Code has something not in diagram
  - STYLE: Diagram is correct but could be clearer

### 3. Report
Generate `docs/diagrams/audit-report.md`:

| Diagram | Location | Status | Issues |
|---------|----------|--------|--------|
| System Context | docs/arch/context.md | CURRENT | None |
| ER Diagram | docs/data/schema.md | STALE | Missing: reviews table |
| Auth Flow | docs/api/auth.md | STALE | OAuth2 flow not shown |

### 4. Auto-Fix (with user approval)
For each STALE diagram:
- Show the diff between existing and proposed
- Ask user to approve the update
- Apply approved changes
```

---

## Working Hooks

### Post-Tool-Use: Auto-Generate Diagram on Schema Change

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "tool": "edit",
          "path": "**/migrations/**"
        },
        "command": "bash -c 'echo {\"ok\": true, \"message\": \"Database migration changed. Consider regenerating ER diagram: claude \\\"Update ER diagram from current migrations\\\"\"}'",
        "description": "Suggest ER diagram update when migrations change"
      },
      {
        "matcher": {
          "tool": "edit",
          "path": "**/routes/**"
        },
        "command": "bash -c 'echo {\"ok\": true, \"message\": \"Route changed. Consider updating sequence diagrams for affected endpoints.\"}'",
        "description": "Suggest sequence diagram update when routes change"
      }
    ]
  }
}
```

### Pre-Push: Diagram Validation

```json
{
  "hooks": {
    "PrePush": [
      {
        "command": "bash -c 'python3 .claude/hooks/validate_mermaid.py'",
        "description": "Validate all mermaid diagrams in staged files"
      }
    ]
  }
}
```

**Hook Script: `.claude/hooks/validate_mermaid.py`**

```python
#!/usr/bin/env python3
"""
Pre-push hook: Validate mermaid syntax in all markdown files
that were changed. Uses mermaid-cli (mmdc) if available, otherwise
does basic syntax checks.
"""
import subprocess
import re
import sys
import json
import os
import tempfile

MERMAID_BLOCK = re.compile(r'```mermaid\n(.*?)```', re.DOTALL)

def get_changed_md_files():
    result = subprocess.run(
        ["git", "diff", "--name-only", "HEAD~1", "--", "*.md"],
        capture_output=True, text=True
    )
    return [f for f in result.stdout.strip().split("\n") if f.endswith(".md")]

def has_mmdc():
    """Check if mermaid-cli is installed."""
    try:
        subprocess.run(["mmdc", "--version"], capture_output=True)
        return True
    except FileNotFoundError:
        return False

def validate_with_mmdc(mermaid_code, filepath, index):
    """Validate using mermaid-cli."""
    with tempfile.NamedTemporaryFile(suffix=".mmd", mode="w", delete=False) as f:
        f.write(mermaid_code)
        f.flush()
        result = subprocess.run(
            ["mmdc", "-i", f.name, "-o", "/dev/null"],
            capture_output=True, text=True
        )
        os.unlink(f.name)
        if result.returncode != 0:
            return f"  {filepath} diagram #{index}: INVALID\n    {result.stderr.strip()}"
    return None

def basic_validate(mermaid_code, filepath, index):
    """Basic syntax check without mmdc."""
    valid_starts = [
        "graph ", "flowchart ", "sequenceDiagram", "classDiagram",
        "stateDiagram", "erDiagram", "journey", "gantt", "pie",
        "gitGraph", "C4Context", "C4Container", "C4Component",
        "mindmap", "timeline", "quadrantChart", "sankey",
    ]
    first_line = mermaid_code.strip().split("\n")[0].strip()
    if not any(first_line.startswith(s) for s in valid_starts):
        return f"  {filepath} diagram #{index}: Unknown diagram type: {first_line}"
    return None

def main():
    files = get_changed_md_files()
    if not files:
        print(json.dumps({"ok": True}))
        return

    use_mmdc = has_mmdc()
    errors = []

    for filepath in files:
        if not os.path.isfile(filepath):
            continue
        with open(filepath) as f:
            content = f.read()

        blocks = MERMAID_BLOCK.findall(content)
        for i, block in enumerate(blocks, 1):
            if use_mmdc:
                err = validate_with_mmdc(block, filepath, i)
            else:
                err = basic_validate(block, filepath, i)
            if err:
                errors.append(err)

    if errors:
        print("Mermaid diagram validation errors:")
        print("\n".join(errors))
        print(json.dumps({"ok": False, "reason": "Invalid mermaid diagrams found"}))
        sys.exit(1)

    print(f"Validated {sum(len(MERMAID_BLOCK.findall(open(f).read())) for f in files if os.path.isfile(f))} diagrams in {len(files)} files")
    print(json.dumps({"ok": True}))

if __name__ == "__main__":
    main()
```

---

## Best Practices for AI Diagram Generation

### 1. One Concept Per Diagram
Do not cram everything into one diagram. Split by concern:
- System context: who uses the system and what external systems exist
- Container: what are the deployable units
- Component: what modules exist inside a container
- Data flow: how data moves for a specific use case

### 2. Consistent Styling
Use a project-level style convention:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'primaryColor': '#4a9eff',
  'primaryTextColor': '#fff',
  'primaryBorderColor': '#2a7fff',
  'lineColor': '#666',
  'secondaryColor': '#f4f4f4',
  'tertiaryColor': '#fff'
}}}%%
graph TD
    A[Styled Node] --> B[Another Node]
```

### 3. Version Control Diagrams
- Store diagrams as mermaid code in markdown (not images)
- Diffs are meaningful and reviewable
- CI can validate syntax
- AI can update them automatically

### 4. Link Diagrams to Code
Add comments in diagrams referencing source files:

```
%% Source: src/services/order-service.ts
%% Last generated: 2026-03-22
```

### 5. Progressive Detail
Start with the highest level and let readers drill down:
- Level 1: System Context (README.md)
- Level 2: Container diagram (docs/architecture/)
- Level 3: Component diagrams (docs/architecture/components/)
- Level 4: Code-level diagrams (inline in source docs)

---

## Sources

- [Mermaid.js: Generation of Diagrams from Text](https://mermaid.js.org/)
- [Mermaid Chart: AI-Powered Diagram Generation](https://www.mermaidchart.com/mermaid-ai)
- [Eraser.io: AI Mermaid Diagram Editor](https://www.eraser.io/ai/mermaid-diagram-editor)
- [Starmorph: Mermaid.js Tutorial - Complete Guide 2026](https://blog.starmorph.com/blog/mermaid-js-tutorial)
- [W3Resource: Mermaid.js Guide 2026](https://www.w3resource.com/javascript/mermaid-js-guide-to-create-diagrams-as-code.php)
- [Obsibrain: Mermaid Diagram Complete Guide 2026](https://www.obsibrain.com/blog/mermaid-diagram-a-complete-guide-to-diagrams-as-code-in-2026)
- [Claude Code Hooks Guide](https://code.claude.com/docs/en/hooks-guide)
