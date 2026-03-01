---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully** - Don't skip past errors or warnings. Read stack traces completely.
2. **Reproduce Consistently** - Can you trigger it reliably? What are the exact steps?
3. **Check Recent Changes** - What changed that could cause this? Git diff, recent commits.
4. **Gather Evidence in Multi-Component Systems** - Add diagnostic instrumentation at each component boundary. Run once to gather evidence showing WHERE it breaks.
5. **Trace Data Flow** - Where does bad value originate? What called this with bad value? Keep tracing up until you find the source.

### Phase 2: Pattern Analysis

1. Find working examples similar to what's broken
2. Compare against references - read completely, don't skim
3. Identify differences - list every difference, however small
4. Understand dependencies

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis** - "I think X is the root cause because Y"
2. **Test Minimally** - Make the SMALLEST possible change to test hypothesis
3. **Verify Before Continuing** - Did it work? Yes → Phase 4. No → form NEW hypothesis.
4. **When You Don't Know** - Say "I don't understand X". Don't pretend to know.

### Phase 4: Implementation

1. **Create Failing Test Case** - Simplest possible reproduction, automated if possible
2. **Implement Single Fix** - Address the root cause identified. ONE change at a time.
3. **Verify Fix** - Test passes? No other tests broken? Issue actually resolved?
4. **If Fix Doesn't Work** - STOP. If ≥ 3 fixes tried: question the architecture.

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "One more fix attempt" (when already tried 2+)

**ALL of these mean: STOP. Return to Phase 1.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem |
