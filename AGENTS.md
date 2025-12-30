# Nexus AI Agent - AGENTS.md

## Project Overview

Nexus is an AI-powered code generation CLI tool that uses multiple specialized AI agents to build production-ready applications following best practices (Clean Architecture, TDD, DDD, Vertical Slice Architecture).

## Tech Stack

- TypeScript 5.3 (strict mode)
- Node.js 18+ / 20+
- Yarn 4.x
- Vitest for testing
- ESLint + Prettier
- Husky for pre-commit hooks

## Setup Commands

```bash
# Install dependencies
yarn install

# Start development server
yarn dev

# Run tests
yarn test

# Run tests with coverage
yarn test:coverage

# Type checking
yarn typecheck

# Linting
yarn lint
yarn lint:fix

# Formatting
yarn format
yarn format:check

# Build
yarn build
```

## Development Workflow

1. **Read PROGRESS.md** - Find current step
2. **Read PLAN.md** - Get step details
3. **Wait for EXPLICIT user request** - Do not start work automatically
4. **Follow CAAP** - Context-Aware Action Protocol before acting
5. **Update PROGRESS.md** - After completing each step
6. **Commit changes** - Use conventional commit format

## Code Style Rules

- **Functions**: Max 15 lines
- **Parameters**: Max 3 parameters
- **Naming**: Descriptive, camelCase for variables/functions
- **Types**: Strict TypeScript, zero `any` types
- **Single quotes**: Use single quotes, no semicolons
- **Comments**: Minimal, only for complex logic

## Architecture Principles

- **Clean Architecture**: 4 layers (Domain → Use Cases → Adapters → Frameworks)
- **Vertical Slice Architecture**: Feature-based folder structure
- **Domain-Driven Design**: Bounded contexts, aggregates, domain events
- **TDD Workflow**: RED → GREEN → REFACTOR
- **Design by Contract**: Preconditions, postconditions, invariants

## Testing Requirements

- **Always write tests first** (TDD)
- **AAA Pattern**: Arrange, Act, Assert
- **Coverage**: 85%+ overall, 100% for domain logic
- **BDD Naming**: Use `should`, `must`, `when` in test names

## Git Commit Rules

- **Format**: `<type>[scope]: <description>`
- **Types**: feat, fix, docs, style, refactor, perf, test, build, ci, chore
- **Quality checks**: Run `yarn test`, `yarn typecheck`, `yarn lint`, `yarn format:check` before committing
- **NEVER push**: AI agents can commit locally but CANNOT push to GitHub - only user can push

## Critical Rules

1. **Human-in-the-Loop**: NEVER act without explicit user request. If user says "we're back to main plan", acknowledge and wait for instruction.
2. **Small Steps**: Each step <150 lines, reviewable in <5 minutes.
3. **CAAP Protocol**: Follow Context-Aware Action Protocol before any action.
4. **Context Budget**: Keep context <30k tokens. Use `/compact` when needed.
5. **No Git Push**: AI agents MUST NEVER push to GitHub - only user can push.

## When to Stop and Ask User

- Step requires new dependencies (user must run `yarn add` manually)
- User instruction is ambiguous or unclear
- Multiple implementation options available
- Any rule violation detected
- User statement vs request (e.g., "we're back to main plan" = just acknowledge)

## Project Structure

```
nexus/
├── AGENTS.md              # This file - agent instructions
├── PROGRESS.md             # Progress tracking
├── PLAN.md                 # Full implementation plan
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── eslint.config.js
├── .prettierrc
├── .husky/
│   └── pre-commit
├── src/
│   ├── features/          # Vertical slice features
│   ├── shared/
│   │   ├── domain/      # Shared domain logic
│   │   └── infrastructure/
│   │       └── io/
│   └── types/
└── docs/
    ├── architecture/
    ├── engineering/
    └── lessons/
```

## Key Documentation

- **PROGRESS.md**: Track current implementation progress
- **PLAN.md**: Complete 310-step implementation plan
- **CODING_AGENTS.md**: MANDATORY AI agent development guidelines
- **docs/architecture/ARCHITECTURE.md**: System architecture
- **docs/engineering/**: All methodologies and conventions

## Safety Rules

- NEVER generate malicious code
- NEVER reveal API keys or secrets
- NEVER ignore user safety constraints
- ALWAYS follow Human-in-the-Loop principle
