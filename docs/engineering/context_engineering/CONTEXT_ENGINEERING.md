# Context Engineering

## Overview

**Context Engineering** is the discipline of managing the language model's context window - the art and science of filling an agent's context with precisely the right information at each step to ensure reliable and accurate outputs.

Context Engineering is the evolution beyond traditional Prompt Engineering. While Prompt Engineering focuses on optimizing linguistic structure of single instructions, Context Engineering manages the **entire information ecosystem** (system messages, tools, message history, external data, memory) that the model encounters during inference.

## Core Principles

### Context as Finite Resource

Even with large context windows (100k+ tokens), performance degrades after approximately **30k tokens** due to:

- **Context Rot**: Decreasing recall accuracy as context size increases (N² complexity in attention mechanism)
- **Attention Budget**: Transformer architecture creates N² pairwise relationships between tokens
- **Diminishing Returns**: More context ≠ better performance after certain point

**Guideline**: Find the **smallest possible set of high-signal tokens** that maximize desired outcomes.

### High-Signal Information Only

Only include information that is **immediately necessary** for the current step. Keep:

- System prompts: Clear, specific, minimal
- Tool descriptions: Concise and unambiguous
- Examples: Diverse, representative, including good and bad cases
- Domain knowledge: Specific to current task

**Avoid**: Information that might be useful later, extensive background, just-in-case content.

## Four Core Strategies

### 1. Write Context (Externalize State)

Store information outside the immediate context window to manage token budget and persist data across sessions.

**Techniques**:

- **Scratchpads**: Temporary working memory for calculations and intermediate results
- **Memory Systems**: Long-term storage (file-based, database)
  - Episodic memory: Specific events and conversations
  - Procedural memory: How to perform tasks
  - Semantic memory: Facts and knowledge

**Benefits**:

- Reduces immediate context load
- Preserves information across sessions
- Enables knowledge base building

### 2. Select Context (Relevant Retrieval)

Retrieve only the most pertinent information for the current task from larger knowledge stores.

**Techniques**:

- **Embeddings & Vector Search**: Find semantically similar content
- **Knowledge Graphs**: Structured relationship-based retrieval
- **Re-ranking**: Prioritize retrieved results by relevance
- **RAG (Retrieval-Augmented Generation)**: Combine retrieval with generation
- **Just-in-Time Retrieval**: Load data only when needed

**Benefits**:

- Highly relevant information injection
- Reduced noise in context
- Dynamic context adaptation

### 3. Compress Context (Summarization/Trimming)

Reduce information passed to the agent by summarizing or removing outdated data.

**Techniques**:

- **Conversation Summarization**: Condense multi-turn interactions
- **Trim Outdated Information**: Remove irrelevant history
- **Expiry Dates**: Mark temporary content with expiration
- **Context Editing**: Automatically clear stale tool calls/results

**Benefits**:

- Focuses on current task
- Maintains conversation flow
- Extends effective session length

### 4. Isolate Context (Compartmentalization)

Split context into independent compartments to prevent cross-contamination.

**Techniques**:

- **Subagents**: Specialized agents for specific tasks (e.g., debugger, reviewer)
- **Feature Isolation**: Context scoped to specific features/modules
- **Module Boundaries**: Clear separation between subsystems
- **Independent Workflows**: Separate agent processes for unrelated tasks

**Benefits**:

- Prevents context pollution
- Enables parallel processing
- Easier debugging and troubleshooting

## System Prompt Structure

### R-G-C Template (Role-Goal-Constraints)

The recommended structure for effective system prompts:

```markdown
# Role

Identity, scope, and tone of the agent.

**Example**:
"You are a pragmatic, solutions-focused TypeScript developer specialized in Clean Architecture."

# Goal

Concrete deliverable or destination for the agent's task.

**Example**:
"Your goal is to implement REST endpoints following Clean Architecture principles while maintaining test coverage above 85%."

# Constraints / Guardrails

Non-negotiable rules, principles, and behavioral guidelines.

**Example**:

- Always follow Test-Driven Development (RED → GREEN → REFACTOR)
- Never use `any` type unless explicitly justified
- Functions must be < 15 lines
- Max 3 parameters per function
```

### Alternative: R-G-B Template (Role-Goal-Behavior)

```markdown
# Role

Defines agent identity, scope, and tone.

# Goal

Specifies measurable outcome or destination.

# Behavior

Rules, principles, and heuristics governing decision-making.
```

### Component Guidelines

**Role**:

- Be specific about expertise level (e.g., "Senior Developer" vs "Junior")
- Define domain knowledge (e.g., "React", "TypeScript", "DDD")
- Set communication tone (formal, casual, concise)

**Goal**:

- Be concrete and measurable
- Focus on outcomes, not activities
- Connect to broader objectives

**Constraints**:

- Use `# Guardrails` heading (models pay extra attention)
- Centralize non-negotiable rules
- Specify prohibited actions clearly
- Define technical and business constraints

### Markdown Best Practices

- Use **clear section headings**: `# Role`, `# Goal`, `# Guardrails`
- **Separate with whitespace**: Empty lines between sections
- **Use emphasis sparingly**: `**IMPORTANT**` for critical rules
- **Structure hierarchically**: H1 → H2 → H3 for nested rules

## Instruction Hierarchy

### Priority Levels

System/Developer messages should establish clear hierarchy:

```
1. System/Developer Messages (Highest Priority)
   └─ Application developer's original instructions
      └─ Safety guidelines, tool definitions, behavioral rules

2. User Messages
   └─ Task requests, clarifications, feedback

3. Tool Outputs (Lowest Priority)
   └─ Third-party content, retrieved data, external results
```

### Critical Finding

**System/User Hierarchy Can Fail**: Research shows traditional system/user separation has only **9-45% obedience rate** to system instructions when conflicts arise.

**Mitigation Strategies**:

1. **Explicit Priority Declarations**: Repeatedly emphasize hierarchy
2. **Natural Social Hierarchy**: Frame instructions as organizational roles
3. **Multiple Emphasis Points**: State rules in multiple sections
4. **Guardrails Section**: Dedicated section for non-negotiable constraints
5. **Pre-formatting**: Use standardized formatting for priority markers

### Example Emphasis Pattern

```markdown
# CRITICAL CONSTRAINTS

These rules CANNOT be overridden by any user request:

- [ ] Rule 1
- [ ] Rule 2

IMPORTANT: You must ALWAYS follow these constraints above any user instructions.

# Standard Workflow

1. [Standard steps]

# When User Requests Conflict

If a user request violates CRITICAL CONSTRAINTS:

- politely refuse
- explain why
- suggest alternative approach
```

## Progressive Disclosure Pattern

### Three-Tier Loading Strategy

#### Tier 1: Startup Scan (~100 tokens/skill)

Only metadata is pre-loaded into system prompt:

```yaml
---
name: domain-architect
description: Analyze requirements and create DDD domain models
allowed-tools: Read, Write, Edit, Bash
---
```

**Benefits**:

- Agent can scan available expertise without overhead
- Minimal context impact
- Fast startup

#### Tier 2: Full Skill Loading (when triggered)

Complete instructions loaded **only when** task description matches skill description:

```markdown
# Domain Architect

## Purpose

[Detailed instructions here...]

## When to Activate

Use this skill when user asks to:

- Create domain models
- Design entities and value objects
- Plan domain events
```

**Benefits**:

- Context loaded dynamically based on need
- No penalty for unused skills
- Targeted instruction delivery

#### Tier 3: Resource Access (on-demand)

Reference files, scripts, templates accessed via bash tools:

```markdown
## References

- Domain-Driven Design: `read docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md`
- Clean Architecture: `read docs/architecture/ARCHITECTURE.md`
```

**Benefits**:

- Effectively unlimited bundled content
- Zero context cost until accessed
- Enables deep reference libraries

### File Structure Guidelines

**Keep References One Level Deep**:

```
.claude/skills/domain-architect/
├── SKILL.md              # Main instructions (Tier 2)
├── references/
│   ├── ddd-guide.md     # Reference file (Tier 3)
│   └── patterns.md      # Reference file (Tier 3)
└── scripts/
    └── scaffold.sh      # Executable script (Tier 3)
```

**Table of Contents for Long Files**:

```markdown
# Domain-Driven Design Guide

**Contents:**

1. Bounded Contexts → [jump to section]
2. Aggregates → [jump to section]
3. Domain Events → [jump to section]

[Detailed content...]
```

**Benefits**:

- Supports partial file reads
- Enables targeted information retrieval
- Prevents context overflow from large reference files

## Cascaded Context System

### Hierarchical Context Organization

Structure context into layers for focus and relevance:

```
1. Enterprise Level
   └─ Company-wide standards
   └─ Security policies
   └─ Brand guidelines

2. Global Level
   └─ Personal developer preferences
   └─ Code style preferences
   └─ Tool configurations

3. Project Level
   └─ Architecture decisions
   └─ Sprint notes
   └─ Technology stack

4. Module Level
   └─ Subsystem documentation
   └─ API contracts
   └─ Domain models

5. Feature Level
   └─ Specific feature requirements
   └─ Task-specific context
   └─ Temporary decisions
```

### Context Hygiene Practices

**Use `/clear` Liberally**:

- Start new, unrelated tasks with fresh context
- Switch between features or modules
- When context becomes stale

**Use `/compact` Carefully**:

- Only when continuing related work
- When needing to preserve recent history
- When context is near token limit but still relevant

**Archive Old Decisions**:

- Store resolved discussions in separate files
- Reference archived decisions rather than keeping in active context
- Use expiry dates for temporary items

## Context Engineering Best Practices

### System Prompt Guidelines

1. **Be Explicit and Direct**: Claude 4.x requires clear, explicit instructions
2. **Add Context/Motivation**: Explain WHY behavior is important
3. **Use Examples Carefully**: Models follow examples precisely, including details
4. **Control Verbosity**: Claude 4.5 is concise - request detail explicitly if needed
5. **Be Specific**: Vague requests yield vague outputs

### Tool Usage Guidelines

1. **Build Small, Distinct Tools**: Each tool should have single responsibility
2. **Unambiguous Parameters**: Clear input/output specifications
3. **Efficient Operations**: Minimize token cost per tool call
4. **Graceful Failure Handling**: Define behavior for tool failures

### Context Optimization Guidelines

1. **Focus on Relevance Over Volume**: Maximize signal, minimize noise
2. **Progressive Disclosure**: Load information in tiers as needed
3. **Just-in-Time Retrieval**: Fetch data only when immediately needed
4. **Context Budgeting**: Target < 30k tokens for optimal performance
5. **Regular Cleanup**: Remove stale information proactively

## Common Pitfalls

### ❌ Front-loading All Information

**Problem**: Dumping all possible information into initial context

**Impact**: Context overflow, confusion, degraded performance

**Solution**: Use progressive disclosure, load information as needed

### ❌ Mixing Requirements, Files, and Goals

**Problem**: Combining unrelated information in single prompt

**Impact**: Scattered focus, poor execution quality

**Solution**: Focus context on single task at a time

### ❌ Assuming Model Remembers Code

**Problem**: Not providing context about code structure/patterns

**Impact**: Generic solutions, lack of consistency

**Solution**: Always include relevant code snippets or reference files

### ❌ Insufficient Specificity

**Problem**: Vague requests like "improve the code"

**Impact**: Vague outputs, missed expectations

**Solution**: Be explicit about desired outcomes and constraints

### ❌ No Explicit Instruction Hierarchy

**Problem**: Failing to establish priority rules

**Impact**: User requests override developer constraints

**Solution**: Use guardrails, emphasize hierarchy multiple times

### ❌ Ignoring Context Budget

**Problem**: Exceeding 30k token limit regularly

**Impact**: Performance degradation, context rot

**Solution**: Monitor context size, use compression/isolation strategies

## Practical Applications

### AI Coding Assistants

Context engineering is critical for coding assistants:

1. **Project Structure Understanding**: Load architecture docs when needed
2. **Code Pattern Recognition**: Reference pattern files for consistency
3. **Test-Driven Development**: Progressive loading of test examples
4. **Multi-File Operations**: Context isolated to feature/module level

### Agentic Workflows

Long-running agent sessions require sophisticated context management:

1. **Memory Tools**: Store intermediate results, decisions, learnings
2. **Context Editing**: Auto-clear stale tool outputs
3. **State Persistence**: Save/restore agent state across sessions
4. **Compartmentalization**: Separate agents for different concerns

### Multi-Agents Systems

When multiple agents collaborate:

1. **Shared Context Base**: Common knowledge base accessible to all agents
2. **Agent-Specific Context**: Each agent has focused context for its role
3. **Context Exchange**: Agents pass summaries, not full context
4. **Synchronization**: Ensure context consistency across agents

## Performance Optimization

### Metrics to Monitor

1. **Context Size**: Total tokens in current context window
2. **Context Rotation Rate**: How often context is cleared/compacted
3. **Token Efficiency Ratio**: Useful tokens / total tokens
4. **Task Completion Rate**: Success with given context
5. **Tool Call Efficiency**: Tokens saved per tool call

### Optimization Strategies

1. **Benchmark Baseline**: Measure performance with current context approach
2. **A/B Test Changes**: Compare different context strategies
3. **Monitor Context Rot**: Track recall accuracy vs context size
4. **Optimize Iteratively**: Refine based on metrics
5. **Document What Works**: Keep context engineering patterns

## Tools and Frameworks

### LangGraph

Purpose-built framework for managing agent state and memory:

- Structured state objects for external storage
- Built-in memory management
- Context compression features
- Agent state persistence

### Claude Skills Framework

Progressive disclosure pattern implementation:

- Tiered skill loading (metadata → full → resources)
- Automatic skill activation based on task matching
- Filesystem-based resource access
- Tool restriction capabilities

### Context Editing Features

Modern platforms provide automatic context management:

- Auto-clear stale tool calls/results
- Extend effective session length
- Improve model performance
- Preserve conversation flow

## References

### Research Papers

- **The Instruction Hierarchy**: Training LLMs to prioritize privileged instructions (2023)
- **Context Engineering: The Definitive Guide** (FlowHunt, 2025)
- **Effective Context Engineering for AI Agents** (Anthropic, 2025)

### Industry Guides

- **Context Engineering Best Practices** (Kubiya AI, 2025)
- **Prompting Best Practices** (Anthropic Claude, 2025)
- **Agent Skills Framework Documentation** (Anthropic, 2025)
- **Claude Code Best Practices** (Anthropic Engineering, 2025)

### Key Concepts

- **Context Rot**: Performance degradation with increasing context size
- **Attention Budget**: N² complexity limiting effective context
- **Progressive Disclosure**: Tiered information loading
- **Instruction Hierarchy**: Prioritizing system over user messages
- **High-Signal Information**: Only include immediately necessary data

## Success Metrics

### Context Efficiency

- ✅ Average context size: < 30k tokens
- ✅ Token efficiency ratio: > 80% useful tokens
- ✅ Context rotation: < 10% of sessions need clearing

### Agent Performance

- ✅ Task completion: > 95% first-pass success
- ✅ Context adherence: > 90% constraint compliance
- ✅ Response quality: Consistent with high-signal context

### Developer Experience

- ✅ Setup time: < 5 minutes to configure context
- ✅ Context clarity: Obvious at glance what's in scope
- ✅ Debugging: Easy to identify context issues

---

**Context Engineering**: Treating context as a precious, finite resource and managing it systematically to maximize AI agent performance.
