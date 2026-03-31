Apply Microsoft's engineering standards for code quality, TypeScript, .NET, and developer experience. Use this skill when writing TypeScript/C# that should follow Microsoft's conventions, building applications with Microsoft's engineering principles, or applying Microsoft's approach to API design, accessibility, and open-source contribution.

## Core Philosophy

Microsoft's engineering culture has evolved significantly — from closed, Windows-centric to one of the world's largest open-source contributors. Key principles:
- **TypeScript-first** — Microsoft created TypeScript; use it fully, not as "JavaScript with types"
- **Accessibility is non-negotiable** — Microsoft's accessibility standards are among the industry's highest
- **Backward compatibility** — don't break existing users; deprecate gracefully
- **Documentation as code** — docs are part of the product, not an afterthought

## TypeScript (Microsoft's Language)

### Use TypeScript strictly
```json
// tsconfig.json — Microsoft's recommended strict settings
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true
  }
}
```

### Type design
```ts
// Prefer interfaces for object shapes (extensible via declaration merging)
interface User {
  id: string
  name: string
  email: string
  role: UserRole
}

// Use type aliases for unions, intersections, and computed types
type UserRole = 'admin' | 'editor' | 'viewer'
type PartialUser = Partial<User>
type UserWithTimestamps = User & { createdAt: Date; updatedAt: Date }

// Use readonly for immutable data
interface Config {
  readonly apiUrl: string
  readonly timeout: number
}

// Discriminated unions for state machines
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

### Avoid common TypeScript anti-patterns
```ts
// NEVER use `any` — use `unknown` and narrow
function processInput(input: unknown) {
  if (typeof input === 'string') {
    return input.toUpperCase()
  }
  throw new TypeError('Expected string')
}

// AVOID type assertions — they bypass the type system
// BAD:  const user = data as User
// GOOD: use a type guard
function isUser(data: unknown): data is User {
  return typeof data === 'object' && data !== null && 'id' in data && 'name' in data
}

// AVOID non-null assertions (!) — handle null explicitly
// BAD:  const name = user!.name
// GOOD: const name = user?.name ?? 'Unknown'
```

## C# / .NET Conventions

```csharp
// Naming: PascalCase for everything public, camelCase for private fields
public class UserService
{
    private readonly IUserRepository _repository;  // private: _camelCase

    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }

    // Async methods: always suffix with Async
    public async Task<User?> GetUserAsync(string userId, CancellationToken ct = default)
    {
        ArgumentException.ThrowIfNullOrEmpty(userId);
        return await _repository.FindByIdAsync(userId, ct);
    }
}

// Use records for immutable data transfer objects
public record UserDto(string Id, string Name, string Email);

// Use pattern matching
string Describe(object obj) => obj switch
{
    User { Role: "admin" } u => $"Admin: {u.Name}",
    User u => $"User: {u.Name}",
    null => "null",
    _ => obj.ToString() ?? "unknown"
};

// Nullable reference types — enable in all new projects
#nullable enable
```

## API Design (Microsoft REST API Guidelines)

Microsoft publishes detailed REST API guidelines. Key rules:

```
# URL structure
GET    /api/v1/users              → list
GET    /api/v1/users/{userId}     → get one
POST   /api/v1/users              → create
PATCH  /api/v1/users/{userId}     → partial update
DELETE /api/v1/users/{userId}     → delete

# Versioning: always in the URL path
/api/v1/...
/api/v2/...

# Response envelope
{
  "value": [...],           // collection
  "@nextLink": "...",       // pagination cursor
  "@count": 100             // total count (optional)
}

# Error response (OData-compatible)
{
  "error": {
    "code": "UserNotFound",
    "message": "User with ID '123' was not found.",
    "innererror": {
      "code": "DatabaseError",
      "message": "..."
    }
  }
}
```

## Accessibility (Microsoft's WCAG Commitment)

Microsoft's accessibility bar is high — they build products for users with disabilities at scale.

```tsx
// Use ARIA roles and properties correctly
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</div>

// Live regions for dynamic content
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

// Focus management
useEffect(() => {
  if (isOpen) {
    firstFocusableRef.current?.focus()
  }
}, [isOpen])

// Color contrast: minimum 4.5:1 for normal text, 3:1 for large text
// Test with Windows High Contrast mode
// Support keyboard navigation for all interactive elements
// Provide text alternatives for all non-text content
```

## Testing (.NET + TypeScript)

```ts
// Jest/Vitest: descriptive test names using "should" pattern
describe('UserService', () => {
  describe('getUser', () => {
    it('should return user when valid ID is provided', async () => { ... })
    it('should throw NotFoundError when user does not exist', async () => { ... })
    it('should throw ValidationError when ID is empty', async () => { ... })
  })
})
```

```csharp
// xUnit: Fact for single cases, Theory for parameterized
[Fact]
public async Task GetUserAsync_WithValidId_ReturnsUser()
{
    // Arrange
    var userId = "user-123";
    _mockRepo.Setup(r => r.FindByIdAsync(userId, default)).ReturnsAsync(new User { Id = userId });

    // Act
    var result = await _service.GetUserAsync(userId);

    // Assert
    Assert.NotNull(result);
    Assert.Equal(userId, result.Id);
}

[Theory]
[InlineData("")]
[InlineData(null)]
public async Task GetUserAsync_WithInvalidId_ThrowsArgumentException(string? userId)
{
    await Assert.ThrowsAsync<ArgumentException>(() => _service.GetUserAsync(userId!));
}
```

## Open Source Contribution Standards

Microsoft is a major OSS contributor. Their contribution standards:
- **CHANGELOG.md** — document every user-facing change
- **CONTRIBUTING.md** — clear guide for external contributors
- **CODE_OF_CONDUCT.md** — Microsoft uses the Contributor Covenant
- **Semantic versioning** — strictly followed
- **Breaking changes** — major version bump, migration guide required
- **Deprecation** — minimum one major version notice before removal
