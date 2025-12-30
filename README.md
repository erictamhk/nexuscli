# Nexus AI Agent System - Project Overview

## About Nexus

**Nexus** is an AI-powered code generation system that uses multiple specialized AI agents to build production-ready applications following best practices. It combines Claude Skills framework with a sophisticated agent orchestration system.

## Vision

Build production-ready applications with AI assistance, following rigorous software engineering methodologies, one small step at a time.

## What Makes Nexus Different

Nexus isn't just another code generator. It's a comprehensive system that:

- **Enforces 30+ software engineering methodologies** through AI agents
- **Breaks work into small, reviewable steps** (<150 lines, <5 minutes each)
- **Generates clean, maintainable code** following strict rules
- **Supports human-in-the-loop development** with approval process
- **Provides built-in features** (auth, users, permissions, etc.)
- **Supports multiple AI providers** (Anthropic, OpenAI, Ollama)
- **Generates both backend and frontend** (Vue, React, Svelte, Solid, Angular, Vanilla)

## Key Characteristics

✅ **Step-by-step generation** with human review after each step
✅ **Tests first** (TDD) - all code starts with failing tests
✅ **Clean architecture** (4 layers with strict dependency rules)
✅ **Domain-driven design** (bounded contexts, ubiquitous language)
✅ **Vertical slice architecture** (feature-based organization)
✅ **Design by contract** (preconditions, postconditions, invariants)
✅ **Pattern language** (consistent patterns across codebase)
✅ **All 23+ design patterns** (Entity, Repository, Factory, etc.)
✅ **All 7 SOLID principles** (SRP, ISP, DIP)
✅ **Development methodologies** (Specification Driven, TDD, BDD, etc.)
✅ **Problem Frames Approach** (Michael A. Jackson's methodology)

## How It Works

1. **Feature Planner Agent** breaks feature into 5-minute reviewable chunks
2. **Domain Architect Agent** analyzes requirements and creates DDD domain model
3. **Clean Architecture Designer Agent** maps domain to 4-layer structure
4. **TDD Generator Agent** generates tests first (RED phase)
5. **Clean Code Implementer Agent** generates code to pass tests (GREEN phase)
6. **Code Reviewer Agent** reviews against strict quality rules
7. **Human Review**: You review each 5-minute step
8. **Migration Planner Agent** auto-generates database migrations
9. **Feature Scaffolder Agent** scaffolds built-in features

## Technology Stack

- **Language**: TypeScript 5.3 (strict mode)
- **Runtime**: Node.js 18+ / 20+
- **Package Manager**: Yarn 4.x (workspaces)
- **Backend**: Express.js + Vitest
- **Frontend**: Vue 3 → React → Svelte → Solid → Angular → Vanilla
- **Database**: SQLite → PostgreSQL → MySQL
- **AI Providers**: Anthropic, OpenAI, Ollama
- **No ORM**: Type-safe query builder

## Quick Start

```bash
# Install Nexus
npm install -g @nexus/cli

# Initialize new project
nexus init my-api

# Scaffold authentication
nexus scaffold authentication

# Plan a feature
nexus plan "Add shopping cart"

# Execute step-by-step with review
nexus execute --review-each-step
```

## License

MIT License - Free for personal and commercial use.

## Documentation

For detailed information, see:

- [Architecture](./docs/architecture/ARCHITECTURE.md) - System architecture overview
- [Methodologies](./docs/engineering/METHODOLOGIES.md) - All development methodologies
- [Patterns](./docs/engineering/patterns/PATTERNS.md) - All 23+ design patterns
- [Conventions](./docs/engineering/conventions/SOLID_PRINCIPLES.md) - SOLID principles
- [Testing](./docs/engineering/testing/TESTING_CONVENTIONS.md) - Testing conventions
- [Development](./docs/engineering/development/DEVELOPMENT_GUIDELINES.md) - Development guidelines
- [Git Version Control](./docs/engineering/development/GIT_VERSION_CONTROL.md) - Git workflow and commit message standards
- [Security](./docs/engineering/security/SECURITY.md) - Security guidelines
- [Code Quality](./docs/engineering/quality/CODE_QUALITY_STANDARDS.md) - Code quality standards
- [Problem Frames](./docs/engineering/problem_frames/PROBLEM_FRAMES.md) - Problem Frames approach
- [Specification Driven](./docs/engineering/specification_driven/SPECIFICATION_DRIVEN_DEVELOPMENT.md) - Specification driven development
- [Domain Driven Design](./docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md) - Domain-driven design
- [Clean Architecture](./docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md) - Clean architecture
- [Test Driven Development](./docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md) - Test-driven development
- [Behavior Driven Development](./docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - Behavior-driven development
- [Living Documentation](./docs/engineering/living_documentation/LIVING_DOCUMENTATION.md) - Living documentation
- [Event Sourcing](./docs/engineering/event_sourcing/EVENT_SOURCING.md) - Event sourcing (optional)
- [CQRS](./docs/engineering/cqrs/CQRS.md) - CQRS (optional)
- [Pattern Language](./docs/engineering/pattern_language/PATTERN_LANGUAGE.md) - Pattern language

## Support

- GitHub: [Issues](https://github.com/your-org/nexus/issues)
- Documentation: [docs/](./docs/)
- Examples: [examples/](./examples/)
