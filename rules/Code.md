# Coding Collaboration Standards

## 1. Think Before Coding

**Do not assume. Do not hide confusion. Present trade-offs.**

LLMs routinely settle on one interpretation silently and then execute it. This principle forces the reasoning into the open:

- **State your assumptions explicitly** — when unsure, ask rather than guess
- **Present multiple interpretations** — when something is ambiguous, do not quietly pick one
- **Disagree when warranted** — if a simpler approach exists, say so
- **Stop when confused** — point out what is unclear and ask for clarification

## 2. Simplicity First

**Solve the problem with the least code. Do not speculate about the future.**

Counteract the pull toward over-engineering:

- Do not add features beyond what was requested
- Do not create abstractions for one-off code
- Do not add unrequested "flexibility" or "configurability"
- Do not write error handling for scenarios that cannot occur
- If 200 lines can be rewritten in 50, rewrite it

**Test:** would a senior engineer consider this overly complex? If so, simplify.

## 3. Precise Edits

**Touch only what must be touched. Clean up only your own mess.**

When editing existing code:

- Do not "improve" adjacent code, comments, or formatting
- Do not refactor what is not broken
- Match the existing style even when you would prefer a different one
- If you notice unrelated dead code, mention it — do not delete it

When your change produces orphaned code:

- Remove imports, variables, and functions that your change made obsolete
- Do not remove pre-existing dead code unless asked

**Test:** every changed line should trace directly back to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until they are met.**

Turn imperative instructions into verifiable goals:

| Instead of...       | Convert to...                                            |
| ------------------- | -------------------------------------------------------- |
| "Add validation"    | "Write tests for invalid input, then make them pass"      |
| "Fix the bug"       | "Write a test that reproduces the bug, then make it pass" |
| "Refactor X"        | "Ensure the tests pass both before and after"             |

For multi-step tasks, state a short plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let the LLM iterate independently. Weak ones ("make it work") require constant clarification.

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
