# Windsurf Compatibility Analysis for Claude Code Skills

**Analysis Date:** 2026-01-22  
**Purpose:** Identify Claude Code-specific features in existing skills and document compatibility issues for Windsurf usage

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

### 2. dispatching-parallel-agents ⚠️ **MAJOR INCOMPATIBILITY**

**Claude Code Features Used:**
- **Subagent system** (core concept)
- **Task tool** for parallel dispatch
- **Forked contexts** for isolation

**Windsurf Compatibility:** ❌ **NOT COMPATIBLE**

**Issues:**
- Entire skill based on dispatching parallel subagents
- Windsurf has **no subagent system**
- Windsurf has **single conversation context only**
- No Task tool for parallel dispatch

**Impact:** Skill cannot be used in Windsurf as designed

**Recommendations:**
1. **Archive or disable this skill** - core concept doesn't translate
2. **Alternative approach:** Use sequential debugging with systematic-debugging skill
3. **Future:** If Windsurf adds subagent support, revisit

---

### 3. executing-plans ⚠️ **MODERATE INCOMPATIBILITY**

**Claude Code Features Used:**
- References `finishing-a-development-branch` skill (which uses git worktrees)
- Assumes separate session execution

**Windsurf Compatibility:** ⚠️ **PARTIALLY COMPATIBLE**

**Issues:**
- Designed for "separate session" execution pattern
- Windsurf doesn't have explicit session forking
- References to sub-skills work differently

**Impact:** Can be used but workflow differs

**Recommendations:**
1. **Adapt for single-session use** - Windsurf works better with continuous conversation
2. **Manual skill invocation** - Use `@executing-plans` to trigger
3. **Simplify batch execution** - Remove assumptions about session isolation

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

### 6. requesting-code-review ⚠️ **MAJOR INCOMPATIBILITY**

**Claude Code Features Used:**
- **Task tool** for dispatching code-reviewer subagent
- **Subagent system** (code-reviewer type)
- References `code-reviewer.md` template for subagent

**Windsurf Compatibility:** ❌ **NOT COMPATIBLE**

**Issues:**
- Core functionality requires dispatching subagent
- Windsurf has no Task tool
- Windsurf has no subagent system
- Template designed for subagent invocation

**Impact:** Cannot dispatch code reviewer as designed

**Recommendations:**
1. **Adapt to manual review process** - Create checklist instead of subagent
2. **Use external tools** - Integrate with GitHub PR reviews or other tools
3. **Self-review mode** - Convert subagent template to self-review checklist
4. **Alternative:** Use Windsurf's planning mode for structured review

---

### 7. subagent-driven-development ⚠️ **MAJOR INCOMPATIBILITY**

**Claude Code Features Used:**
- **Subagent system** (entire skill based on this)
- **Task tool** for dispatching implementer, spec reviewer, quality reviewer
- **Forked contexts** for isolated task execution
- References `implementer-prompt.md`, `spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md`

**Windsurf Compatibility:** ❌ **NOT COMPATIBLE**

**Issues:**
- Entire workflow based on dispatching multiple subagents per task
- "Fresh subagent per task" is core principle
- Windsurf has no subagent system
- No way to fork contexts

**Impact:** Skill fundamentally incompatible with Windsurf architecture

**Recommendations:**
1. **Archive this skill** - cannot be adapted without subagents
2. **Alternative workflow:** Use Windsurf's planning mode with manual checkpoints
3. **Sequential execution:** Implement tasks sequentially in single conversation
4. **Manual reviews:** Convert subagent reviews to self-review checklists

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

### 10. using-git-worktrees ✅ **COMPATIBLE**

**Claude Code Features Used:**
- References `CLAUDE.md` (Claude Code convention)

**Windsurf Compatibility:** ✅ **FULLY COMPATIBLE**

**Notes:**
- All git worktree commands work identically
- CLAUDE.md reference is just a convention (can be adapted)
- Windsurf can use same worktree patterns

**Recommendations:**
- Consider referencing `.windsurf/` directory conventions
- Otherwise works as-is

---

### 11. using-superpowers ⚠️ **MODERATE INCOMPATIBILITY**

**Claude Code Features Used:**
- **Skill tool** (explicitly mentioned)
- References Claude Code's skill invocation mechanism

**Windsurf Compatibility:** ⚠️ **PARTIALLY COMPATIBLE**

**Issues:**
- Line 16: "In Claude Code: Use the `Skill` tool"
- Windsurf skill invocation works differently
- **Automatic skill invocation is unreliable in Windsurf** (major limitation from research)

**Impact:** Core guidance needs platform-specific adaptation

**Recommendations:**
1. **Update line 16-18** to include Windsurf-specific instructions:
   ```markdown
   **In Claude Code:** Use the `Skill` tool. When you invoke a skill, its content is loaded and presented to you—follow it directly.
   
   **In Windsurf:** Skills are auto-discovered but automatic invocation is unreliable. Manually invoke with `@skill-name` syntax when needed. Skills load progressively.
   ```

2. **Add Windsurf-specific guidance:**
   - Emphasize manual skill invocation via `@skill-name`
   - Note that automatic invocation may fail
   - Recommend explicit skill references when reliability matters

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

### 13. writing-plans ⚠️ **MODERATE INCOMPATIBILITY**

**Claude Code Features Used:**
- References `executing-plans` and `subagent-driven-development` skills
- Assumes subagent execution options

**Windsurf Compatibility:** ⚠️ **PARTIALLY COMPATIBLE**

**Issues:**
- Lines 99-117: Offers "Subagent-Driven" vs "Parallel Session" execution
- Subagent-driven option not available in Windsurf
- References skills that don't work in Windsurf

**Impact:** Plan creation works, but execution handoff needs adaptation

**Recommendations:**
1. **Update execution handoff section** (lines 97-117):
   ```markdown
   ## Execution Handoff
   
   After saving the plan, offer execution choice:
   
   **In Claude Code:**
   - Subagent-Driven (this session) - Fresh subagent per task
   - Parallel Session (separate) - Batch execution with checkpoints
   
   **In Windsurf:**
   - Sequential execution in current conversation with planning mode
   - Use `@executing-plans` to implement task-by-task
   - Manual checkpoints between tasks
   ```

2. **Simplify for Windsurf** - Remove subagent references in execution options

---

### 14. writing-skills ⚠️ **MODERATE INCOMPATIBILITY**

**Claude Code Features Used:**
- **Subagent testing** (core testing methodology)
- References `~/.claude/skills` directory (Claude Code-specific)
- Line 12: "Personal skills live in agent-specific directories (`~/.claude/skills` for Claude Code)"
- Testing methodology assumes subagent dispatch

**Windsurf Compatibility:** ⚠️ **PARTIALLY COMPATIBLE**

**Issues:**
- Lines 532-560: RED-GREEN-REFACTOR testing requires subagents
- "Run pressure scenario with subagent" cannot be done in Windsurf
- Testing methodology needs adaptation
- Directory path is Claude Code-specific

**Impact:** Skill creation guidance works, but testing methodology incompatible

**Recommendations:**
1. **Update line 12** to include Windsurf path:
   ```markdown
   **Personal skills live in agent-specific directories:**
   - Claude Code: `~/.claude/skills`
   - Windsurf: `~/.codeium/windsurf/skills`
   - Codex: `~/.codex/skills`
   ```

2. **Adapt testing methodology** (lines 532-560):
   - Replace "subagent testing" with "conversation testing"
   - Test skills by using them in current conversation
   - Document behavior changes with/without skill loaded
   - Note: Less rigorous than subagent isolation, but best available in Windsurf

3. **Add Windsurf-specific testing notes:**
   - Manual skill invocation required for reliable testing
   - Test with `@skill-name` syntax
   - Verify skill loads and affects behavior

---

## Summary by Compatibility Level

### ✅ Fully Compatible (6 skills)
No changes needed, work as-is:
1. `brainstorming`
2. `finishing-a-development-branch`
3. `receiving-code-review`
4. `systematic-debugging`
5. `test-driven-development`
6. `verification-before-completion`

### ⚠️ Partially Compatible (5 skills)
Work with adaptations or have limitations:
1. `executing-plans` - Adapt for single-session use
2. `using-git-worktrees` - Minor path convention updates
3. `using-superpowers` - Add Windsurf invocation guidance
4. `writing-plans` - Update execution handoff options
5. `writing-skills` - Adapt testing methodology, update paths

### ❌ Not Compatible (3 skills)
Cannot be used as designed:
1. `dispatching-parallel-agents` - Requires subagent system
2. `requesting-code-review` - Requires subagent dispatch
3. `subagent-driven-development` - Entire workflow based on subagents

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

## Recommended Actions

### Immediate Actions

1. **Disable incompatible skills** - Prevent confusion:
   - Move to `skills/_archived/` or add `.disabled` suffix:
     - `dispatching-parallel-agents.disabled/`
     - `requesting-code-review.disabled/`
     - `subagent-driven-development.disabled/`

2. **Update `using-superpowers`** - Critical for skill discovery:
   - Add Windsurf-specific invocation guidance
   - Document unreliable auto-invocation
   - Emphasize manual `@skill-name` syntax

3. **Update `writing-skills`** - For future skill creation:
   - Add Windsurf skills directory path
   - Adapt testing methodology for no-subagent environment

### Medium-Term Actions

4. **Adapt partially compatible skills:**
   - `executing-plans` - Remove session forking assumptions
   - `writing-plans` - Update execution handoff for Windsurf
   - `using-git-worktrees` - Minor path updates

5. **Create Windsurf-specific alternatives:**
   - `windsurf-code-review` - Manual review checklist (replaces `requesting-code-review`)
   - `windsurf-sequential-development` - Single-conversation workflow (replaces `subagent-driven-development`)

### Long-Term Actions

6. **Monitor Windsurf updates:**
   - Watch for subagent system addition
   - Watch for hooks system addition
   - Re-enable skills when features become available

7. **Contribute improvements:**
   - If skills become platform-agnostic, consider contributing back
   - Share Windsurf adaptations with community

---

## Conclusion

**6 of 14 skills (43%)** work without modification in Windsurf.

**5 of 14 skills (36%)** work with adaptations or have limitations.

**3 of 14 skills (21%)** cannot be used due to fundamental architectural differences.

The primary incompatibility is the **subagent system**, which is core to Claude Code's architecture but absent in Windsurf. Skills that rely on parallel subagent dispatch, context isolation, or the Task tool cannot be adapted without fundamental rewrites.

For Windsurf usage, focus on:
- ✅ **Process/methodology skills** (debugging, TDD, verification) - fully compatible
- ⚠️ **Workflow skills** (planning, execution) - adapt for single-context use
- ❌ **Subagent-dependent skills** - disable or replace with manual alternatives

The most critical update is to `using-superpowers` to document Windsurf's unreliable automatic skill invocation and emphasize manual triggering via `@skill-name`.
