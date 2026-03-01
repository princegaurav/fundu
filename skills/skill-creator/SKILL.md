---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, update or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

A skill for creating new skills and iteratively improving them.

The process of creating a skill:

1. Decide what you want the skill to do and roughly how it should do it
2. Write a draft of the skill
3. Create a few test prompts and run through the skill on them
4. Help the user evaluate the results both qualitatively and quantitatively
5. Rewrite the skill based on feedback
6. Repeat until satisfied

Your job when using this skill is to figure out where the user is in this process and then jump in and help them progress through these stages.

## Creating a Skill

### Capture Intent

Start by understanding the user's intent. If the current conversation already contains a workflow the user wants to capture, extract answers from the conversation history first.

1. What should this skill enable the agent to do?
2. When should this skill trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases to verify the skill works?

### Write the SKILL.md

Based on the user interview, fill in these components:

**Frontmatter:**
```yaml
---
name: skill-name
description: "Trigger description — when to use this skill, key phrases, actions"
---
```

**Body:**
- Overview of what the skill does
- When to use it (trigger conditions)
- Step-by-step instructions
- Examples
- Edge cases

### SKILL.md Best Practices

- **Description is the trigger**: The `description` field is how the agent decides whether to use the skill. Make it specific about:
  - What actions it handles (verbs: "create", "build", "analyze")
  - What objects it applies to (nouns: "React component", "PDF file")
  - What phrases should trigger it
- **Keep instructions actionable**: Write instructions the agent can follow directly
- **Include examples**: Concrete examples reduce ambiguity
- **Test edge cases**: Think about what might go wrong

### Skill Structure

```
skills/
  my-skill/
    SKILL.md          # Main skill file (required)
    rules/            # Optional: additional reference files
      rule-1.md
      rule-2.md
```

### Testing the Skill

After writing the skill, test it by:

1. Asking prompts that SHOULD trigger it — verify it activates
2. Asking prompts that SHOULD NOT trigger it — verify it doesn't activate
3. Running through the skill's instructions manually
4. Checking the output matches what was intended

### Iterating

After testing:
- If triggering is wrong → revise the `description` field
- If instructions are unclear → add examples or rewrite steps
- If output is wrong → add constraints or examples
- Repeat until satisfied

## Installing the New Skill

Once the skill is ready, add it to the `skills/` directory of the plugin:

```
skills/
  your-new-skill/
    SKILL.md
```

## Communicating with the User

Pay attention to context cues to understand how technical the user is. Explain terms like "YAML frontmatter" or "eval" if unsure whether the user knows them.
