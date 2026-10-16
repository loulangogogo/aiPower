# Coding Collaboration Standards

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Basic Communication Standards

- All conversation, explanations, analysis, and feedback must be in Chinese.
- All code comments, documentation, and operational plans must be in Chinese.

- All generated text files must be UTF-8 encoded.
- After generating code, check for garbled Chinese text.
- If garbled text, anomalous characters, or encoding errors are found, fix them before producing output.

## 6. Commenting Standards

- Every generated function must include a function-level Chinese comment.
- Every key piece of logic must include the necessary Chinese explanatory comment.
- Comments must clearly describe:
  - Purpose and usage
  - Parameter descriptions
  - Return value description
  - Key business logic

## 7. Code Readability Standards

- Generated code must be:
  - Readable
  - Maintainable
  - Consistent with the existing project style
- Prohibited:
  - Meaningless abstraction
  - Excessive encapsulation
  - Unrequested extensibility

## 8. Prohibited Actions

- Refactoring the project without authorization;
- Modifying unrelated modules without authorization;
- Introducing new frameworks without authorization;
- Adding unrequested features without authorization;
- Changing the project structure without authorization;
- Changing the code style without authorization.
