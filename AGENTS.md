# Nexus AI Agents

## Overview

**Nexus** is an AI-powered code generation system that uses multiple specialized AI agents to build production-ready applications following best practices. It combines Claude Skills framework with a sophisticated agent orchestration system to generate clean, maintainable code.

## Core Philosophy

Nexus operates on these fundamental principles, combining multiple software engineering methodologies:

### Development Methodologies

1. **Human-in-the-Loop**: Every step is reviewed and approved by humans before integration
2. **Specification Driven Development**: Specifications drive implementation, not the reverse
3. **Small Steps**: Changes are broken down into <150 lines, reviewable in <5 minutes
4. **Tests First**: TDD workflow ensures code quality from start (RED → GREEN → REFACTOR)
5. **Behavior Driven Development**: Features are defined by behavior, not implementation details
6. **Executable Specifications**: Tests serve as living documentation and executable specifications
7. **Living Documentation**: Documentation evolves with the codebase, always stays current

### Architecture & Design

8. **Clean Architecture**: 4-layer separation (Domain → Use Cases → Adapters → Frameworks)
9. **Domain-Driven Design**: Business logic is the center, not database tables or frameworks
10. **Vertical Slice Architecture**: Folder structure organized by features, not technical layers
11. **Design by Contract**: Interfaces specify preconditions, postconditions, and invariants
12. **Pattern Language**: Consistent patterns and idioms across the codebase
13. **Problem Frames Approach**: Software development as problem framing, not just problem solving

### Domain Modeling

14. **Domain Event Storming**: Collaborative session to discover domain events and aggregates
15. **Domain Event Storytelling**: Narrative approach to understanding domain event flows
16. **Specification by Example**: Concrete examples clarify abstract requirements

### Quality & Standards

17. **Type Safety**: Strict TypeScript with zero `any` types
18. **Strict Code Rules**: Functions <15 lines, max 3 parameters, descriptive names
19. **Clean Code Principles**: Code that is easy to read, understand, and maintain

### Optional Advanced Patterns

20. **Event Sourcing** (Optional): Store state changes as sequence of events
21. **CQRS** (Optional): Command Query Responsibility Segregation for complex domains

## AI Agent Workflow

### Before Starting Work

**1. Read PROGRESS.md** to find current step:
```bash
Read file: PROGRESS.md
```

**2. Identify what needs to be done** based on step status:
- ✅ Complete → Already done, skip
- 🟡 In Progress → Continue working on this step
- [ ] Not Started → Start working on this step
- [~] Partially Complete → Finish remaining parts

**3. Update step status** to "In Progress":
```markdown
- [🟡] **3. Configure Vitest for testing**
  - Started: 2025-12-30
  - Status: In Progress
```

### During Work

**Find relevant documentation** using this guide:

| When you need... | Read this file... |
|-----------------|-------------------|
| Architecture decisions | docs/architecture/ARCHITECTURE.md |
| Clean Architecture | docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md |
| Domain-Driven Design | docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md |
| Test-Driven Development | docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md |
| Testing conventions | docs/engineering/conventions/TESTING_CONVENTIONS.md |
| Naming conventions | docs/engineering/conventions/NAMING_CONVENTIONS.md |
| SOLID principles | docs/engineering/conventions/SOLID_PRINCIPLES.md |
| Development guidelines | docs/engineering/development/DEVELOPMENT_GUIDELINES.md |
| Git workflow | docs/engineering/development/GIT_VERSION_CONTROL.md |
| Design patterns | docs/engineering/patterns/PATTERNS.md |
| Pattern Language | docs/engineering/pattern_language/PATTERN_LANGUAGE.md |

**Use Grep tool** to find specific patterns:
```
Search for: "Repository" → Look for pattern implementations
Search for: "test" → Look for testing guidelines
```

### After Completing Step

**1. Update PROGRESS.md** with completion status:

```markdown
- [x] **3. Configure Vitest for testing** ✅
  - Completed: 2025-12-30
  - Commit: a1b2c3d
  - Notes: Created vitest.config.ts with coverage settings
```

**2. Update summary statistics** at top of PROGRESS.md:
- Update overall progress percentage
- Update milestone completion percentage
- Update statistics table

**3. Add entry to Recent Activity table**:
```markdown
| 2025-12-30 | 3 | Configure Vitest for testing | ✅ Complete |
```

**4. Update "Next Steps"** section if needed

### Git Commit Requirements

Always follow commit message format from [docs/engineering/development/GIT_VERSION_CONTROL.md](docs/engineering/development/GIT_VERSION_CONTROL.md):

```bash
# Format: <type>[optional scope]: <description>

# Examples:
feat(test): configure Vitest with coverage settings
fix(auth): handle null email validation
docs(readme): update installation instructions
```

**Before committing**:
- ✅ All tests passing: `yarn test`
- ✅ Typecheck passing: `yarn typecheck`
- ✅ Linting passing: `yarn lint`
- ✅ Code formatted: `yarn format`
- ✅ Commit message follows conventional commits

---

## Agent Architecture

Nexus uses a multi-agent system where specialized agents collaborate to build applications:

```
┌─────────────────────────────────────────────────────────┐
│                  Agent Orchestrator                    │
│              (Coordinates all agents)                   │
└──────────────┬──────────────┬────────────────────────┘
               │              │
     ┌──────────▼──────┐  ┌──▼────────────┐
     │   Domain        │  │   Clean       │
     │   Architect    │  │   Architecture │
     │   Agent        │  │   Designer    │
     └─────────────────┘  └───────────────┘
               │              │
     ┌──────────▼──────┐  ┌──▼────────────┐
     │   TDD          │  │   Clean Code  │
     │   Generator    │  │   Implementer │
     └─────────────────┘  └───────────────┘
               │              │
     ┌──────────▼──────┐  ┌──▼────────────┐
     │   Migration     │  │   Code        │
     │   Planner      │  │   Reviewer    │
     └─────────────────┘  └───────────────┘
               │
     ┌──────────▼──────┐
     │   Feature       │
     │   Scaffolder   │
     └─────────────────┘
```

## Core Agents

### 1. Domain Architect Agent

**Purpose**: Analyze requirements and create Domain Model using Domain-Driven Design principles, including Problem Frames Approach and Domain Event Storming.

See [docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md](docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md) and [docs/engineering/problem_frames/PROBLEM_FRAMES.md](docs/engineering/problem_frames/PROBLEM_FRAMES.md)

### 2. Clean Architecture Designer Agent

**Purpose**: Map domain model to Clean Architecture layers with strict dependency rules, following Vertical Slice Architecture and Design by Contract principles.

See [docs/architecture/ARCHITECTURE.md](docs/architecture/ARCHITECTURE.md) and [docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md](docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md)

### 3. TDD Generator Agent

**Purpose**: Generate comprehensive tests first following Test-Driven Development principles, incorporating Behavior Driven Development, Specification by Example, and Executable Specifications.

See [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md) and [docs/engineering/conventions/TESTING_CONVENTIONS.md](docs/engineering/conventions/TESTING_CONVENTIONS.md)

### 4. Clean Code Implementer Agent

**Purpose**: Generate implementation code that passes all generated tests while adhering to strict clean code principles and Design Patterns, following a consistent Pattern Language.

See [docs/engineering/patterns/PATTERNS.md](docs/engineering/patterns/PATTERNS.md)

### 5. Migration Planner Agent

**Purpose**: Automatically generate database migration plans from schema changes.

### 6. Feature Scaffolder Agent

**Purpose**: Generate new features using pre-built templates following best practices.

### 7. Feature Planner Agent

**Purpose**: Break down complex features into small, reviewable steps using Behavior Driven Development and Impact Mapping.

See [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) and [docs/engineering/example_mapping/EXAMPLE_MAPPING.md](docs/engineering/example_mapping/EXAMPLE_MAPPING.md)

### 8. Code Reviewer Agent

**Purpose**: Review generated code against all strict rules before human review.

## Workflow Engine

The Workflow Engine orchestrates all agents in a step-by-step process:

```
┌─────────────────────────────────────────────────────────┐
│ 1. Plan Feature                                       │
│    Feature Planner → Break into steps                   │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ 2. For Each Step:                                    │
│                                                      │
│    a) Domain Architect → Domain Model                 │
│       ↓                                              │
│    b) Clean Architecture Designer → Layer Design       │
│       ↓                                              │
│    c) TDD Generator → Generate Tests (RED)            │
│       ↓                                              │
│    d) Clean Code Implementer → Generate Code (GREEN)  │
│       ↓                                              │
│    e) Run Tests → Check if passing                    │
│       ↓                                              │
│    f) Code Reviewer → Review against rules            │
│       ↓                                              │
│    g) Human Review → Approve/Reject                  │
│       ↓                                              │
│    h) Schema Change? → Migration Planner              │
│                                                      │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ 3. Complete Feature                                   │
│    All tests passing, code reviewed                    │
└─────────────────────────────────────────────────────────┘
```

## Step Generation Limits

Each step is strictly limited to ensure easy review:

### Per-Step Limits
- **Max files changed**: 3 files
- **Max lines added**: 150 lines total
- **Max files created**: 2 files
- **Max test files**: 2 files
- **Review time target**: < 5 minutes

### Step Approval Checklist
Each step must include:
- ✅ Files changed list with line counts
- ✅ Diff preview (max 50 lines shown)
- ✅ All tests passing (including new tests)
- ✅ Code quality checklist verified
- ✅ Rollback instructions
- ✅ Explanation of changes

## AI Provider Support

Nexus supports multiple AI providers with automatic fallback:

### Supported Providers

1. **Anthropic** (Claude models)
   - claude-3-5-sonnet-20241022
   - claude-3-opus-20240229
   - claude-3-haiku-20240307

2. **OpenAI** (GPT models)
   - gpt-4-turbo
   - gpt-4
   - gpt-3.5-turbo

3. **Ollama** (Local AI)
   - llama3.2
   - mistral
   - codellama

### Provider Configuration

```typescript
const providerConfig: AIProvider = {
  type: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-5-sonnet-20241022',
  maxTokens: 4096,
  fallback: {
    type: 'ollama',
    baseURL: 'http://localhost:11434',
    model: 'llama3.2'
  }
};
```

## Built-in Features

Nexus includes pre-built, production-ready features:

### 1. User Management
- User CRUD operations
- Email validation
- Password hashing
- Role assignment

### 2. JWT Authentication
- Token generation and validation
- Token refresh mechanism
- "Remember me" functionality
- Logout handling

### 3. Email Verification
- Verification token generation
- Email sending (SMTP)
- Token validation
- Account activation

### 4. Password Reset
- Reset token generation
- Email notification
- Token validation
- Password update

### 5. Permissions (RBAC)
- Role management
- Permission management
- Role-permission assignment
- Access control middleware

### 6. Email Notifications
- SMTP provider
- Email templates
- Background email queue
- Delivery tracking

### 7. Background Jobs
- In-memory queue (development)
- Redis/Bull queue (production)
- Job scheduling (cron)
- Job retry policy

### 8. File Upload/Download
- Local storage
- AWS S3 integration
- File validation
- Streaming support

## Frontend Support

Nexus can generate frontend code for multiple frameworks:

### Supported Frameworks (Priority Order)
1. Vue 3 + Vite
2. React 18 + Vite
3. Svelte 4 + Vite
4. SolidJS + Vite
5. Angular 17+
6. Vanilla JS + TypeScript

### Frontend Features
- API client (auto-generated from backend)
- Type-safe interfaces (shared TypeScript types)
- Authentication flows (login, register, logout)
- State management (Pinia/Zustand/stores)
- Routing configuration
- Component scaffolding templates

## Database Support

Nexus supports multiple databases with a type-safe query builder (no ORM):

### Phase 1: SQLite (Development & Testing)
- Embedded, zero-config
- Perfect for rapid development
- Full feature set

### Phase 2: PostgreSQL (Production)
- Production-ready
- Full feature parity
- Connection pooling
- Advanced features (JSONB, arrays)

### Phase 3: MySQL (Alternative Production)
- Popular alternative
- Full feature support
- Query builder compatibility

## CLI Usage

```bash
# Initialize new project
nexus init my-api

# Scaffold built-in feature
nexus scaffold user-management

# Generate feature plan
nexus plan "Add shopping cart"

# Execute step-by-step with review
nexus execute --review-each-step

# Generate migrations
nexus migrate:generate

# Run migrations
nexus migrate:up

# Run tests
nexus test

# Check status
nexus status

# Add frontend
nexus frontend:init --framework=vue
```

## Claude Skills Framework

Nexus uses the Claude Skills framework to package agent capabilities:

### Skill Structure
```
.claude/skills/skill-name/
├── SKILL.md           # YAML frontmatter + instructions
├── handler.ts         # Skill execution handler
├── scripts/           # Executable scripts
├── references/        # Documentation
└── assets/            # Templates, resources
```

### SKILL.md Format
```yaml
---
name: domain-architect
description: Analyze requirements and create DDD domain model
allowed-tools: Read, Write, Edit, Bash
---

# Domain Architect

## Purpose
Analyze requirements and create Domain Model using DDD principles.

## Instructions
1. Identify bounded contexts
2. Define entities, value objects, aggregates
3. Map domain events
4. Create domain service contracts

## Examples
Input: "Blog with posts and comments"
Output:
  - Blog Context
    - Entity: Post (id, title, content)
    - Entity: Comment (id, content, postId)
```

## Testing Philosophy

### Executable Specifications
Tests serve as **living documentation** - executable specifications that:
- Define expected behavior of system
- Run automatically to verify correctness
- Provide examples of how to use the code
- Keep documentation synchronized with implementation
- Serve as regression tests

See [docs/engineering/living_documentation/LIVING_DOCUMENTATION.md](docs/engineering/living_documentation/LIVING_DOCUMENTATION.md)

### Living Documentation
Documentation evolves with codebase, always stays current:
- Tests ARE documentation
- README files reference specific test suites
- API docs generated from JSDoc + test examples
- No outdated documentation possible

### Specification by Example
Use concrete examples to clarify behavior:
- Edge cases demonstrated with test cases
- Valid and invalid inputs shown
- Expected outputs clearly defined
- Integration scenarios covered

See [docs/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md](docs/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md)

## Success Metrics

### Code Quality
- ✅ 85%+ test coverage
- ✅ 0 `any` types (unless justified)
- ✅ Functions < 15 lines
- ✅ Cyclomatic complexity < 5

### Developer Experience
- ✅ Each step reviewable in < 10 minutes
- ✅ Clear rollback paths
- ✅ Human-readable at glance
- ✅ Minimal manual intervention

### System Reliability
- ✅ All tests pass before each step
- ✅ Migrations reversible
- ✅ Zero data loss in migrations
- ✅ Clear error messages

## AI Agent Development Guidelines

**CRITICAL**: All AI agents MUST read and follow the mandatory guidelines in **[CODING_AGENTS.md](CODING_AGENTS.md)** before writing any code.

These guidelines include:
- Architecture adherence rules
- Pattern implementation checklists
- Naming conventions (STRICT ENFORCEMENT)
- Code style rules
- Testing requirements
- Validation rules
- Prohibited actions
- Feature isolation rules
- Decision tree for AI agents

## Documentation Structure

```
nexus/
├── README.md                           # Project overview
├── CODING_AGENTS.md                     # AI agent development guidelines (MANDATORY)
├── docs/
│   ├── architecture/
│   │   └── ARCHITECTURE.md            # System architecture
│   └── engineering/
│       ├── METHODOLOGIES.md             # All methodologies overview
│       ├── patterns/
│       │   └── PATTERNS.md            # All design patterns
│       ├── conventions/
│       │   ├── SOLID_PRINCIPLES.md     # SOLID principles
│       │   ├── NAMING_CONVENTIONS.md   # Naming conventions
│       │   └── TESTING_CONVENTIONS.md  # Testing conventions
│       ├── development/
│       │   ├── DEVELOPMENT_GUIDELINES.md # Development best practices
│       │   └── GIT_VERSION_CONTROL.md  # Git workflow and commit standards
│       ├── problem_frames/
│       │   └── PROBLEM_FRAMES.md       # Problem Frames Approach
│       ├── specification_driven/
│       │   └── SPECIFICATION_DRIVEN_DEVELOPMENT.md
│       ├── domain_driven/
│       │   └── DOMAIN_DRIVEN_DEVELOPMENT.md
│       ├── clean_architecture/
│       │   └── CLEAN_ARCHITECTURE.md
│       ├── test_driven/
│       │   └── TEST_DRIVEN_DEVELOPMENT.md
│       ├── behavior_driven/
│       │   └── BEHAVIOR_DRIVEN_DEVELOPMENT.md
│       ├── living_documentation/
│       │   └── LIVING_DOCUMENTATION.md
│       ├── specification_by_example/
│       │   └── SPECIFICATION_BY_EXAMPLE.md
│       ├── example_mapping/
│       │   └── EXAMPLE_MAPPING.md
│       ├── design_by_contract/
│       │   └── DESIGN_BY_CONTRACT.md
│       ├── event_sourcing/
│       │   └── EVENT_SOURCING.md
│       ├── cqrs/
│       │   └── CQRS.md
│       └── pattern_language/
│           └── PATTERN_LANGUAGE.md
```

## References

### Core Documentation
- [README.md](README.md) - Project overview and quick start
- [CODING_AGENTS.md](CODING_AGENTS.md) - MANDATORY AI agent guidelines
- [PROGRESS.md](PROGRESS.md) - **⭐ TRACK PROGRESS HERE** - Update this file after each step
- [docs/architecture/ARCHITECTURE.md](docs/architecture/ARCHITECTURE.md) - System architecture

### Engineering Documentation
- [docs/engineering/METHODOLOGIES.md](docs/engineering/METHODOLOGIES.md) - All methodologies overview
- [docs/engineering/patterns/PATTERNS.md](docs/engineering/patterns/PATTERNS.md) - Design patterns

### Conventions
- [docs/engineering/conventions/SOLID_PRINCIPLES.md](docs/engineering/conventions/SOLID_PRINCIPLES.md) - SOLID principles
- [docs/engineering/conventions/NAMING_CONVENTIONS.md](docs/engineering/conventions/NAMING_CONVENTIONS.md) - Naming conventions
- [docs/engineering/conventions/TESTING_CONVENTIONS.md](docs/engineering/conventions/TESTING_CONVENTIONS.md) - Testing conventions
- [docs/engineering/development/DEVELOPMENT_GUIDELINES.md](docs/engineering/development/DEVELOPMENT_GUIDELINES.md) - Development best practices
- [docs/engineering/development/GIT_VERSION_CONTROL.md](docs/engineering/development/GIT_VERSION_CONTROL.md) - Git workflow and commit message standards

## License

Nexus is released under **MIT License**, allowing free use, modification, and distribution in both personal and commercial projects.

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## Resources

- [Documentation](./docs/)
- [Getting Started](./docs/getting-started/quickstart.md)
- [Architecture Guide](./docs/architecture/overview.md)

## Support

- GitHub Issues: [Report bugs or request features](https://github.com/your-org/nexus/issues)
- Discussions: [Ask questions and share ideas](https://github.com/your-org/nexus/discussions)

---

**Nexus: Build production-ready applications with AI, following best practices, one small step at a time.**
