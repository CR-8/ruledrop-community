Apply security best practices across web applications, APIs, and infrastructure. Use this skill when building authentication systems, handling user data, designing APIs, reviewing code for vulnerabilities, configuring servers, or any time security is a concern.

## OWASP Top 10 — Quick Reference

Address these in every web application:

1. **Broken Access Control** — enforce authorization on every request server-side
2. **Cryptographic Failures** — encrypt sensitive data at rest and in transit
3. **Injection** — parameterize all queries, sanitize all inputs
4. **Insecure Design** — threat model before building, not after
5. **Security Misconfiguration** — harden defaults, disable unused features
6. **Vulnerable Components** — audit and update dependencies regularly
7. **Auth Failures** — implement MFA, secure session management
8. **Integrity Failures** — verify software and data integrity (CI/CD, SRI)
9. **Logging Failures** — log security events, alert on anomalies
10. **SSRF** — validate and restrict outbound requests

## Authentication & Sessions

```js
// Password hashing — always bcrypt, argon2, or scrypt. Never MD5/SHA1.
import bcrypt from 'bcrypt'
const SALT_ROUNDS = 12  // minimum 10, 12 recommended

async function hashPassword(plain) {
  return bcrypt.hash(plain, SALT_ROUNDS)
}
async function verifyPassword(plain, hash) {
  return bcrypt.compare(plain, hash)
}

// JWT — short-lived access tokens + refresh token rotation
const ACCESS_TOKEN_TTL  = '15m'
const REFRESH_TOKEN_TTL = '7d'

// Session cookies — always set these flags
res.cookie('session', token, {
  httpOnly: true,    // no JS access
  secure: true,      // HTTPS only
  sameSite: 'strict', // CSRF protection
  maxAge: 15 * 60 * 1000,
  path: '/',
})
```

**Never:**
- Store passwords in plain text or with reversible encryption
- Put sensitive data in JWTs (they're base64, not encrypted)
- Use long-lived tokens without rotation
- Roll your own crypto

## Injection Prevention

### SQL — always parameterized queries
```js
// GOOD
const user = await db.query('SELECT * FROM users WHERE email = $1', [email])

// NEVER
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`)
```

### Command injection
```js
import { execFile } from 'child_process'

// GOOD — execFile doesn't invoke a shell
execFile('convert', [inputPath, '-resize', '800x600', outputPath], callback)

// NEVER — exec passes through shell, user input can escape
exec(`convert ${userInput} output.jpg`, callback)
```

### XSS prevention
```js
// Sanitize HTML input with DOMPurify (client) or sanitize-html (server)
import DOMPurify from 'dompurify'
const clean = DOMPurify.sanitize(userHtml, { ALLOWED_TAGS: ['b', 'i', 'em', 'strong'] })

// React escapes by default — only dangerouslySetInnerHTML needs sanitization
// Never: <div dangerouslySetInnerHTML={{ __html: userInput }} />
// Always: <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

## Input Validation

Validate on the server. Always. Client-side validation is UX, not security.

```js
import { z } from 'zod'

const CreateUserSchema = z.object({
  email:    z.string().email().max(254),
  password: z.string().min(12).max(128),
  name:     z.string().min(1).max(100).regex(/^[\w\s'-]+$/),
  age:      z.number().int().min(13).max(120).optional(),
})

// In route handler
const result = CreateUserSchema.safeParse(req.body)
if (!result.success) {
  return res.status(422).json({ error: result.error.flatten() })
}
```

## HTTPS & Transport Security

```
# HTTP response headers — set on every response
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'; ...
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Use [helmet](https://helmetjs.github.io/) in Express to set these automatically:
```js
import helmet from 'helmet'
app.use(helmet())
```

## CORS

```js
const corsOptions = {
  origin: (origin, callback) => {
    const allowed = ['https://app.example.com', 'https://www.example.com']
    if (!origin || allowed.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error('Not allowed by CORS'))
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}
app.use(cors(corsOptions))
```

Never use `origin: '*'` with `credentials: true`.

## Rate Limiting

```js
import rateLimit from 'express-rate-limit'

// General API
app.use('/api/', rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }))

// Auth endpoints — stricter
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
  message: { error: 'Too many attempts, try again later' }
}))
```

## Secrets Management

```bash
# Never commit secrets. Use environment variables.
DATABASE_URL=postgres://...
JWT_SECRET=...
API_KEY=...

# Use a secrets manager in production
# AWS Secrets Manager, HashiCorp Vault, Doppler, etc.
```

```js
// Validate required env vars at startup — fail fast
const required = ['DATABASE_URL', 'JWT_SECRET', 'REDIS_URL']
for (const key of required) {
  if (!process.env[key]) throw new Error(`Missing required env var: ${key}`)
}
```

## File Upload Security

```js
// Validate MIME type server-side (not just extension)
import fileType from 'file-type'

async function validateUpload(buffer) {
  const type = await fileType.fromBuffer(buffer)
  const allowed = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf']
  if (!type || !allowed.includes(type.mime)) {
    throw new Error('Invalid file type')
  }
  if (buffer.length > 10 * 1024 * 1024) {  // 10MB limit
    throw new Error('File too large')
  }
}

// Never serve uploaded files from the same origin as the app
// Use a separate domain (e.g., uploads.example.com) or CDN
```

## Dependency Security

```bash
# Audit regularly
npm audit
npm audit fix

# Pin exact versions in production
npm install --save-exact package-name

# Use Dependabot or Renovate for automated updates
```

## Security Logging

Log these events (never log passwords, tokens, or PII):
- Failed login attempts (with IP, timestamp)
- Successful logins (with IP, user agent)
- Permission denied errors
- Input validation failures on sensitive endpoints
- Rate limit triggers
- Admin actions

```js
// Structured security log
logger.warn('auth.failed', {
  event: 'login_failed',
  email: maskEmail(email),  // mask PII
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  timestamp: new Date().toISOString(),
})
```
