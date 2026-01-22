# Windsurf Compatibility Analysis for Claude Code Skills

**Analysis Date:** 2026-01-22  
**Last Updated:** 2026-01-22 (Post-Adaptation)  
**Purpose:** Document Claude Code skill adaptations for Windsurf compatibility

**Status:** ✅ All incompatible skills removed, all partially compatible skills updated

---

## Executive Summary

These skills were designed for Claude Code and contain several features that **do not exist or work differently in Windsurf**:

### Critical Incompatibilities
1. **Subagent System** - Core feature in 5+ skills, **does not exist in Windsurf**
2. **Hooks System** - Referenced but **not available in Windsurf**
3. **Task Tool** - Used for subagent dispatch, **not available in Windsurf**
4. **Skill Tool** - Invocation mechanism differs between platforms
5. **Automatic Skill Invocation** - **Unreliable in Windsurf**, requires manual triggering

### Impact Assessment
- **High Impact:** 5 skills heavily dependent on subagents
- **Medium Impact:** 3 skills reference subagents optionally
- **Low Impact:** 6 skills work with minor adaptations

---

## Skill-by-Skill Analysis

### 1. brainstorming ✅ **COMPATIBLE**

**Claude Code Features Used:** None  
**Windsurf Compatibility:** Fully compatible

**Notes:**
- Pure process guidance skill
- References other skills (`using-git-worktrees`, `writing-plans`) but doesn't use Claude Code-specific features
- Works as-is in Windsurf

**Recommendations:** No changes needed

---

### 2. dispatching-parallel-agents ✅ **DELETED**

**Claude Code Features Used:**
- **Subagent system** (core concept)
- **Task tool** for parallel dispatch
- **Forked contexts** for isolation

**Windsurf Compatibility:** ❌ **NOT COMPATIBLE**

**Action Taken:** ✅ **Skill deleted** - Entire skill folder removed

**Reason:** Entire skill based on dispatching parallel subagents, which is fundamentally incompatible with Windsurf's single-context architecture

---

### 3. executing-plans ✅ **UPDATED**

**Claude Code Features Used:**
- References `finishing-a-development-branch` skill
- Assumed separate session execution

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Removed "separate session" from description
2. ✅ Updated skill reference from `superpowers:finishing-a-development-branch` to `@finishing-a-development-branch`

**Result:** Works seamlessly in Windsurf for batch execution with review checkpoints

---

### 4. finishing-a-development-branch ✅ **COMPATIBLE**

**Claude Code Features Used:** None (pure git operations)

**Windsurf Compatibility:** Fully compatible

**Notes:**
- All git commands work identically
- No Claude Code-specific features
- Test verification works in both platforms

**Recommendations:** No changes needed

---

### 5. receiving-code-review ✅ **COMPATIBLE**

**Claude Code Features Used:** None

**Windsurf Compatibility:** Fully compatible

**Notes:**
- Pure guidance on handling feedback
- No platform-specific features
- Communication patterns apply to both

**Recommendations:** No changes needed

---

### 6. requesting-code-review ✅ **CONVERTED**

**Claude Code Features Used:**
- **Task tool** for dispatching code-reviewer subagent
- **Subagent system** (code-reviewer type)
- References `code-reviewer.md` template for subagent

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Converted from subagent dispatch to structured self-review process
2. ✅ Integrated checklist directly into SKILL.md (removed separate code-reviewer.md)
3. ✅ Added clear output format for documenting findings
4. ✅ Kept severity categories (Critical/Important/Minor)
5. ✅ Changed from "dispatch reviewer" to "perform review against checklist"

**Result:** Now provides comprehensive self-review guidance with same rigor as original subagent approach

---

### 7. subagent-driven-development ✅ **DELETED**

**Claude Code Features Used:**
- **Subagent system** (entire skill based on this)
- **Task tool** for dispatching implementer, spec reviewer, quality reviewer
- **Forked contexts** for isolated task execution

**Windsurf Compatibility:** ❌ **NOT COMPATIBLE**

**Action Taken:** ✅ **Skill deleted** - Entire skill folder removed

**Reason:** Entire workflow based on "fresh subagent per task" principle, which is fundamentally incompatible with Windsurf's architecture. Cannot be adapted without complete rewrite.

---

### 8. systematic-debugging ✅ **COMPATIBLE**

**Claude Code Features Used:** None

**Windsurf Compatibility:** Fully compatible

**Notes:**
- Pure debugging methodology
- References `test-driven-development` skill (also compatible)
- All debugging techniques work identically
- Supporting files (root-cause-tracing.md, etc.) are methodology, not platform-specific

**Recommendations:** No changes needed

---

### 9. test-driven-development ✅ **COMPATIBLE**

**Claude Code Features Used:** None

**Windsurf Compatibility:** Fully compatible

**Notes:**
- Pure TDD methodology
- All test commands work identically
- No platform-specific features
- References `testing-anti-patterns.md` (methodology file)

**Recommendations:** No changes needed

---

### 10. using-git-worktrees ✅ **UPDATED**

**Claude Code Features Used:**
- References `CLAUDE.md` (Claude Code convention)
- Referenced `subagent-driven-development`

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Updated config check to include `.windsurf/rules.md` alongside CLAUDE.md
2. ✅ Removed reference to `subagent-driven-development` skill
3. ✅ Updated skill references to use `@` syntax

**Result:** Works seamlessly with Windsurf project conventions

---

### 11. using-superpowers → using-skills ✅ **UPDATED & RENAMED**

**Claude Code Features Used:**
- **Skill tool** (explicitly mentioned)
- References Claude Code's skill invocation mechanism

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Renamed skill from `using-superpowers` to `using-skills`
2. ✅ Replaced Claude Code-specific instructions with Windsurf-specific guidance
3. ✅ Documented automatic skill discovery from `~/.codeium/windsurf/skills/`
4. ✅ Added warning about unreliable automatic invocation
5. ✅ Emphasized manual `@skill-name` invocation for reliability
6. ✅ Simplified process flow (removed dot diagram, added numbered steps)

**Result:** Now provides clear Windsurf-specific skill usage guidance

---

### 12. verification-before-completion ✅ **COMPATIBLE**

**Claude Code Features Used:** None

**Windsurf Compatibility:** Fully compatible

**Notes:**
- Pure verification methodology
- All verification commands work identically
- No platform-specific features

**Recommendations:** No changes needed

---

### 13. writing-plans ✅ **UPDATED**

**Claude Code Features Used:**
- References `executing-plans` and `subagent-driven-development` skills
- Assumed subagent execution options

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Updated plan header from `superpowers:executing-plans` to `@executing-plans`
2. ✅ Removed "Subagent-Driven" and "Parallel Session" execution options
3. ✅ Simplified to single execution approach using `@executing-plans`
4. ✅ Added recommendation to use `@requesting-code-review` after each batch

**Result:** Plans now guide toward Windsurf-compatible execution workflows

---

### 14. writing-skills ✅ **UPDATED**

**Claude Code Features Used:**
- **Subagent testing** (core testing methodology)
- References `~/.claude/skills` directory (Claude Code-specific)
- Testing methodology assumed subagent dispatch

**Windsurf Compatibility:** ✅ **NOW FULLY COMPATIBLE**

**Changes Made:**
1. ✅ Added Windsurf skills directory: `~/.codeium/windsurf/skills/`
2. ✅ Updated TDD mapping table: "Pressure scenario with subagent" → "Pressure scenario in conversation"
3. ✅ Changed CLAUDE.md reference to "project rules/memories"
4. ✅ Updated RED-GREEN-REFACTOR methodology for no-subagent environment
5. ✅ Added Windsurf-specific testing guidance (start new conversation without skill, test with `@skill-name`)
6. ✅ Updated skill reference from `superpowers:test-driven-development` to `@test-driven-development`

**Result:** Skill creation and testing now works within Windsurf's single-context architecture

---

## Summary by Compatibility Level

### ✅ Fully Compatible - No Changes (6 skills)
Work as-is in Windsurf:
1. `brainstorming`
2. `finishing-a-development-branch`
3. `receiving-code-review`
4. `systematic-debugging`
5. `test-driven-development`
6. `verification-before-completion`

### ✅ Updated for Windsurf (5 skills)
Adapted and now fully compatible:
1. `executing-plans` - ✅ Removed session forking assumptions
2. `using-git-worktrees` - ✅ Updated config references, removed subagent refs
3. `using-superpowers` → `using-skills` - ✅ Renamed, added Windsurf guidance
4. `writing-plans` - ✅ Updated execution handoff options
5. `writing-skills` - ✅ Adapted testing methodology, added Windsurf paths

### ✅ Converted for Windsurf (1 skill)
Redesigned to work without subagents:
1. `requesting-code-review` - ✅ Converted to self-review checklist

### ❌ Deleted - Not Compatible (2 skills)
Removed due to fundamental incompatibility:
1. `dispatching-parallel-agents` - ❌ Deleted (requires subagent system)
2. `subagent-driven-development` - ❌ Deleted (entire workflow based on subagents)

---

## Key Windsurf Limitations from Research

Based on the Perplexity research, these are the critical limitations affecting skill usage:

### 1. **Unreliable Automatic Skill Invocation** ⚠️
- **Issue:** Windsurf frequently fails to auto-invoke skills even when requests clearly match
- **Impact:** Users must manually trigger skills via `@skill-name`
- **Workaround:** Always use explicit skill references for reliability

### 2. **No Subagent System** ❌
- **Issue:** Windsurf has single conversation context only
- **Impact:** Cannot dispatch isolated subagents for parallel work or specialized tasks
- **Workaround:** Sequential execution in single conversation

### 3. **No Hooks System** ❌
- **Issue:** Cannot automate actions after code changes (formatters, tests, linters)
- **Impact:** Automation must be built into skills or done manually
- **Workaround:** Explicit commands in skills

### 4. **No Task Tool** ❌
- **Issue:** No mechanism to dispatch subagents
- **Impact:** Skills relying on Task tool cannot work
- **Workaround:** None - fundamental architectural difference

### 5. **Single Context Only** ⚠️
- **Issue:** No context forking or isolation
- **Impact:** Long conversations can lead to context bloat
- **Workaround:** Start new conversations for major tasks

### 6. **Less Cost Visibility** ⚠️
- **Issue:** No `/cost` command equivalent
- **Impact:** Harder to manage token consumption
- **Workaround:** Be mindful of conversation length

---

## Actions Completed

### ✅ All Immediate Actions Complete

1. ✅ **Deleted incompatible skills:**
   - `dispatching-parallel-agents/` - Removed
   - `subagent-driven-development/` - Removed

2. ✅ **Converted incompatible skill:**
   - `requesting-code-review` - Converted to self-review checklist
   - Removed `code-reviewer.md` template (integrated into SKILL.md)

3. ✅ **Updated `using-superpowers` → `using-skills`:**
   - Renamed skill folder
   - Added Windsurf-specific invocation guidance
   - Documented unreliable auto-invocation
   - Emphasized manual `@skill-name` syntax

4. ✅ **Updated `writing-skills`:**
   - Added Windsurf skills directory path
   - Adapted testing methodology for no-subagent environment

5. ✅ **Adapted all partially compatible skills:**
   - `executing-plans` - Removed session forking assumptions
   - `writing-plans` - Updated execution handoff for Windsurf
   - `using-git-worktrees` - Updated config references

### Ongoing Actions

6. **Monitor Windsurf updates:**
   - Watch for subagent system addition
   - Watch for hooks system addition
   - Consider re-implementing deleted skills if features become available

7. **Maintain compatibility:**
   - Test skills regularly in Windsurf
   - Update as Windsurf evolves
   - Share adaptations with community if useful

---

## Conclusion

**Final Status: 12 of 14 skills (86%) now fully compatible with Windsurf**

### Compatibility Breakdown
- ✅ **6 skills (43%)** - No changes needed
- ✅ **5 skills (36%)** - Updated and now fully compatible
- ✅ **1 skill (7%)** - Converted to Windsurf-compatible approach
- ❌ **2 skills (14%)** - Deleted due to fundamental incompatibility

### Key Adaptations Made

1. **Removed subagent dependencies** - All references to Task tool, subagent dispatch, and context forking removed or replaced
2. **Updated skill references** - Changed from `superpowers:skill-name` to `@skill-name` syntax
3. **Simplified workflows** - Adapted multi-session/parallel patterns to single-context sequential execution
4. **Added Windsurf-specific guidance** - Documented platform differences, invocation methods, and limitations

### Remaining Limitations

Windsurf users should be aware of:
- **No subagent system** - Cannot dispatch isolated agents for parallel work
- **Unreliable auto-invocation** - Must use explicit `@skill-name` for reliability
- **Single context only** - Long conversations may require starting fresh for major tasks
- **No hooks system** - Automation must be explicit in skills

### Result

All skills in `~/.codeium/windsurf/skills/` are now fully functional in Windsurf. The skill library provides comprehensive guidance for:
- ✅ Process methodology (brainstorming, debugging, TDD, verification)
- ✅ Workflow management (planning, execution, code review)
- ✅ Development practices (git worktrees, skill creation)

Users can confidently use the entire skill library with the understanding that some workflows (like code review) are now self-directed rather than subagent-assisted.
