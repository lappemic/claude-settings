# Security Audit

Scan codebase for security vulnerabilities and best practice violations.

**Usage**: `/sec` or `/sec <area>` where area is: `auth`, `api`, `data`, `deps`

## Instructions

### Full Audit (`/sec`)

Run all security checks and generate comprehensive report.

### Authentication Audit (`/sec auth`)

1. **Check authentication patterns**:
   - Verify auth middleware on protected routes
   - Check for proper session handling
   - Validate token storage (not in localStorage for sensitive apps)

2. **Supabase auth checks**:
   ```typescript
   // Every protected route should have:
   const { data: { user } } = await supabase.auth.getUser()
   if (!user) redirect('/login')
   ```

3. **Common issues**:
   - [ ] Missing auth checks on API routes
   - [ ] Client-side only auth (no server validation)
   - [ ] Exposed user IDs in URLs without authorization

### API Security (`/sec api`)

1. **Input validation**:
   ```typescript
   // All inputs should be validated
   const schema = z.object({
     email: z.string().email(),
     name: z.string().min(1).max(100),
   })
   const validated = schema.parse(body)
   ```

2. **Rate limiting check**:
   - Verify rate limiting on auth endpoints
   - Check for abuse protection on public APIs

3. **Headers check**:
   ```typescript
   // Security headers should be set
   {
     'X-Content-Type-Options': 'nosniff',
     'X-Frame-Options': 'DENY',
     'Content-Security-Policy': '...'
   }
   ```

### Data Security (`/sec data`)

1. **RLS policy audit**:
   ```bash
   # List all tables and their policies
   supabase db diff
   ```

2. **Check each table**:
   - [ ] RLS enabled
   - [ ] SELECT policy (who can read)
   - [ ] INSERT policy (who can create)
   - [ ] UPDATE policy (who can modify)
   - [ ] DELETE policy (who can remove)

3. **Sensitive data handling**:
   - [ ] No PII in logs
   - [ ] Passwords properly hashed (Supabase handles this)
   - [ ] API keys in environment variables only

### Dependency Audit (`/sec deps`)

1. **Run npm audit**:
   ```bash
   npm audit
   npm audit fix
   ```

2. **Check for outdated packages**:
   ```bash
   npm outdated
   ```

3. **Review high-risk dependencies**:
   - Authentication libraries
   - Crypto libraries
   - Database drivers

## Security Checklist

### Authentication
- [ ] Server-side auth validation on all protected routes
- [ ] Secure session management
- [ ] Password requirements enforced
- [ ] Rate limiting on login/signup

### Authorization
- [ ] RLS policies on all tables
- [ ] Resource ownership verified before access
- [ ] Admin routes properly protected

### Data Protection
- [ ] Input validation with Zod
- [ ] SQL injection prevented (Supabase parameterized)
- [ ] XSS prevention (React handles most)
- [ ] CSRF protection

### Infrastructure
- [ ] HTTPS only
- [ ] Security headers configured
- [ ] Secrets in environment variables
- [ ] .env files in .gitignore

## Output Format

```
## Security Audit Report

### Critical Vulnerabilities
- [CVE/Issue]: [Description] - [Remediation]

### Warnings
- [Issue]: [Risk Level] - [Recommendation]

### Passed Checks
- [List of passing security measures]

### Action Items
1. [Priority fix 1]
2. [Priority fix 2]
```
