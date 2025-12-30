# Context-Aware Action Protocol (CAAP)

**CRITICAL: ALL AI agents MUST follow this protocol before taking ANY action.**

## Overview

The Context-Aware Action Protocol (CAAP) ensures you load the right documentation, follow all rules, and never miss critical instructions. It's the systematic application of Context Engineering principles to every agent action.

**Purpose**: Prevent missed rules, ensure complete context, and maintain context budget discipline.

**When to Apply**: Before ANY git command, file edit, code generation, or action.

---

## Protocol At a Glance

```
┌─────────────────────────────────────────────────────────┐
│ Phase 1: Task Classification & Discovery                │
│ 1. Classify task type                                    │
│ 2. Answer Three-Question Rule                           │
│ 3. Run Discovery Scan (grep)                            │
│ 4. Load ONLY Tier 2 Documents                           │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 2: Action Execution                               │
│ 5. Create Constraints Checklist                         │
│ 6. Validate Before Acting                               │
│ 7. Execute Action                                       │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 3: Commit Validation (If Applicable)              │
│ 8. Load GIT_VERSION_CONTROL.md                         │
│ 9. Create Detailed Commit Message                      │
│ 10. Run Quality Checks                                  │
│ 11. Only Then Commit                                    │
└─────────────────────────────────────────────────────────┘
```

---

## Phase 1: Task Classification & Discovery

### Step 1: Classify Your Task

Before ANY git command, file edit, or action, classify your task type using this matrix:

| Task Type        | When You're...        | Tier 2 Docs to Load                                                                                               |
| ---------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **COMMITTING**   | Running `git commit`  | `docs/engineering/development/GIT_VERSION_CONTROL.md`                                                             |
| **CODING**       | Writing/editing code  | `CODING_AGENTS.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md`                                        |
| **TESTING**      | Writing tests         | `docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md` |
| **ARCHITECTURE** | Designing structure   | `docs/architecture/ARCHITECTURE.md` + `docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md`                 |
| **DOMAIN**       | Domain modeling       | `docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md` + `docs/engineering/problem_frames/PROBLEM_FRAMES.md`    |
| **REVIEW**       | Reviewing code        | `CODING_AGENTS.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md`                                        |
| **DOCS**         | Editing documentation | `docs/engineering/development/GIT_VERSION_CONTROL.md` (if committing)                                             |
| **GENERAL**      | Any agent task        | `docs/engineering/context_engineering/CONTEXT_ENGINEERING.md` + `AGENTS.md` (full)                                |

**Examples**:

```
User: "Fix the login bug"
→ Task Type: CODING
→ Load: CODING_AGENTS.md + TESTING_CONVENTIONS.md

User: "Commit these changes"
→ Task Type: COMMITTING
→ Load: GIT_VERSION_CONTROL.md

User: "Create domain model for shopping cart"
→ Task Type: DOMAIN
→ Load: DOMAIN_DRIVEN_DESIGN.md + PROBLEM_FRAMES.md

User: "Review this PR"
→ Task Type: REVIEW
→ Load: CODING_AGENTS.md + TESTING_CONVENTIONS.md
```

### Step 2: Three-Question Rule

Before acting, answer these 3 questions:

**Question 1: What file/section will I modify?**

→ Use grep to find section in AGENTS.md and look for "See [link]" references

```bash
# Example for committing:
grep -A 30 "Git Commit Requirements" AGENTS.md

# Example for coding:
grep -A 20 "Strict Code Rules" CODING_AGENTS.md
```

**Question 2: What type of change is this?**

→ Use Task Type Matrix above to identify which docs to load

```bash
# If writing code → CODING task
# If running git commit → COMMITTING task
# If designing structure → ARCHITECTURE task
```

**Question 3: What are validation requirements?**

→ Read PROGRESS.md "After Completing Step" section (finished plans archived in docs/archive/)

```bash
# Find validation requirements
grep -A 20 "After Completing Step" PROGRESS.md
```

### Step 3: Discovery Scan (Find References)

Before reading documentation, scan AGENTS.md with grep to find what you need:

```bash
# Find relevant section (example for committing)
grep -A 30 "Git Commit Requirements" AGENTS.md

# Find what documents exist
grep "^## " AGENTS.md

# Find all "See [link]" references in relevant section
grep -B 10 -A 2 "GIT_VERSION_CONTROL" AGENTS.md

# Find code requirements
grep -A 50 "Strict Code Rules" CODING_AGENTS.md

# Find validation checkpoints
grep -A 30 "VALIDATION CHECKPOINT" AGENTS.md
```

**Purpose**: This discovers what documentation EXISTS without loading everything.

### Step 4: Load ONLY Tier 2 Documents (Not Tier 3)

**Tier 2 (Load Now - These contain the rules you MUST follow)**:

- `docs/engineering/development/GIT_VERSION_CONTROL.md` (for committing)
- `CODING_AGENTS.md` (for coding)
- `docs/engineering/conventions/TESTING_CONVENTIONS.md` (for testing)
- `docs/architecture/ARCHITECTURE.md` (for structure)
- `docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md` (for architecture)
- `docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md` (for domain modeling)
- `docs/engineering/context_engineering/CONTEXT_ENGINEERING.md` (always)

**Tier 3 (Reference On-Demand - Read with Read tool only when needed)**:

- `docs/engineering/patterns/PATTERNS.md` (only when implementing specific pattern)
- `docs/engineering/conventions/SOLID_PRINCIPLES.md` (only when refactoring)
- `docs/engineering/conventions/NAMING_CONVENTIONS.md` (only when naming is uncertain)
- Other reference materials

**Key Principle**: Load Tier 2 BEFORE action, use Read tool for Tier 3 DURING action only when needed.

**Loading Example**:

```bash
# WRONG: Loading too much
Read docs/engineering/patterns/PATTERNS.md
Read docs/engineering/conventions/SOLID_PRINCIPLES.md
Read CODING_AGENTS.md
Read TESTING_CONVENTIONS.md

# RIGHT: Only load what you need
Read CODING_AGENTS.md              # Tier 2 - mandatory for coding
Read TESTING_CONVENTIONS.md        # Tier 2 - mandatory for testing
# Use Read tool for PATTERNS.md later only if actually implementing a pattern
```

---

## Phase 2: Action Execution

### Step 5: Create Constraints Checklist

From the Tier 2 documents you loaded, extract all constraints into a checklist:

**Example for Committing**:

```markdown
## Constraints Checklist (from GIT_VERSION_CONTROL.md)

Commit Message Format:

- [ ] Subject: `<type>[scope]: <description>`
- [ ] Body: Detailed bullets explaining WHY, not just WHAT
- [ ] Footer: Optional references (Closes #123, Co-authored-by)

Pre-commit Quality Checks:

- [ ] All tests passing: `yarn test`
- [ ] Typecheck passing: `yarn typecheck`
- [ ] Linting passing: `yarn lint`
- [ ] Code formatted: `yarn format`

Commit Types:

- [ ] feat: New feature
- [ ] fix: Bug fix
- [ ] docs: Documentation changes
- [ ] style: Code style changes (formatting)
- [ ] refactor: Code refactoring
- [ ] test: Adding or updating tests
- [ ] chore: Maintenance tasks
```

**Example for Coding**:

```markdown
## Constraints Checklist (from CODING_AGENTS.md)

Code Rules:

- [ ] Functions < 15 lines
- [ ] Max 3 parameters per function
- [ ] Descriptive names (no abbreviations)
- [ ] No `any` types (unless explicitly justified)

Architecture Rules:

- [ ] Clean Architecture 4-layer separation
- [ ] Dependencies point inward only
- [ ] No circular dependencies
- [ ] Domain layer has no framework dependencies

TDD Rules:

- [ ] Tests written first (RED phase)
- [ ] Minimal code to pass tests (GREEN phase)
- [ ] Refactor only after tests pass (REFACTOR phase)
```

### Step 6: Validate Before Acting

Check your constraints checklist:

```markdown
## Pre-Action Validation

Document Loading:

- [ ] All constraints identified from loaded docs
- [ ] No contradictions between docs
- [ ] Context budget < 30k tokens
- [ ] Progressive disclosure followed

Constraint Completeness:

- [ ] All critical constraints identified
- [ ] All behavior rules understood
- [ ] All prohibited actions listed
- [ ] All required tools available

Readiness Check:

- [ ] I understand what I need to do
- [ ] I know the validation requirements
- [ ] I have the documentation I need
- [ ] I'm ready to execute action
```

**If any check fails**: STOP, re-read documentation, ask for clarification.

### Step 7: Execute Action

Only now, execute the action:

- Edit files with full context
- Run quality checks (if applicable)
- Validate against all constraints

**During Action**:

```markdown
## While Executing

Context Management:

- [ ] Use only loaded Tier 2 documentation
- [ ] Read Tier 3 docs only when explicitly needed
- [ ] Track context usage (stay < 30k tokens)

Constraint Adherence:

- [ ] Follow all critical constraints
- [ ] No prohibited actions taken
- [ ] All required steps completed

Quality Validation:

- [ ] Tests pass (if applicable)
- [ ] Linting passes (if applicable)
- [ ] Type checking passes (if applicable)
```

---

## Phase 3: Commit Validation (If Applicable)

### Step 8: Load GIT_VERSION_CONTROL.md

Before ANY commit, read the full file:

```bash
Read docs/engineering/development/GIT_VERSION_CONTROL.md
```

**CRITICAL**: This is mandatory for all commits, no exceptions.

### Step 9: Create Detailed Commit Message

Follow format with BODY (mandatory):

```markdown
<type>[scope]: <description>

<detailed body with bullets>

- Explain what changed
- Why it matters
- Lines added/modified
- Review time
- Step completion context

<footer>
```

**Example**:

```markdown
feat(auth): implement JWT token validation

- Added JWT verification middleware for protected routes
- Implemented token refresh flow with automatic token rotation
- Added error handling for expired and invalid tokens
- Lines added: 127 lines in src/middleware/auth.ts
- Review time: ~3 minutes
- Completes step 4 from PLAN.md: Implement JWT authentication

Closes #42
```

**Commit Types**:

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring (no functional change)
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (dependencies, config)

**Scope Examples**:

- `feat(auth)`: Authentication feature
- `fix(api): API bug fix
- `docs(readme)`: README documentation
- `test(user)`: User-related tests

### Step 10: Run Quality Checks

From GIT_VERSION_CONTROL.md commit checklist:

```bash
# Run ALL these commands and verify they pass
yarn test           # ✅ Must pass
yarn typecheck      # ✅ Must pass
yarn lint           # ✅ Must pass
yarn format:check   # ✅ Must pass

# IF ANY FAIL: Fix issues and re-run before continuing
# If tests don't exist yet (early project), document why in commit body
```

**Quality Check Validation**:

```markdown
## Quality Check Results

Tests:

- [ ] All tests passing
- [ ] Test coverage adequate
- [ ] No failing tests

Type Check:

- [ ] No TypeScript errors
- [ ] All types inferred correctly
- [ ] No `any` types (unless justified)

Linting:

- [ ] No ESLint errors
- [ ] No warnings
- [ ] Code follows conventions

Formatting:

- [ ] Code properly formatted
- [ ] No Prettier errors
- [ ] Consistent style
```

**If any check fails**:

1. Identify what failed
2. Fix the issue
3. Re-run the failed check
4. Re-run all checks to ensure no regressions
5. Only commit when ALL checks pass

**Early Project Exception**:

If tests don't exist yet (early in project):

```bash
# Document this in commit body
test(auth): initial test setup

- Created test infrastructure
- Added test configuration
- No tests yet (early project stage)
- Tests will be added in next step

Note: Test suite is empty - will implement tests in next step
```

### Step 11: Only Then Commit

Commit only AFTER all quality checks pass and commit message follows full format.

**Final Pre-Commit Checklist**:

```markdown
## Final Pre-Commit Validation

Documentation:

- [ ] Read GIT_VERSION_CONTROL.md fully
- [ ] Commit message includes mandatory body
- [ ] Commit type and scope are appropriate

Quality Checks:

- [ ] yarn test: ✅ PASSING
- [ ] yarn typecheck: ✅ PASSING
- [ ] yarn lint: ✅ PASSING
- [ ] yarn format:check: ✅ PASSING

Context:

- [ ] All changes are intentional
- [ ] No accidental commits
- [ ] No secrets or sensitive data
- [ ] No large binary files

Ready:

- [ ] All validation checkpoints complete
- [ ] Ready to commit
```

---

## Never Violate These Rules

### ❌ NEVER do these:

- ❌ **NEVER act without classifying task type**
  - Always start with Step 1: Classify Task Type
  - Use Task Type Matrix to identify documentation

- ❌ **NEVER skip discovery scan (grep for references)**
  - Always scan AGENTS.md before reading docs
  - Find what exists before loading

- ❌ **NEVER assume "I know this"**
  - Documentation is the source of truth
  - Rules may change over time
  - Always verify against documentation

- ❌ **NEVER load ALL docs (causes context overflow)**
  - Load only Tier 2 documents for task type
  - Use Read tool for Tier 3 on-demand
  - Monitor context budget (< 30k tokens)

- ❌ **NEVER commit without validation**
  - Always run quality checks before committing
  - Always include detailed commit message body
  - Always read GIT_VERSION_CONTROL.md first

- ❌ **NEVER skip quality checks**
  - yarn test: Must pass
  - yarn typecheck: Must pass
  - yarn lint: Must pass
  - yarn format:check: Must pass

- ❌ **NEVER commit without detailed message body**
  - Subject line is not enough
  - Body must explain WHY not just WHAT
  - Include step context and line counts

---

## Always Follow These Rules

### ✅ ALWAYS do these:

- ✅ **ALWAYS classify task type first**
  - Use Task Type Matrix
  - Identify Tier 2 documents to load
  - Document your classification

- ✅ **ALWAYS use grep to discover references**
  - Find relevant sections
  - Identify linked documentation
  - Don't assume document structure

- ✅ **ALWAYS read Tier 2 docs for task type**
  - Load only necessary documentation
  - Extract all constraints
  - Create constraints checklist

- ✅ **ALWAYS extract constraints from loaded docs**
  - Create detailed checklist
  - Verify no contradictions
  - Understand all requirements

- ✅ **ALWAYS create validation checklist**
  - Extract all checkpoints
  - Document validation steps
  - Ensure completeness

- ✅ **ALWAYS validate before acting**
  - Check all preconditions
  - Verify context budget
  - Confirm readiness

- ✅ **ALWAYS run quality checks before committing**
  - Run all quality commands
  - Fix any failures
  - Re-run to verify fixes

- ✅ **ALWAYS include detailed commit message body**
  - Explain WHY, not just WHAT
  - Include line counts and review time
  - Reference step completion

---

## Context Budget Management

### Target: < 30k tokens per session

### How to Stay Within Budget:

**1. Task Classification** → 0 tokens (mental step)

- No tokens required

**2. Discovery Scan (grep)** → ~1k tokens

- Use grep to find sections
- Don't read full files yet

**3. Load Tier 2 Docs** → ~5-10k tokens (task-specific only)

- Load only what's needed for task type
- Don't load all documentation

**4. Read Targeted Sections** → ~2-3k tokens (not full files)

- Use Read tool with specific sections
- Don't load entire Tier 3 files

**5. Execute Action** → Context from above steps

- Work with loaded context
- Load Tier 3 on-demand only

**Total**: ~10-15k tokens (well under 30k limit)

### Context Budget Monitoring:

```markdown
## Context Budget Tracking

Current Phase: Phase 2 - Action Execution

Token Usage:

- Task Classification: 0 tokens (mental)
- Discovery Scan (grep): 1,200 tokens
- Tier 2 Docs Loaded: 6,500 tokens
- Tier 3 References: 0 tokens (on-demand)
- Current Total: 7,700 tokens

Budget: < 30,000 tokens ✅
Remaining: 22,300 tokens

Status: ✅ Within budget
```

### Context Budget Best Practices:

1. **Use grep instead of reading full files**
   - Scan for relevant sections
   - Only read what you need

2. **Load documentation progressively**
   - Tier 1: Always loaded (~100 tokens)
   - Tier 2: Task-specific (~3k-5k tokens)
   - Tier 3: On-demand (unlimited, but accessed via Read tool)

3. **Remove stale context**
   - Clean up after each phase
   - Don't keep unnecessary information
   - Archive decisions to files

4. **Monitor context size proactively**
   - Track tokens at each step
   - Stop when approaching limit
   - Use `/compact` if needed

---

## Troubleshooting Guide

### Problem: I'm not sure what task type this is

**Solution**: Use the decision tree:

```
Is it a git commit?
├─ Yes → COMMITTING task
└─ No → Is it writing/editing code?
    ├─ Yes → CODING task
    └─ No → Is it writing tests?
        ├─ Yes → TESTING task
        └─ No → Is it designing structure?
            ├─ Yes → ARCHITECTURE task
            └─ No → GENERAL task
```

### Problem: I can't find the relevant documentation

**Solution**: Use discovery scan:

```bash
# Find all sections in AGENTS.md
grep "^## " AGENTS.md

# Find specific section
grep -A 30 "Git Commit" AGENTS.md

# Find references
grep "See \[" AGENTS.md
```

### Problem: Documents contradict each other

**Solution**:

1. Identify which document is higher priority
2. Check AGENTS.md for hierarchy
3. If still unclear, stop and ask user
4. Document the conflict resolution

### Problem: Context is approaching 30k tokens

**Solution**:

1. Stop and assess what's essential
2. Remove outdated tool results
3. Archive decisions to external files
4. Use `/compact` if continuing related work
5. Use `/clear` if starting new task

### Problem: Quality checks are failing

**Solution**:

1. Identify which check is failing
2. Read the error message carefully
3. Fix the specific issue
4. Re-run the failing check only
5. Re-run all checks to ensure no regressions
6. Only proceed when ALL pass

### Problem: I'm not sure if I've completed all validation checkpoints

**Solution**: Use the checklist:

```markdown
## CAAP Completion Checklist

Phase 1: Task Classification & Discovery

- [ ] Task type classified
- [ ] Three-Question Rule answered
- [ ] Discovery scan completed
- [ ] Tier 2 docs loaded

Phase 2: Action Execution

- [ ] Constraints checklist created
- [ ] Validation before acting completed
- [ ] Action executed

Phase 3: Commit Validation (if applicable)

- [ ] GIT_VERSION_CONTROL.md loaded
- [ ] Detailed commit message created
- [ ] Quality checks run and passing
- [ ] Commit executed

Overall:

- [ ] Context budget < 30k tokens
- [ ] All constraints followed
- [ ] No rules violated
- [ ] Documentation updated (if applicable)
```

---

## Quick Reference Card

### CAAP at a Glance

```
┌─────────────────────────────────────────┐
│ 1. CLASSIFY TASK TYPE                   │
│    → Use Task Type Matrix              │
├─────────────────────────────────────────┤
│ 2. DISCOVER WITH GREP                  │
│    → Scan AGENTS.md for references     │
├─────────────────────────────────────────┤
│ 3. LOAD TIER 2 DOCS ONLY              │
│    → Task-specific documentation       │
├─────────────────────────────────────────┤
│ 4. CREATE CONSTRAINTS CHECKLIST       │
│    → Extract all rules                │
├─────────────────────────────────────────┤
│ 5. VALIDATE BEFORE ACTING             │
│    → Check preconditions              │
├─────────────────────────────────────────┤
│ 6. EXECUTE ACTION                     │
│    → Follow all constraints           │
├─────────────────────────────────────────┤
│ 7. COMMIT VALIDATION (if applicable)   │
│    → Load GIT_VERSION_CONTROL        │
│    → Create detailed message          │
│    → Run quality checks               │
│    → Only then commit                 │
└─────────────────────────────────────────┘
```

### Tiered Documentation Access

| Tier | When to Load          | Loading Strategy           | Token Cost    |
| ---- | --------------------- | -------------------------- | ------------- |
| 1    | Always (startup scan) | Only metadata (name, desc) | ~100 tokens   |
| 2    | Task matches skill    | Full instructions          | ~3k-5k tokens |
| 3    | Explicitly referenced | On-demand via Read tool    | Unlimited     |

### Task Type Matrix

| Task Type    | Documentation                                           | Key Files                              |
| ------------ | ------------------------------------------------------- | -------------------------------------- |
| COMMITTING   | `GIT_VERSION_CONTROL.md`                                | `docs/engineering/development/`        |
| CODING       | `CODING_AGENTS.md` + `TESTING_CONVENTIONS.md`           | Root + `docs/engineering/conventions/` |
| TESTING      | `TEST_DRIVEN_DEVELOPMENT.md` + `TESTING_CONVENTIONS.md` | `docs/engineering/test_driven/`        |
| ARCHITECTURE | `ARCHITECTURE.md` + `CLEAN_ARCHITECTURE.md`             | `docs/architecture/`                   |
| DOMAIN       | `DOMAIN_DRIVEN_DESIGN.md` + `PROBLEM_FRAMES.md`         | `docs/engineering/domain_driven/`      |

---

## Success Metrics

### CAAP Compliance

- ✅ Task classification: 100% of actions
- ✅ Documentation loading: 100% of actions
- ✅ Constraint validation: 100% of actions
- ✅ Quality checks: 100% of commits
- ✅ Context budget: < 30k tokens per session

### Quality Metrics

- ✅ Commit message completeness: 100% include body
- ✅ Quality check pass rate: 100% before commit
- ✅ Constraint adherence: 100% of critical constraints
- ✅ Documentation consistency: 0 contradictions

### Context Efficiency

- ✅ Average session context: < 15k tokens
- ✅ Document loading efficiency: Only Tier 2 for 95% of actions
- ✅ Tier 3 on-demand usage: Only when explicitly needed
- ✅ Context overflow: 0 sessions exceed 30k tokens

---

## Key Takeaways

### CAAP Prevents:

1. **Missed Rules** - By systematically loading relevant documentation
2. **Context Overflow** - By progressive disclosure and budget monitoring
3. **Incomplete Validation** - By enforcing checkpoint completion
4. **Quality Failures** - By requiring quality checks before commit
5. **Poor Commits** - By enforcing detailed commit messages

### CAAP Ensures:

1. **Complete Context** - All relevant documentation loaded
2. **Rule Compliance** - All constraints followed
3. **Quality Assurance** - All checks passing
4. **Clear Communication** - Detailed commit messages
5. **Systematic Process** - Repeatable, reliable workflow

### CAAP Benefits:

1. **Reliability** - Fewer errors and rule violations
2. **Efficiency** - Optimal context usage
3. **Quality** - Higher code quality standards
4. **Traceability** - Clear documentation of decisions
5. **Maintainability** - Consistent, predictable processes

---

**Context-Aware Action Protocol**: Systematic approach to ensure complete context, rule compliance, and quality outcomes for every agent action.

**Remember**: This protocol prevents missing rules, ensures complete context, and maintains context budget discipline. ALWAYS follow CAAP before taking any action.
