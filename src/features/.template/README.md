# Feature Template Structure

This template shows the standard structure for each feature in Nexus.

## Directory Structure

```
feature-name/
├── domain/                          # Layer 1: Domain (Business Logic)
│   ├── entities/                     # Domain entities
│   ├── value-objects/               # Value objects (immutable)
│   └── domain-services/            # Domain services
├── usecases/                        # Layer 2: Use Cases (Application Logic)
│   ├── port/
│   │   ├── in/                      # Input ports (use case interfaces)
│   │   │   └── use-case-name/
│   │   │       ├── UseCaseNameUseCase.ts
│   │   │       ├── UseCaseNameInput.ts
│   │   │       └── UseCaseNameOutput.ts
│   │   └── out/                     # Output ports (presenter interfaces)
│   ├── service/                      # Use case implementations
│   ├── port/
│   │   ├── dto/                      # Data Transfer Objects
│   │   ├── po/                      # Persistent Objects
│   │   └── mapper/                   # Mappers (DTO ↔ PO ↔ Domain)
├── adapters/                         # Layer 3: Adapters (External Interfaces)
│   ├── in/                           # Inbound adapters
│   │   ├── controller/console/         # CLI controllers
│   │   ├── controller/web/            # REST controllers
│   │   └── controller/cli/           # CLI command handlers
│   └── out/                          # Outbound adapters
│       ├── presenter/                  # Output formatters
│       └── repository/                # Repository implementations
├── tests/                           # Co-located tests
│   ├── unit/
│   │   ├── domain/
│   │   ├── usecases/
│   │   └── adapters/
│   ├── integration/
│   └── e2e/
└── migrations/                       # Database migrations for this feature
```

## Clean Architecture Layers

1. **Domain Layer** (Inner Circle)
   - No external dependencies
   - Contains business logic only
   - Entities, value objects, domain services

2. **Use Cases Layer** (Second Circle)
   - Orchestrate business operations
   - Depends only on Domain Layer
   - Input/Output ports, services, DTOs, POs, mappers

3. **Adapters Layer** (Third Circle)
   - Handle external communications
   - Depends on Use Cases + Domain Layers
   - Controllers, presenters, repositories

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

**All dependencies point inward toward Domain Layer.**

## Creating a New Feature

1. Copy this `.template` folder to a new feature name
2. Rename files according to your feature
3. Implement following TDD: Tests → Code
4. Follow Clean Architecture dependency rules

## Example: User Feature

```
user/
├── domain/
│   ├── entities/
│   │   └── User.ts
│   ├── value-objects/
│   │   ├── Email.ts
│   │   └── UserId.ts
├── usecases/
│   ├── port/in/
│   │   └── create-user/
│   │       ├── CreateUserUseCase.ts
│   │       └── CreateUserInput.ts
│   ├── service/
│   │   └── CreateUserService.ts
│   └── port/
│       ├── dto/
│       │   └── UserDto.ts
│       └── mapper/
│           └── UserMapper.ts
└── adapters/
    ├── in/
    │   └── controller/cli/
    │       └── CreateUserCLIController.ts
    └── out/
        └── repository/
            └── UserRepository.ts
```

## References

- [docs/architecture/ARCHITECTURE.md](../../../docs/architecture/ARCHITECTURE.md)
- [docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md](../../../docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md)
- [FOLDER_STRUCTURE.md](../../../FOLDER_STRUCTURE.md)
