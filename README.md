# Windsurf Skills - Complete Guide

## What Are Skills in Windsurf?

**Skills** are reusable guides that provide proven techniques, patterns, methodologies, and best practices for specific development tasks. They teach Cascade (Windsurf's AI agent) how to approach problems using established workflows and domain expertise.

### Core Concept

Instead of Cascade figuring out methodologies from scratch each time, skills provide structured guidance on how to approach specific types of tasks - like debugging systematically, following TDD principles, or conducting code reviews. Think of them as reference manuals that shape how Cascade works.

## How Skills Work

### 1. Discovery & Storage

Windsurf automatically discovers skills from:

- **User-level**: `~/.codeium/windsurf/skills/` (personal skills, global across all projects)
- **Project-level**: `.windsurf/skills/` (project-specific skills, if implemented)

Each skill is stored in its own folder with a `SKILL.md` file:
```
~/.codeium/windsurf/skills/
  ├── brainstorming/
  │   └── SKILL.md
  ├── test-driven-development/
  │   ├── SKILL.md
  │   └── testing-anti-patterns.md
  └── systematic-debugging/
      └── SKILL.md
```

### 2. Progressive Disclosure

Skills use a progressive loading strategy to manage context efficiently:

1. **Startup**: Skill metadata (name, description) loads automatically
2. **Invocation**: Full skill instructions load when triggered
3. **Resources**: Supporting files load only as needed

### 3. Invocation

**@ syntax**: `@skill-name`

Examples:
- `@test-driven-development` - Load TDD methodology
- `@systematic-debugging` - Load debugging approach
- `@requesting-code-review` - Load code review process

**Critical limitation**: Automatic skill invocation is **unreliable** in Windsurf. Always use explicit `@skill-name` invocation when you need guaranteed skill loading.

### 4. Execution Process

When you invoke a skill, Cascade:

1. **Loads skill content** - Reads the SKILL.md file and any supporting resources
2. **Follows instructions** - Applies the methodology/guidance to current task
3. **Maintains context** - Keeps skill guidance active throughout the conversation
4. **Adapts to situation** - Applies principles flexibly to your specific context

## Skill File Structure

Skills are markdown files with YAML frontmatter:

```markdown
---
name: skill-name
description: Use when [triggering condition] - describes ONLY when to use
---

# Skill Title

## Overview
Brief explanation of what this skill provides

**Core principle:** One-sentence guiding principle

## When to Use
- Specific triggering conditions
- Symptoms that indicate this skill applies

## The Process/Methodology
Step-by-step guidance or principles to follow

## Examples
Concrete examples showing the skill in action

## Red Flags
Common mistakes or anti-patterns to avoid
```

### YAML Frontmatter Fields

- **`name`** (required) - Skill identifier (letters, numbers, hyphens only)
- **`description`** (required) - When to use this skill (max 1024 chars total frontmatter)
  - Must start with "Use when..."
  - Focus on triggering conditions, not what it does
  - Written in third person

### Supporting Files

Skills can include additional files for:
- **Heavy reference material** (100+ lines) - API docs, comprehensive syntax guides
- **Reusable tools** - Scripts, utilities, templates
- **Examples** - Detailed code examples

Keep inline: Principles, concepts, short code patterns (<50 lines)

## Creating Skills

### Skill Types

1. **Technique** - Concrete method with steps (e.g., `using-git-worktrees`)
2. **Pattern** - Way of thinking about problems (e.g., `brainstorming`)
3. **Reference** - API docs, syntax guides, tool documentation

### When to Create a Skill

**Create when:**
- Technique wasn't intuitively obvious to you
- You'd reference this again across projects
- Pattern applies broadly (not project-specific)
- Others would benefit from this knowledge

**Don't create for:**
- One-off solutions
- Standard practices well-documented elsewhere
- Project-specific conventions (use project rules/memories instead)
- Mechanical constraints (automate those, don't document)

### Creation Process (TDD for Documentation)

Skills follow Test-Driven Development principles applied to documentation:

#### RED Phase - Write Failing Test
1. Create pressure scenario WITHOUT the skill loaded
2. Document exact behavior - what choices did Cascade make?
3. Identify patterns in rationalizations/failures

**In Windsurf**: Start new conversation without invoking skill, present scenario, document behavior

#### GREEN Phase - Write Minimal Skill
1. Write skill addressing those specific rationalizations
2. Don't add extra content for hypothetical cases
3. Test same scenarios WITH skill loaded (`@skill-name`)
4. Verify Cascade now complies

#### REFACTOR Phase - Close Loopholes
1. Identify NEW rationalizations from testing
2. Add explicit counters (for discipline skills)
3. Build rationalization table from all test iterations
4. Re-test until bulletproof

### Best Practices

✅ **DO:**
- Use only letters, numbers, hyphens in skill names
- Start description with "Use when..."
- Include specific triggers/symptoms in description
- Write in third person
- Test skill before deploying
- Include clear examples
- Add "Red Flags" section for common mistakes

❌ **DON'T:**
- Use parentheses or special characters in names
- Write narrative storytelling ("In session X, we found...")
- Create multi-language dilution (example-js.js, example-py.py)
- Skip testing the skill
- Make it too generic or too specific

## Skill Discovery Workflow

How Cascade finds and uses your skills:

1. **Encounters problem** ("tests are flaky")
2. **Searches skills** (description matches problem)
3. **Finds skill** (description triggers on keywords)
4. **Scans overview** (is this relevant?)
5. **Reads patterns** (quick reference table)
6. **Loads example** (only when implementing)

**Optimization tip**: Put searchable keywords early and often in your skill content.

## Use Cases

### Process Methodologies
- **`brainstorming`** - Explore ideas into designs before coding
- **`test-driven-development`** - TDD methodology and principles
- **`systematic-debugging`** - Root cause investigation approach
- **`verification-before-completion`** - Verify work before claiming done

### Workflow Guidance
- **`executing-plans`** - Implement plans with review checkpoints
- **`writing-plans`** - Create detailed implementation plans
- **`requesting-code-review`** - Structured self-review process
- **`receiving-code-review`** - Handle code review feedback

### Development Practices
- **`using-git-worktrees`** - Isolated workspace management
- **`finishing-a-development-branch`** - Complete and merge work
- **`writing-skills`** - Create new skills using TDD approach

### Meta Skills
- **`using-skills`** - How to find and use skills effectively

## Skills vs. Workflows

| Feature | Skills | Workflows |
|---------|--------|-----------|
| **Invocation** | `@skill-name` | `/workflow-name` |
| **Purpose** | Provide methodology/guidance | Automate specific task sequences |
| **Execution** | Contextual guidance for agent | Sequential steps, autonomous |
| **Storage** | `~/.codeium/windsurf/skills/*/SKILL.md` | `.windsurf/workflows/*.md` |
| **Scope** | Reusable techniques/patterns | Task-specific automation |
| **Auto-invoke** | Unreliable (manual @ recommended) | No (manual slash command) |
| **When to use** | Learning methodologies/best practices | Repetitive multi-step tasks |

**Use skills when:** You want to teach Cascade a methodology or pattern (TDD, debugging, code review principles)

**Use workflows when:** You have a specific sequence of steps to automate (deployment, testing, PR review)

## Important Windsurf Limitations

### Unreliable Automatic Invocation ⚠️

**Issue**: Windsurf frequently fails to auto-invoke skills even when requests clearly match the description.

**Impact**: You must manually trigger skills via `@skill-name` for reliability.

**Workaround**: Always use explicit skill references when reliability matters.

### No Subagent System ❌

**Issue**: Windsurf has single conversation context only (unlike Claude Code which has subagents).

**Impact**: Skills that relied on subagent dispatch in Claude Code have been adapted for single-context use.

**Example**: `requesting-code-review` was converted from subagent dispatch to self-review checklist.

### Single Context Only ⚠️

**Issue**: No context forking or isolation between conversations.

**Impact**: Long conversations can lead to context bloat.

**Workaround**: Start new conversations for major tasks to keep context fresh.

## This Repository's Skills

### ✅ Fully Compatible Skills (14 total)

All skills in this repository have been adapted for Windsurf compatibility:

#### Process Methodologies (6 skills)
- **`brainstorming`** - Explore ideas into designs
- **`systematic-debugging`** - Root cause investigation
- **`test-driven-development`** - TDD methodology
- **`verification-before-completion`** - Verify before claiming done
- **`receiving-code-review`** - Handle review feedback
- **`requesting-code-review`** - Self-review checklist (converted from subagent)

#### Workflow Management (3 skills)
- **`executing-plans`** - Batch execution with checkpoints
- **`writing-plans`** - Create implementation plans
- **`finishing-a-development-branch`** - Complete and merge work

#### Development Practices (2 skills)
- **`using-git-worktrees`** - Isolated workspace management
- **`writing-skills`** - Create new skills using TDD

#### Design & UI (2 skills)
- **`designing-frontends`** - Creative direction, bold aesthetics, unique visual identity
- **`ui-ux-pro-max`** - Systematic UX rules, accessibility, pre-delivery checklists

#### Meta Skills (1 skill)
- **`using-skills`** - How to find and use skills

### ❌ Deleted Skills (2 total)

These skills were removed due to fundamental incompatibility with Windsurf:

- **`dispatching-parallel-agents`** - Required subagent system
- **`subagent-driven-development`** - Entire workflow based on subagents

## Adaptation History

**Date**: 2026-01-22

All Claude Code skills have been successfully adapted for Windsurf:
- **6 skills** - No changes needed (pure methodology)
- **5 skills** - Updated for Windsurf (removed subagent references, updated paths)
- **1 skill** - Converted approach (requesting-code-review: subagent → self-review)
- **2 skills** - Deleted (fundamental incompatibility)

**Result**: 12 of 14 skills (86%) now fully compatible with Windsurf

See `WINDSURF_COMPATIBILITY_ANALYSIS.md` for detailed adaptation documentation.

---

## Getting Started

1. **Browse available skills** - Review the skill folders in this directory
2. **Invoke a skill** - Use `@skill-name` when you need guidance
3. **Follow the methodology** - Apply the skill's principles to your task
4. **Create your own** - Use `@writing-skills` to learn how to create new skills

**Remember**: Always use explicit `@skill-name` invocation for reliability in Windsurf!