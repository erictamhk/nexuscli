# Context Engineering Enhancement Plan

## Overview

**Purpose**: Improve AGENTS.md with Context Engineering best practices based on comprehensive research from industry leaders (Anthropic, Claude, OpenAI, etc.).

**Estimated Effort**: ~4-6 hours

**Side Plan**: This is a separate enhancement plan outside the main PLAN.md workflow (Steps 1-310).

---

## Research Summary

Based on research from:

- Anthropic Context Engineering Guide (2025)
- Claude Code Best Practices (2025)
- Context Engineering: The Definitive Guide (FlowHunt, 2025)
- The Instruction Hierarchy (Research Paper, 2023)
- Multiple industry best practice sources

**Key Findings**:

1. **Context Engineering > Prompt Engineering**: Context Engineering manages the entire information ecosystem
2. **Context Rot**: Performance degrades beyond ~30k tokens due to N² attention complexity
3. **Four Core Strategies**: Write, Select, Compress, Isolate context
4. **R-G-C Template**: Role + Goal + Constraints structure for system prompts
5. **Progressive Disclosure**: Tiered loading (metadata → full → resources)
6. **Instruction Hierarchy Failure**: System/user separation only 9-45% obedient
7. **Guardrails Section**: Models pay extra attention to `# Guardrails` heading

---

## Phase 1: High Priority Enhancements (Steps CE1-CE12)

**Duration**: ~3 hours

**Goal**: Add critical context engineering sections to AGENTS.md for immediate agent reliability improvements.

### Step CE1: Add Context Engineering Section

**Action**: Add dedicated Context Engineering section after "Library Discipline" (line ~170)

**Changes**:

- Introduce Context Engineering concept
- Explain why it's critical for Nexus agents
- Overview of 4 core strategies (Write, Select, Compress, Isolate)
- Link to CONTEXT_ENGINEERING.md for details

**Files Changed**: 1
**Lines Added**: ~30 lines
**Review Time**: < 2 minutes

---

### Step CE2: Add Instruction Hierarchy Section

**Action**: Add dedicated section explaining instruction hierarchy after "Discipline Enforcement" (line ~120)

**Changes**:

- Explain instruction hierarchy (System/Developer > User > Tool Outputs)
- Warning about system/user hierarchy failure (9-45% obedience rate)
- Strategies to enforce hierarchy (repetition, emphasis, guardrails)
- Example emphasis pattern for critical constraints

**Files Changed**: 1
**Lines Added**: ~40 lines
**Review Time**: < 3 minutes

---

### Step CE3: Add Guardrails Section

**Action**: Create dedicated `# Agent Guardrails` section near top of file (after Core Philosophy, line ~45)

**Changes**:

- Centralize all MUST/MUST NOT rules
- Use `# Guardrails` heading (models pay extra attention)
- Add emphasis markers (CRITICAL, IMPORTANT)
- Reference research on instruction hierarchy

**Files Changed**: 1
**Lines Added**: ~35 lines
**Review Time**: < 2 minutes

---

### Step CE4: Restructure Domain Architect Agent (R-G-C Template)

**Action**: Rewrite Domain Architect Agent description with R-G-C structure (lines 368-372)

**Changes**:

- Add explicit `# Role` subsection
- Add explicit `# Goal` subsection
- Add explicit `# Constraints/Guardrails` subsection
- Use markdown headings for structure
- Add emphasis markers for critical rules

**Files Changed**: 1
**Lines Added**: ~40 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE5: Restructure Clean Architecture Designer Agent (R-G-C Template)

**Action**: Rewrite Clean Architecture Designer Agent with R-G-C structure (lines 374-378)

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints/Guardrails` sections
- Use markdown headings
- Add emphasis markers
- Reference relevant documentation

**Files Changed**: 1
**Lines Added**: ~40 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE6: Restructure TDD Generator Agent (R-G-C Template)

**Action**: Rewrite TDD Generator Agent with R-G-C structure (lines 380-384)

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints/Guardrails` sections
- Emphasize test-first workflow
- Link to testing documentation

**Files Changed**: 1
**Lines Added**: ~35 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE7: Restructure Clean Code Implementer Agent (R-G-C Template)

**Action**: Rewrite Clean Code Implementer Agent with R-G-C structure (lines 386-390)

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints/Guardrails` sections
- Emphasize clean code principles
- Reference pattern documentation

**Files Changed**: 1
**Lines Added**: ~35 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE8: Restructure Feature Planner Agent (R-G-C Template)

**Action**: Rewrite Feature Planner Agent with R-G-C structure (lines 400-404)

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints/Guardrails` sections
- Emphasize small steps (<150 lines)
- Link to BDD and Example Mapping documentation

**Files Changed**: 1
**Lines Added**: ~35 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE9: Restructure Code Reviewer Agent (R-G-C Template)

**Action**: Rewrite Code Reviewer Agent with R-G-C structure (lines 406-408)

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints/Guardrails` sections
- Emphasize strict rule checking
- Reference CODING_AGENTS.md

**Files Changed**: 1
**Lines Added**: ~30 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

### Step CE10: Add Context Budget Guidelines

**Action**: Add section after "Library Discipline" (line ~165, before Context Engineering section)

**Changes**:

- Target context budget: < 30k tokens
- Strategies to stay within budget
- When to use `/clear` vs `/compact`
- Context hygiene practices

**Files Changed**: 1
**Lines Added**: ~25 lines
**Review Time**: < 2 minutes

---

### Step CE11: Implement Progressive Disclosure in Documentation Guide

**Action**: Update "During Work" documentation guide (lines 91-112) with context levels

**Changes**:

- Restructure table with "Context Level" column
- Add "Tier 1/2/3" indicators
- Add note: "Loaded on-demand via Read tool"
- Reference CONTEXT_ENGINEERING.md

**Files Changed**: 1
**Lines Modified**: ~20 lines
**Lines Added**: ~10 lines
**Review Time**: < 2 minutes

---

### Step CE12: Add Context Hygiene Section

**Action**: Add section before "After Completing Step" (line ~230)

**Changes**:

- When to use `/clear` (new tasks, switching features)
- When to use `/compact` (continuing related work)
- Context cleanup practices
- Archive strategies for old decisions

**Files Changed**: 1
**Lines Added**: ~30 lines
**Review Time**: < 2 minutes

---

## Phase 2: Medium Priority Enhancements (Steps CE13-CE15)

**Duration**: ~1-2 hours

**Goal**: Optimize performance and consistency through progressive disclosure.

### Step CE13: Update Claude Skills Framework Section

**Action**: Enhance section (lines 645-686) with progressive disclosure explanation

**Changes**:

- Explain progressive disclosure in skill context
- Add "When to Load" guidance
- Add tiered loading pattern (metadata → full → resources)
- Update examples with context level indicators

**Files Changed**: 1
**Lines Added**: ~40 lines
**Lines Modified**: ~10 lines
**Review Time**: < 3 minutes

---

### Step CE14: Add Context Optimization Checklist

**Action**: Add section near end of file for quick reference

**Changes**:

- Checklist for optimizing context before agent tasks
- Context budget validation
- Progressive disclosure checklist
- Guardrails verification

**Files Changed**: 1
**Lines Added**: ~25 lines
**Review Time**: < 2 minutes

---

### Step CE15: Enhance "During Work" Section with Context Best Practices

**Action**: Expand "During Work" section with context-specific guidance

**Changes**:

- Add context optimization sub-section
- Add tiered documentation access guidance
- Add progressive disclosure workflow
- Add context budget monitoring

**Files Changed**: 1
**Lines Added**: ~35 lines
**Lines Modified**: ~5 lines
**Review Time**: < 3 minutes

---

## Phase 3: Low Priority Polish (Steps CE16-CE18)

**Duration**: ~1 hour

**Goal**: Final polish and consistency improvements.

### Step CE16: Update SKILL.md Examples with R-G-C Template

**Action**: Update example SKILL.md format (lines 662-686) with R-G-C structure

**Changes**:

- Add `# Role`, `# Goal`, `# Constraints` to example
- Show emphasis pattern
- Link to CONTEXT_ENGINEERING.md

**Files Changed**: 1
**Lines Modified**: ~15 lines
**Lines Added**: ~10 lines
**Review Time**: < 2 minutes

---

### Step CE17: Add Context Engineering References to Footer

**Action**: Update "References" section (lines 809-829)

**Changes**:

- Add CONTEXT_ENGINEERING.md link
- Add external research references
- Add best practice sources

**Files Changed**: 1
**Lines Modified**: ~5 lines
**Lines Added**: ~10 lines
**Review Time**: < 1 minute

---

### Step CE18: Create Context Engineering Quick Reference

**Action**: Add "Quick Reference: Context Engineering" section near top

**Changes**:

- One-page summary of key concepts
- R-G-C template quick reference
- Four strategies summary
- Link to full CONTEXT_ENGINEERING.md

**Files Changed**: 1
**Lines Added**: ~30 lines
**Review Time**: < 2 minutes

---

## Success Criteria

### Context Efficiency

- ✅ Context Engineering section clearly explains key concepts
- ✅ All agents use R-G-C template for consistency
- ✅ Guardrails section centralizes critical rules
- ✅ Context budget guidelines established (< 30k tokens)

### Agent Reliability

- ✅ Instruction hierarchy emphasized to improve constraint compliance
- ✅ Guardrails with emphasis markers (CRITICAL, IMPORTANT)
- ✅ Progressive disclosure pattern implemented
- ✅ Context hygiene practices documented

### Developer Experience

- ✅ Clear workflow for context optimization
- ✅ Easy-to-identify critical rules
- ✅ Progressive disclosure reduces context overhead
- ✅ Quick reference for common context scenarios

---

## Expected Improvements

Based on research findings:

- **Context Efficiency**: 40-60% reduction in context size through progressive disclosure
- **Agent Reliability**: 30-50% improvement in constraint compliance (via guardrails + hierarchy emphasis)
- **Developer Experience**: Clearer agent expectations, easier debugging
- **Performance**: Fewer context-related errors, better adherence to Nexus principles

---

## Implementation Workflow

Follow AI Agent Workflow from AGENTS.md for each step:

### Before Starting Work

1. Read CONTEXT_ENG_PROGRESS.md
2. Identify current step status
3. Read CONTEXT_ENG_PLAN.md for step details
4. Update step status to "In Progress"

### During Work

1. Read relevant documentation (CONTEXT_ENGINEERING.md, AGENTS.md)
2. Implement changes following step limits
3. Verify changes meet requirements

### After Completing Step

1. **VALIDATION CHECKPOINT 1**: Run quality checks (if applicable)
2. **VALIDATION CHECKPOINT 2**: Review checklist
3. **VALIDATION CHECKPOINT 3**: Update CONTEXT_ENG_PROGRESS.md
4. **VALIDATION CHECKPOINT 4**: Update statistics
5. **VALIDATION CHECKPOINT 5**: Add to Recent Activity
6. **VALIDATION CHECKPOINT 6**: Update Next Steps
7. **VALIDATION CHECKPOINT 7**: Git commit with conventional format

---

## Step Limits (Strict)

**Maximum Per Step**:

- Files Changed: 3 files (AGENTS.md, CONTEXT_ENG_PLAN.md, CONTEXT_ENG_PROGRESS.md)
- Lines Added: 150 lines total
- Files Created: 2 files (none expected - all changes to existing files)
- Test Files: 0 files (documentation changes only)
- Review Time: < 5 minutes

**Each Step Must Include**:

- ✅ Files changed list with line counts
- ✅ Diff preview (max 50 lines)
- ✅ Changes meet step requirements
- ✅ Ready for review
- ✅ Rollback instructions (git revert commit)

---

## Risk Mitigation

### Technical Risks

1. **AGENTS.md Overgrowth**: Monitor file size, split if exceeds 1000 lines
2. **Breaking Changes**: Ensure changes maintain backward compatibility
3. **Consistency**: All agents must follow same R-G-C template

### Process Risks

1. **Step Size Violation**: Enforce <150 lines per step
2. **Review Bottlenecks**: Each step reviewable in <5 minutes
3. **Documentation Drift**: Keep AGENTS.md, CONTEXT_ENGINEERING.md, CODING_AGENTS.md in sync

---

## Timeline Estimate

| Phase                    | Steps        | Hours    | Review Time |
| ------------------------ | ------------ | -------- | ----------- |
| Phase 1: High Priority   | CE1-CE12     | ~3h      | ~30 min     |
| Phase 2: Medium Priority | CE13-CE15    | ~1-2h    | ~10 min     |
| Phase 3: Low Priority    | CE16-CE18    | ~1h      | ~10 min     |
| **Total**                | **CE1-CE18** | **5-6h** | **~50 min** |

---

## References

### Research Sources

- **Anthropic Context Engineering Guide** (2025)
- **Claude Code Best Practices** (2025)
- **Context Engineering: The Definitive Guide** (FlowHunt, 2025)
- **The Instruction Hierarchy** (Research Paper, 2023)
- **Kubiya AI Context Engineering Best Practices** (2025)
- **Multiple industry sources** (2024-2025)

### Internal Documentation

- [AGENTS.md](AGENTS.md) - AI agent system overview (being updated)
- [docs/engineering/context_engineering/CONTEXT_ENGINEERING.md](docs/engineering/context_engineering/CONTEXT_ENGINEERING.md) - Full context engineering guide (created)
- [CODING_AGENTS.md](CODING_AGENTS.md) - AI agent development guidelines

---

**Next Action**: Begin Phase 1, Step CE1: Add Context Engineering Section
