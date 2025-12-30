# Nexus Architecture - System Architecture Overview

## Overview

Nexus implements a sophisticated multi-agent AI system to build production-ready applications. The architecture follows multiple proven methodologies:

- **Hexagonal Architecture** (Ports and Adapters)
- **Clean Architecture** (4 layers)
- **Vertical Slice Architecture** (feature-based)
- **Domain-Driven Design** (DDD)
- **Test-Driven Development** (TDD)
- **Specification Driven Development**
- **Behavior Driven Development** (BDD)
- **Problem Frames Approach** (Michael A. Jackson)

## Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Nexus System Architecture                 │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Multi-Agent Orchestration              │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────┬──────────┐  ┌──────────────┐ │
│  │   Domain        │  │   Clean       │ │
│  │   Architect    │  │   Architecture │ │   TDD         │
│  │   Agent        │  │   Designer    │ │   Generator   │ │
│  └────────────┴──────────┘ └────────────┴──┘ └────────────┴─┘
│                                                              │
│  ┌────────────┬──────────┐ ┌──────────────┐ ┌────────────┐ ┌────────────┐
│  │   Clean Code    │ │   Migration    │ │   Feature     │ │   Code        │
│  │   Implementer  │  │   Planner     │ │   Scaffolder  │  │   Reviewer     │
│  └────────────┴──────────┘ └────────────┴──┘ └────────────┴─┘ └────────────┴─┘
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│         Workflow Engine (Step-by-step execution)         │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
│                                                              │
└─────────────────────────────────────────────────────────┘
```

## Layers

### **Layer 1: Domain Layer** (Inner Circle - Business Logic)

**Purpose**: Core business logic with no external dependencies

**Contains**:
- Entities (business objects with identity)
- Value Objects (immutable, self-validating)
- Domain Services (complex business operations)
- Domain Events (state changes in domain)

**Dependencies**: None

---

### **Layer 2: Use Case Layer** (Second Circle - Application Logic)

**Purpose**: Orchestrate business operations

**Contains**:
- Use Case Interfaces (Input/Output ports)
- Use Case Services (implementations)
- Command/Query separation (CQRS optional)

**Dependencies**: Only on Domain Layer

---

### **Layer 3: Adapters Layer** (Third Circle - External Interfaces)

**Purpose**: Handle external communications

**Contains**:
- **Inbound Adapters**: Controllers (REST, CLI)
- **Outbound Adapters**: Presenters, Repositories

**Dependencies**: Only on Use Case Layer + Domain Layer

---

### **Layer 4: Frameworks Layer** (Outer Circle - Infrastructure)

**Purpose**: Infrastructure implementations

**Contains**:
- Express.js server setup
- Database connections (SQLite, PostgreSQL, MySQL)
- Email providers (SMTP)
- Background job queues

**Dependencies**: None (implementation details only)

## Dependency Rules

```
         Frameworks Layer
              ↑
              ↑
         Adapters Layer
              ↑
              ↑
         Use Cases Layer
              ↑
              ↑
         Domain Layer
              ↑
              ↑
         (No dependencies)
```

## Communication Flow

```
User Request
    ↓
Controller (Adapters Layer)
    ↓
Use Case (Use Cases Layer)
    ↓
Repository (Adapters Layer)
    ↓
Entity (Domain Layer)
```

**Key Principle**: All dependencies point **inward** toward the Domain Layer.

## Project Structure

### **Vertical Slice Architecture** (Feature-Based Organization)

```
nexus/
├── docs/                           # Documentation
├── src/
│   ├── features/                      # Feature modules
│   │   ├── user-management/
│   │   │   ├── domain/              # Layer 1: Domain
│   │   │   │   ├── entities/
│   │   │   │   ├── value-objects/
│   │   │   │   └── domain-services/
│   │   │   ├── use-cases/           # Layer 2: Application
│   │   │   │   ├── port/
│   │   │   │   │   ├── in/      # Input ports (interfaces)
│   │   │   │   │   │   └── out/     # Output ports (interfaces)
│   │   │   │   ├── service/             # Implementations
│   │   │   │   └── port/          # DTOs, POs, Mappers
│   │   │   ├── adapters/             # Layer 3: Interfaces
│   │   │   │   ├── in/         # Controllers (inbound)
│   │   │   │   └── out/       # Presenters, Repositories (outbound)
│   │   │   └── tests/               # Co-located tests
│   │   └── migrations/           # Database migrations
│   └── shared/                        # Cross-cutting concerns
│       ├── domain/                # Shared domain concepts
│       ├── infrastructure/      # Shared infrastructure
│       └── types/                # Shared TypeScript types
└── packages/                          # NPM packages
    ├── cli/                             # Nexus CLI tool
    ├── core/                            # Core engine
    ├── skills/                           # Claude Skills
    └── built-in-features/           # Pre-built features
└── examples/                          # Example projects
```

### **Shared Infrastructure**

```
shared/
├── adapter/
│   └── out/
│       └── repository/           # Repository implementations
├── io/
    ├── express/                 # Express setup
    ├── database/              # Database connections
    └── config/                # Configuration
```

## Data Flow

```
Client Request
    ↓
Controller (creates request)
    ↓
Use Case (orchestrates flow)
    ↓
Input Class (validated by constructor)
    ↓
Mapper (converts Input to Domain)
    ↓
Entity (business logic)
    ↓
Mapper (converts Domain to PO)
    ↓
Repository (persists PO)
    ↓
Repository (returns Entity)
    ↓
Mapper (converts Entity to DTO)
    ↓
Presenter (formats DTO for output)
    ↓
Response (returned to client)
```

## Key Architectural Principles

1. **Dependency Inversion**: High-level modules don't depend on low-level modules
2. **Interface Segregation**: Small, focused interfaces
3. **Single Responsibility**: Each class has one reason to change
4. **Open/Closed Principle**: Open for extension, closed for modification
5. **Liskov Substitution Principle**: Subtypes must be substitutable
6. **Dependency Injection**: Dependencies provided through constructors

## Benefits

- **Testability**: Each layer can be tested independently
- **Maintainability**: Clear separation makes code easier to understand
- **Flexibility**: New adapters can be added without changing core
- **Scalability**: Different technologies can be swapped easily
- **Domain Isolation**: Business logic protected from external changes
- **Parallel Development**: Different teams can work on different features
- **Easy Onboarding**: New developers can focus on one layer

## See Also

- [Vertical Slice vs Layer-Based Structure](./docs/architecture/vertical_slice/VERTICAL_SLICE_STRUCTURE.md)
- [Dependency Rules](./docs/architecture/dependency_rules/DEPENDENCY_RULES.md)
- [Communication Flow](./docs/architecture/communication_flow/COMMUNICATION_FLOW.md)
- [Design Patterns](./docs/engineering/patterns/PATTERNS.md)
