Apply Google's engineering standards for code style, design, and review. Use this skill when writing code that should follow Google's style guides, conducting code reviews using Google's methodology, or structuring a codebase to Google's engineering standards.

## Core Philosophy

Google's engineering culture is built on three pillars:
- **Readability** — code is read far more than it's written; optimize for the reader
- **Consistency** — a consistent codebase is easier to maintain than a "clever" one
- **Simplicity** — prefer the straightforward solution; complexity must justify itself

> "Make it work, make it right, make it fast — in that order."

## Code Review (Google's CL Review Standard)

Google's code review process is one of the most documented in the industry.

### Reviewer mindset
- Look for correctness, design, complexity, tests, naming, comments, style
- Approve when the CL **improves the overall health of the codebase**, even if imperfect
- Don't demand perfection — demand improvement
- Be kind; critique the code, not the author

### Review speed
- Respond to code reviews within **one business day**
- If you can't do a full review, leave a comment saying when you can
- Fast reviews reduce context-switching cost for the author

### CL size
- Keep CLs small — one logical change per CL
- Large CLs are harder to review and more likely to introduce bugs
- If a CL is too large, ask the author to split it

### Comments
- Prefix with `Nit:` for minor style issues that shouldn't block approval
- Explain *why* something is wrong, not just *that* it's wrong
- If you ask a question, make clear whether it's blocking or informational

## Naming Conventions

### Python (Google Python Style Guide)
```python
# Modules: lowercase_with_underscores
import my_module

# Classes: CapWords
class MyClass:
    pass

# Functions/methods: lowercase_with_underscores
def calculate_total(items):
    pass

# Constants: UPPER_CASE_WITH_UNDERSCORES
MAX_RETRY_COUNT = 3

# Private: single leading underscore
def _internal_helper():
    pass

# "Really private" (name mangling): double leading underscore
def __very_private(self):
    pass
```

### JavaScript/TypeScript (Google JS Style Guide)
```ts
// Variables/functions: camelCase
const userCount = 0
function getUserById(id: string) {}

// Classes/interfaces/types: PascalCase
class UserService {}
interface UserRepository {}
type UserId = string

// Constants: UPPER_SNAKE_CASE for module-level constants
const MAX_CONNECTIONS = 100

// Private class members: use # (native private fields)
class Counter {
  #count = 0
  increment() { this.#count++ }
}

// Enums: PascalCase name, UPPER_SNAKE_CASE members
enum Direction {
  NORTH = 'NORTH',
  SOUTH = 'SOUTH',
}
```

## Code Style Rules

### Functions
- **Do one thing** — a function that does two things should be two functions
- **Short is better** — aim for functions under 40 lines; if longer, consider splitting
- **Avoid side effects** — pure functions are easier to test and reason about
- **Fail fast** — validate inputs at the top, return early on error

```python
# GOOD: early return, single responsibility
def get_user(user_id: str) -> User:
    if not user_id:
        raise ValueError("user_id cannot be empty")
    user = db.find(user_id)
    if user is None:
        raise NotFoundError(f"User {user_id} not found")
    return user
```

### Comments
Google's rule: **comments explain *why*, not *what*.**

```python
# BAD: states the obvious
# Increment counter
counter += 1

# GOOD: explains non-obvious reasoning
# Use exponential backoff to avoid thundering herd on retry
delay = min(base_delay * (2 ** attempt), max_delay)
```

Write comments for:
- Non-obvious algorithms or business logic
- Workarounds for known bugs (with bug tracker reference)
- Public API documentation (docstrings)
- TODO/FIXME with owner and context

### Error Handling
```python
# Be specific — catch the narrowest exception possible
try:
    result = parse_config(path)
except FileNotFoundError:
    logger.error("Config file not found: %s", path)
    raise
except json.JSONDecodeError as e:
    logger.error("Invalid JSON in config: %s", e)
    raise ConfigError(f"Failed to parse config: {e}") from e
```

## Testing (Google's Testing Philosophy)

### The Testing Pyramid
- **Unit tests** — fast, isolated, test one thing. The bulk of your tests.
- **Integration tests** — test component interactions. Fewer than unit tests.
- **E2E tests** — test full user flows. Slow, expensive, use sparingly.

### Test naming
```python
# Pattern: test_<what>_<condition>_<expected>
def test_get_user_with_valid_id_returns_user():
    ...

def test_get_user_with_empty_id_raises_value_error():
    ...

def test_get_user_when_not_found_raises_not_found_error():
    ...
```

### Test structure (AAA)
```python
def test_calculate_discount_for_premium_user():
    # Arrange
    user = User(tier='premium')
    cart = Cart(items=[Item(price=100)])

    # Act
    discount = calculate_discount(user, cart)

    # Assert
    assert discount == 20  # 20% for premium users
```

### What to test
- Every public function/method
- All error paths and edge cases
- Boundary conditions (empty, zero, max values)
- Don't test implementation details — test behavior

## Design Principles

### SOLID (Google engineers apply these rigorously)
- **S** — Single Responsibility: one reason to change
- **O** — Open/Closed: open for extension, closed for modification
- **L** — Liskov Substitution: subtypes must be substitutable for base types
- **I** — Interface Segregation: many specific interfaces over one general one
- **D** — Dependency Inversion: depend on abstractions, not concretions

### Prefer composition over inheritance
```python
# PREFER: composition
class EmailNotifier:
    def send(self, message): ...

class UserService:
    def __init__(self, notifier: EmailNotifier):
        self._notifier = notifier

# AVOID: deep inheritance chains
class UserService(BaseService, NotificationMixin, LoggingMixin): ...
```

## Go-Specific (Google's primary language)

```go
// Error handling: always check errors, wrap with context
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// Naming: short, clear names in narrow scope; longer names in wider scope
// i, j for loop vars; err for errors; ctx for context

// Interfaces: define at the point of use, not the point of implementation
// Keep interfaces small (1-3 methods)

// Goroutines: always have a clear owner and shutdown path
// Use context.Context for cancellation
```
