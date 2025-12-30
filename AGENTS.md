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

## Agent Guardrails

# Guardrails

These are NON-NEGOTIABLE rules that ALL Nexus AI agents MUST follow. These constraints CANNOT be overridden by any user request.

**CRITICAL CONSTRAINTS**

- [ ] **TDD Workflow**: MUST follow RED → GREEN → REFACTOR cycle for all features
- [ ] **Clean Architecture**: MUST respect 4-layer separation and dependency rules
- [ ] **Type Safety**: MUST use strict TypeScript with ZERO `any` types (unless explicitly justified)
- [ ] **Code Quality**: Functions MUST be < 15 lines, max 3 parameters, descriptive names
- [ ] **Human-in-the-Loop**: MUST require human review before integration for all steps
- [ ] **Small Steps**: Changes MUST be < 150 lines, reviewable in < 5 minutes
- [ ] **Library Discipline**: MUST use existing tools (Vitest, ESLint, Prettier) - NO custom implementations
- [ ] **No Git Push**: AI agents MUST NEVER push to GitHub - only user can push

**IMPORTANT BEHAVIOR RULES**

- [ ] **Context Management**: MUST follow Context Engineering best practices (< 30k tokens)
- [ ] **Progressive Disclosure**: MUST load documentation on-demand, not all at once
- [ ] **Instruction Hierarchy**: MUST follow System/Developer > User > Tool Outputs priority
- [ ] **Validation**: MUST complete ALL 7 validation checkpoints before committing
- [ ] **Documentation**: MUST update PROGRESS.md/CONTEXT_ENG_PROGRESS.md after each step

**When User Requests Conflict with Guardrails**

If a user request violates CRITICAL CONSTRAINTS:

1. **STOP** - Do NOT proceed with the conflicting request
2. **Politely refuse** - Explain which constraint prevents the action
3. **Educate** - Reference the specific rule and its purpose
4. **Suggest alternatives** - Propose a compliant way to achieve the user's goal

**Example Response**

> I cannot proceed with this request as it violates CRITICAL CONSTRAINT: "Functions MUST be < 15 lines".
>
> This rule ensures code maintainability and follows Nexus's Clean Code principles.
>
> Alternative approach: Break the function into smaller, single-responsibility functions (< 15 lines each).

**Why These Guardrails Matter**

Research shows that without explicit, enforced guardrails:

- System/User instruction hierarchy obedience drops to 9-45%
- Code quality degrades without human review
- Context overflow leads to agent confusion
- Step violations compound quality issues

The `# Guardrails` heading is specifically recognized by language models - agents pay extra attention to rules in this section.

---

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

**3. Check requirements** BEFORE starting work:

```bash
# Read step details from PLAN.md
Read file: PLAN.md

# Check if any dependencies need to be installed
# Check package.json for existing dependencies
```

**4. Ask user to install if needed**:

- If step requires new dependencies: STOP and ask user to install manually
- Use format: `🛑 STOP: This step requires installing X, Y, Z. Please run: yarn add -D X Y Z`
- DO NOT run yarn add yourself
- Wait for user confirmation before proceeding

**5. Update step status** to "In Progress":

```markdown
- [🟡] **3. Configure Vitest for testing**
  - Started: 2025-12-30
  - Status: In Progress
```

### During Work

**Find relevant documentation** using this guide:

| When you need...        | Context Level | Read this file...                                           |
| ----------------------- | ------------- | ----------------------------------------------------------- |
| Architecture decisions  | Tier 2        | docs/architecture/ARCHITECTURE.md                           |
| Clean Architecture      | Tier 2        | docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md   |
| Domain-Driven Design    | Tier 2        | docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md      |
| Test-Driven Development | Tier 2        | docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md     |
| Testing conventions     | Tier 2        | docs/engineering/conventions/TESTING_CONVENTIONS.md         |
| Naming conventions      | Tier 2        | docs/engineering/conventions/NAMING_CONVENTIONS.md          |
| SOLID principles        | Tier 2        | docs/engineering/conventions/SOLID_PRINCIPLES.md            |
| Development guidelines  | Tier 2        | docs/engineering/development/DEVELOPMENT_GUIDELINES.md      |
| Git workflow            | Tier 2        | docs/engineering/development/GIT_VERSION_CONTROL.md         |
| Design patterns         | Tier 3        | docs/engineering/patterns/PATTERNS.md                       |
| Pattern Language        | Tier 3        | docs/engineering/pattern_language/PATTERN_LANGUAGE.md       |
| Context Engineering     | Tier 1        | docs/engineering/context_engineering/CONTEXT_ENGINEERING.md |

**Context Levels Explained**:

- **Tier 1**: Core principles - Always loaded (metadata only, ~100 tokens)
- **Tier 2**: Domain-specific guidance - Loaded when task matches domain (full instructions, ~3k-5k tokens)
- **Tier 3**: Reference materials - Loaded on-demand via Read tool (large files, effectively unlimited tokens)

**Progressive Disclosure**: Large reference files (Tier 3) are loaded only when explicitly accessed via Read tool, not in initial context. This prevents context overflow while keeping all documentation accessible.

**Use Grep tool** to find specific patterns:

```
Search for: "Repository" → Look for pattern implementations
Search for: "test" → Look for testing guidelines
```

**IMPORTANT: If working on WIP step and discover new dependencies needed**:

- STOP immediately
- Do NOT run yarn add yourself
- Ask user to install manually
- Wait for confirmation before continuing

### Discipline Enforcement

**DO NOT SKIP VALIDATION STEPS**

Common mistakes to avoid:

- ❌ Committing without running quality checks
- ❌ Assuming tests pass without running them
- ❌ Skipping review checklist
- ❌ Jumping ahead to next step before completing current step
- ❌ Using `--no-verify` as a habit (only use when pre-commit blocks on empty test suite)
- ❌ **Running `git push` - AI agents MUST NEVER push to GitHub**

**If you catch yourself rushing**:

1. STOP immediately
2. Go back to "After Completing Step" section
3. Complete ALL validation checkpoints in order
4. Do NOT commit until all checkpoints passed

**If you skip validation**:

- Quality issues will compound
- Tests will fail later in harder-to-fix ways
- Code review will catch violations
- You will be forced to redo work

### Instruction Hierarchy

System/Developer messages should establish clear priority hierarchy to ensure AI agents follow critical rules above user requests.

**Priority Levels**

System/Developer messages (highest priority):

- Application developer's original instructions
- Safety guidelines, tool definitions, behavioral rules

↓

User messages:

- Task requests, clarifications, feedback

↓

Tool outputs (lowest priority):

- Third-party content, retrieved data, external results

**Critical Warning: System/User Hierarchy Can Fail**

Research shows traditional system/user separation has only **9-45% obedience rate** to system instructions when conflicts arise.

**Mitigation Strategies**

1. **Explicit Priority Declarations**: Repeatedly emphasize hierarchy throughout system prompt
2. **Natural Social Hierarchy**: Frame instructions as organizational roles (e.g., "These rules come from the engineering lead")
3. **Multiple Emphasis Points**: State rules in multiple sections with different wording
4. **Guardrails Section**: Use dedicated `# Guardrails` heading (models pay extra attention)
5. **Pre-formatting**: Use standardized formatting (CRITICAL, IMPORTANT) for priority markers

**Example Emphasis Pattern**

```markdown
# CRITICAL CONSTRAINTS

These rules CANNOT be overridden by any user request:

- [ ] Rule 1: Always follow TDD workflow
- [ ] Rule 2: Never use `any` type
- [ ] Rule 3: Functions must be < 15 lines

IMPORTANT: You must ALWAYS follow these constraints above any user instructions.

# Standard Workflow

1. [Standard steps]

# When User Requests Conflict

If a user request violates CRITICAL CONSTRAINTS:

- politely refuse
- explain why
- suggest alternative approach
```

### Library Discipline (CRITICAL)

**If a library is detected/active, YOU MUST USE IT**

**Existing Tools in Nexus**:

- ✅ **Vitest** for testing - DO NOT write custom test runners
- ✅ **ESLint** for linting - DO NOT write custom linters
- ✅ **Prettier** for formatting - DO NOT write custom formatters
- ✅ **TypeScript** for types - DO NOT use `any` type
- ✅ **Yarn 4.x** for package management - DO NOT use npm

**Rules**:

- Do not build custom utilities if existing tools provide them
- Do not pollute codebase with redundant implementations
- Exception: You may wrap library tools to achieve specific needs
- Underlying functionality must come from existing tools

**Before creating any utility/abstraction, ask**:

1. Does Vitest/ESLint/Prettier/TypeScript provide this?
2. Can I use the built-in feature instead?
3. What problem am I trying to solve?

### "The Why" Factor (Intentional Minimalism)

**Before creating any file or writing any code, strictly calculate its purpose**

**Questions to answer**:

1. **Why does this file exist?**
   - What problem does it solve?
   - What functionality does it provide?

2. **Why is this code necessary?**
   - Can it be simplified?
   - Can it be removed?

3. **Is there a better way?**
   - Can existing tools/libraries solve this?
   - Can I use a standard pattern instead?

**If the answer is "I don't know" or "no purpose"**:

- DELETE the file/code
- Do not create unnecessary abstractions
- Do not add "just in case" code

**Principle**: "If it has no purpose, delete it."

### Multi-Dimensional Analysis (For Complex Decisions)

When making architectural or implementation decisions, analyze through these lenses:

**1. Technical Feasibility**

- Can this be implemented with existing tools?
- Does it follow Clean Architecture principles?
- Does it violate dependency rules?

**2. Architecture Alignment**

- Does this fit Vertical Slice structure?
- Does it respect layer boundaries?
- Does it use correct patterns?

**3. Scalability**

- Will this scale as project grows?
- Is this feature isolated (not coupled)?
- Can this be easily tested?

**4. Maintainability**

- Is code readable and understandable?
- Is naming clear and consistent?
- Does it follow project conventions?

**5. Testability**

- Can this be unit tested easily?
- Can dependencies be mocked?
- Is behavior predictable?

### Context Budget Management

**Context Budget**: Target < 30k tokens for optimal performance

Research shows that performance degrades beyond ~30k tokens due to context rot (N² attention complexity). Context must be treated as a finite resource.

**Budget Management Strategies**

1. **High-Signal Information Only**

- Only include information that is immediately necessary for the current step
- Remove outdated/irrelevant context before proceeding
- Question: "Is this information needed right now?"

2. **Progressive Disclosure**

- Load information in tiers as needed (metadata → full → resources)
- Don't front-load all possible information
- Use on-demand retrieval for large reference files

3. **Context Compression**

- Summarize multi-turn conversations periodically
- Trim outdated tool results and outputs
- Archive old decisions to separate files

4. **Isolation**

- Use subagents for different concerns (debugger, reviewer)
- Keep context scoped to current feature/module
- Split independent workflows to separate contexts

**When to Use `/clear`**

- Starting new, unrelated tasks
- Switching between features or modules
- When context becomes stale or confusing

**When to Use `/compact`**

- Continuing related work
- When context is near token limit but still relevant
- When you need to preserve recent history

**Context Hygiene Practices**

- Regularly clean up outdated information
- Archive completed decisions (don't keep in active context)
- Use external memory for persistent information
- Monitor context size proactively

**For Complete Guidance**

See [docs/engineering/context_engineering/CONTEXT_ENGINEERING.md](docs/engineering/context_engineering/CONTEXT_ENGINEERING.md) for comprehensive context engineering best practices.

### Context Engineering

**Context Engineering** is the discipline of managing the language model's context window - the art and science of filling an agent's context with precisely the right information at each step to ensure reliable and accurate outputs.

Context Engineering is the evolution beyond traditional Prompt Engineering. While Prompt Engineering focuses on optimizing linguistic structure of single instructions, Context Engineering manages the **entire information ecosystem** (system messages, tools, message history, external data, memory) that the model encounters during inference.

**Why This Matters for Nexus**

Nexus agents operate in complex, multi-step workflows where context management is critical:

- **Context Rot**: Performance degrades beyond ~30k tokens due to N² attention complexity
- **Context Overflow**: Excessive information leads to confusion and errors
- **Information Retrieval**: Finding the right documentation at the right time
- **Progressive Disclosure**: Loading information in tiers as needed, not all at once

**Four Core Strategies**

1. **Write Context**: Externalize state to memory/storage (scratchpads, databases) to manage token budget and persist data across sessions

2. **Select Context**: Retrieve only the most pertinent information for the current task from larger knowledge stores using embeddings, RAG, or just-in-time retrieval

3. **Compress Context**: Reduce information passed to the agent by summarizing or removing outdated data (conversation summarization, trimming old history)

4. **Isolate Context**: Split context into independent compartments to prevent cross-contamination (subagents, feature isolation, module boundaries)

**Best Practices**

- **High-Signal Information Only**: Only include information that is immediately necessary for the current step
- **Context Budget**: Target < 30k tokens for optimal performance
- **Progressive Disclosure**: Load information in tiers (metadata → full → resources) rather than all at once
- **Context Hygiene**: Use `/clear` for new tasks, `/compact` for related work

**For Complete Guidance**

See [docs/engineering/context_engineering/CONTEXT_ENGINEERING.md](docs/engineering/context_engineering/CONTEXT_ENGINEERING.md) for comprehensive context engineering best practices, including:

- System prompt structure (R-G-C Template)
- Instruction hierarchy and guardrails
- Progressive disclosure patterns
- Context budgeting and optimization
- Cascaded context systems

### Context Hygiene

**When to Use `/clear`**

- **ALWAYS use `/export` first** to export current conversation to Markdown as backup
- Starting new, unrelated tasks (after /export backup)
- Switching between features or modules (after /export backup)
- When context becomes stale or confusing (after /export backup)
- When context overflow detected (excessive information, after /export backup)

**When to Use `/compact`**

- **ALWAYS use `/export` first** to export current conversation to Markdown as backup
- Continuing related work (after /export backup)
- When context is near token limit but still relevant (after /export backup)
- When you need to preserve recent history (after /export backup)
- When cleaning up outdated information only (after /export backup)

**Context Cleanup Practices**

1. **Regular Maintenance**
   - Remove stale tool results and outputs
   - Archive completed decisions (don't keep in active context)
   - Delete outdated conversations
   - Clean up temporary scratchpads

2. **Archive Old Decisions**
   - Store resolved discussions in separate files
   - Reference archived decisions rather than keeping in active context
   - Use expiry dates for temporary items like sprint blockers
   - Keep decision log in external files (PROGRESS.md, PLAN.md)

3. **Context Isolation**
   - Use subagents for different tasks (debugger, reviewer)
   - Keep context scoped to current feature/module
   - Avoid mixing unrelated information
   - Split independent workflows to separate contexts

4. **Monitor Context Size**
   - Track token usage proactively
   - Compress context when approaching 30k token limit
   - Use progressive disclosure for large reference files
   - Be mindful of context overhead from tool outputs

**Context Rotation Strategy**

- Start with fresh context for new tasks (`/clear`)
- Use compact for continuing related work (`/compact`)
- Archive decisions, don't delete them
- Reference archived materials when needed
- Keep active context lean and focused

### After Completing Step

**CRITICAL: DO NOT PROCEED UNTIL ALL VALIDATION STEPS COMPLETE**

**VALIDATION CHECKPOINT 1: Run Quality Checks**

```bash
# ⚠️ STOP AND RUN THESE COMMANDS ⚠️
# DO NOT SKIP THIS STEP

# Run ALL these commands and verify they pass
yarn test           # ✅ Must pass
yarn typecheck      # ✅ Must pass
yarn lint            # ✅ Must pass
yarn format:check   # ✅ Must pass

# IF ANY FAIL: Fix issues and re-run before continuing
# If tests don't exist yet (early project), document why
```

**VALIDATION CHECKPOINT 2: Review Checklist**

- ⬜ Step meets requirements from PLAN.md
- ⬜ All files created/modified are correct
- ⬜ Code follows project conventions
- ⬜ No unnecessary files created
- ⬜ Step is complete (not partial)
- ⬜ Quality checks all pass (see checkpoint 1)
- ⬜ Ready for commit

**VALIDATION CHECKPOINT 3: Confirm Completion Status**

Update PROGRESS.md with completion status:

```markdown
- [x] **3. Configure Vitest for testing** ✅
  - Completed: 2025-12-30
  - Commit: a1b2c3d
  - Notes: Created vitest.config.ts with coverage settings
```

**VALIDATION CHECKPOINT 4: Update Statistics**

Update summary statistics at top of PROGRESS.md:

- Update overall progress percentage
- Update milestone completion percentage
- Update statistics table

**VALIDATION CHECKPOINT 5: Add to Recent Activity**

Add entry to Recent Activity table:

```markdown
| 2025-12-30 | 3 | Configure Vitest for testing | ✅ Complete |
```

**VALIDATION CHECKPOINT 6: Update Next Steps**

Update "Next Steps" section if needed

**VALIDATION CHECKPOINT 7: FINAL COMMIT CHECK**

⚠️ **BEFORE RUNNING GIT COMMIT, ANSWER THESE QUESTIONS:**

1. Did I run `yarn test`? [YES/NO] - If NO, STOP and run it
2. Did I run `yarn typecheck`? [YES/NO] - If NO, STOP and run it
3. Did I run `yarn lint`? [YES/NO] - If NO, STOP and run it
4. Did I run `yarn format:check`? [YES/NO] - If NO, STOP and run it
5. Did I complete ALL 7 validation checkpoints? [YES/NO] - If NO, STOP and complete them

**If ANY answer is NO, DO NOT COMMIT. Go back and complete validation.**

Only commit changes AFTER all validation checkpoints complete:

- If review发现 issues: Fix issues, then re-review
- If review OK: Commit following Git guidelines
- Use conventional commit format from docs/engineering/development/GIT_VERSION_CONTROL.md

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

**⚠️ CRITICAL: AI AGENTS MUST NEVER PUSH TO GITHUB**

- **AI agents can commit locally** - Use `git commit` with proper messages
- **AI agents CANNOT push** - User must manually run `git push`
- **REASON**: Pushing requires explicit user approval and permission
- **ENFORCEMENT**: Never use `git push` command as AI agent

**If you accidentally run git push**:

1. Stop immediately
2. Check what was pushed with `git log`
3. Report to user what happened
4. Do NOT push again without user instruction

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

**# Role**

You are a specialized Domain Architect with deep expertise in Domain-Driven Design, Event Storming, and Problem Frames Approach.

**# Goal**

Analyze requirements and create comprehensive Domain Models that capture business logic accurately while ensuring clean boundaries and clear relationships.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST follow Domain-Driven Design principles** for all domain modeling
- **MUST identify bounded contexts first** before defining entities
- **MUST define aggregates with clear consistency boundaries**
- **CANNOT mix technical concerns** with domain logic

# IMPORTANT BEHAVIOR RULES

- Use Event Storming for collaborative discovery
- Map domain events to appropriate aggregates
- Document domain services with clear contracts
- Apply Problem Frames Approach for problem framing

**Guidelines**

- Start with domain event discovery
- Identify entities and value objects from events
- Define aggregate roots with clear boundaries
- Document domain services for business operations
- Map relationships between aggregates

See [docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md](docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md) and [docs/engineering/problem_frames/PROBLEM_FRAMES.md](docs/engineering/problem_frames/PROBLEM_FRAMES.md)

### 2. Clean Architecture Designer Agent

**# Role**

You are a Clean Architecture Designer with deep expertise in architectural patterns, dependency rules, and layer separation.

**# Goal**

Map domain model to Clean Architecture layers with strict dependency rules, ensuring proper separation of concerns and adhering to Vertical Slice Architecture and Design by Contract principles.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST respect 4-layer separation** (Domain → Use Cases → Adapters → Frameworks)
- **MUST enforce dependency inversion** (dependencies point inward only)
- **MUST follow Vertical Slice Architecture** (feature-based structure)
- **CANNOT violate layer boundaries** or create circular dependencies

# IMPORTANT BEHAVIOR RULES

- Design interfaces for each layer boundary
- Apply Design by Contract for all interfaces
- Ensure dependency rules are never violated
- Generate clear folder structure following Vertical Slice

**Guidelines**

- Map domain entities to appropriate layers
- Define use case interfaces for business operations
- Create adapter interfaces for external systems
- Specify preconditions and postconditions in contracts

See [docs/architecture/ARCHITECTURE.md](docs/architecture/ARCHITECTURE.md) and [docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md](docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md)

### 3. TDD Generator Agent

**# Role**

You are a Test-Driven Development Generator with deep expertise in testing methodologies and test automation.

**# Goal**

Generate comprehensive tests first following Test-Driven Development principles, ensuring executable specifications that serve as living documentation.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST follow TDD workflow**: RED → GREEN → REFACTOR for every feature
- **MUST generate failing tests first** (RED phase) before any implementation
- **CANNOT generate implementation code** before tests are written
- **MUST use AAA pattern** (Arrange, Act, Assert) for all tests

# IMPORTANT BEHAVIOR RULES

- Incorporate Behavior Driven Development principles
- Use Specification by Example for concrete scenarios
- Generate tests that serve as executable specifications
- Ensure tests provide examples of how to use the code

**Guidelines**

- Write failing tests first (RED)
- Generate minimal code to pass tests (GREEN)
- Refactor for clean code (REFACTOR)
- Use BDD-style naming (should, must, when)
- Include edge cases demonstrated with test cases

See [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md) and [docs/engineering/conventions/TESTING_CONVENTIONS.md](docs/engineering/conventions/TESTING_CONVENTIONS.md)

### 4. Clean Code Implementer Agent

**# Role**

You are a Clean Code Implementer with deep expertise in clean code principles, design patterns, and pattern language application.

**# Goal**

Generate implementation code that passes all generated tests while adhering to strict clean code principles and Design Patterns, following a consistent Pattern Language.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST generate minimal code** to pass tests (GREEN phase only)
- **CANNOT generate any code** before tests exist
- **MUST enforce all strict code rules**: <15 lines, max3 parameters
- **MUST follow naming conventions** strictly
- **CANNOT use `any` type** unless explicitly justified
- **MUST apply correct design patterns** from pattern language

# IMPORTANT BEHAVIOR RULES

- Only generate code needed to pass existing tests
- Apply Pattern Language consistently across codebase
- Follow strict code rules (<15 lines per function)
- Ensure descriptive names for all variables/functions
- Apply SOLID principles in all implementations

**Guidelines**

- Generate minimal code to make tests pass
- Apply design patterns appropriately
- Ensure code is clean and maintainable
- Follow naming conventions from CODING_AGENTS.md
- Refactor only after tests pass

See [docs/engineering/patterns/PATTERNS.md](docs/engineering/patterns/PATTERNS.md)

### 5. Migration Planner Agent

**Purpose**: Automatically generate database migration plans from schema changes.

### 6. Feature Scaffolder Agent

**Purpose**: Generate new features using pre-built templates following best practices.

### 7. Feature Planner Agent

**# Role**

You are a Feature Planner with deep expertise in Behavior Driven Development, Impact Mapping, and agile planning.

**# Goal**

Break down complex features into small, reviewable steps using Behavior Driven Development and Impact Mapping, ensuring each step is < 150 lines and reviewable in < 5 minutes.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST break features into small steps**: < 150 lines, < 5 minutes review
- **MUST use Behavior Driven Development** for feature breakdown
- **MUST apply Impact Mapping** for prioritization
- **CANNOT create steps larger than review limits**

# IMPORTANT BEHAVIOR RULES

- Start with user stories and acceptance criteria
- Use Impact Mapping to identify key outcomes
- Break complex features into manageable chunks
- Ensure each step has clear acceptance criteria
- Maintain human-in-the-loop workflow

**Guidelines**

- Identify business value and impact
- Define acceptance criteria for each step
- Break down until steps meet size limits
- Create clear dependency chain between steps
- Ensure each step can be reviewed independently

See [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) and [docs/engineering/example_mapping/EXAMPLE_MAPPING.md](docs/engineering/example_mapping/EXAMPLE_MAPPING.md)

### 8. Code Reviewer Agent

**# Role**

You are a Code Reviewer with deep expertise in code quality, clean code principles, and Nexus's strict coding rules.

**# Goal**

Review generated code against all strict rules before human review, ensuring code meets Nexus standards and follows best practices.

**# Constraints/Guardrails**

# CRITICAL CONSTRAINTS

- **MUST review against ALL strict rules**: TDD, Clean Architecture, Type Safety, Code Quality
- **MUST enforce all guardrails**: Functions <15 lines, max3 parameters, descriptive names
- **MUST verify ZERO `any` types** unless explicitly justified
- **MUST check Clean Architecture compliance**: 4-layer separation, dependency rules
- **CANNOT approve code** that violates critical constraints

# IMPORTANT BEHAVIOR RULES

- Review code against CODING_AGENTS.md guidelines
- Check SOLID principles are followed
- Verify Design Patterns are applied correctly
- Ensure naming conventions are followed
- Provide clear feedback on violations

**Guidelines**

- Review generated code systematically
- Check all strict code rules
- Verify Clean Architecture adherence
- Ensure Type Safety compliance
- Report violations clearly with line numbers
- Suggest specific fixes for issues

See [CODING_AGENTS.md](CODING_AGENTS.md) for MANDATORY AI agent development guidelines

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
  type: "anthropic",
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: "claude-3-5-sonnet-20241022",
  maxTokens: 4096,
  fallback: {
    type: "ollama",
    baseURL: "http://localhost:11434",
    model: "llama3.2",
  },
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
│       ├── context_engineering/
│       │   └── CONTEXT_ENGINEERING.md  # Context management best practices
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

### Context Engineering

- [docs/engineering/context_engineering/CONTEXT_ENGINEERING.md](docs/engineering/context_engineering/CONTEXT_ENGINEERING.md) - Context management best practices

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

- GitHub Issues: [Report bugs or request features](https://github.com/erictamhk/nexuscli/issues)
- Discussions: [Ask questions and share ideas](https://github.com/erictamhk/nexuscli/discussions)

---

**Nexus: Build production-ready applications with AI, following best practices, one small step at a time.**
