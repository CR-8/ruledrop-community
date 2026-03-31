Perform structured, actionable code reviews. Use this skill when asked to review code, audit a PR, check for bugs, assess code quality, or give feedback on an implementation.

## Review Framework

Work through these layers in order. Don't jump to style nits before checking correctness.

### 1. Correctness
Does the code do what it's supposed to do?
- Logic errors, off-by-one bugs, wrong conditions
- Edge cases: empty input, null/undefined, zero, negative numbers, very large values
- Concurrency issues: race conditions, missing awaits, unhandled promise rejections
- Data integrity: mutations where immutability is expected, shared state bugs

### 2. Security
Could this code be exploited?
- Injection: SQL, command, HTML/XSS — any user input reaching a sink without sanitization
- Authentication/authorization: missing checks, insecure defaults, exposed secrets
- Sensitive data: passwords or tokens in logs, responses, or error messages
- Dependencies: known vulnerable packages (flag, don't enumerate CVEs)

### 3. Performance
Will this scale?
- N+1 queries — loops that trigger database or network calls
- Missing indexes on frequently queried fields
- Unnecessary re-renders or recomputations in UI code
- Large payloads being loaded into memory at once
- Blocking operations on the main thread

### 4. Maintainability
Will the next developer understand this?
- Functions doing more than one thing (split them)
- Names that don't reflect what the thing actually does
- Magic numbers and strings without named constants
- Deep nesting that can be flattened with early returns
- Missing or misleading comments on non-obvious logic

### 5. Test Coverage
Is the important behavior tested?
- Happy path covered
- Error paths and edge cases tested
- Tests that assert behavior, not implementation details
- Flaky tests (time-dependent, order-dependent, network-dependent)

## Output Format

Structure your review as:

**Summary** — one paragraph: overall quality, biggest concern, general recommendation.

**Issues** — grouped by severity:
- 🔴 **Critical** — bugs, security holes, data loss risk. Must fix before shipping.
- 🟡 **Warning** — performance problems, missing error handling, unclear logic. Should fix.
- 🔵 **Suggestion** — style, naming, minor improvements. Nice to have.

For each issue:
- File and line reference (if provided)
- What the problem is and why it matters
- A concrete fix or alternative

**Positives** — call out what's done well. Good reviews aren't just a list of problems.

## Tone

Be direct and specific. Vague feedback ("this could be better") wastes everyone's time. Say exactly what's wrong and how to fix it. Be respectful — critique the code, not the author. If something is genuinely good, say so.

## Language-Specific Notes

**JavaScript/TypeScript**
- Prefer `const` over `let`; avoid `var`
- Use optional chaining (`?.`) and nullish coalescing (`??`) over manual null checks
- Async functions should have try/catch or `.catch()` — unhandled rejections crash Node
- TypeScript: avoid `any`; use `unknown` + type narrowing instead

**Python**
- Use type hints on function signatures
- Prefer list/dict comprehensions over imperative loops for simple transforms
- Context managers (`with`) for file and connection handling
- Raise specific exceptions, not bare `Exception`

**SQL**
- Parameterized queries only — never string concatenation with user input
- Explicit column lists in SELECT — avoid `SELECT *` in production queries
- Check for missing indexes on JOIN and WHERE columns
