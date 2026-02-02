# Semantic Conventions

## Commit Message Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(auth): add password reset flow` |
| `fix` | Bug fix | `fix(chat): prevent duplicate messages` |
| `refactor` | Code change that neither fixes nor adds | `refactor(api): extract validation logic` |
| `style` | Formatting, whitespace, no code change | `style(components): fix indentation` |
| `docs` | Documentation only | `docs(readme): update setup instructions` |
| `test` | Adding or updating tests | `test(auth): add login edge cases` |
| `chore` | Maintenance, dependencies, config | `chore(deps): upgrade next to 14.1` |
| `perf` | Performance improvement | `perf(queries): add database indexes` |

### Scope Derivation

Derive scope from the primary directory/module affected:

- `features/chat/*` → `chat`
- `app/api/auth/*` → `auth`
- `components/ui/*` → `ui`
- `lib/supabase/*` → `supabase`
- Multiple areas → use most significant or omit scope

### Subject Guidelines

- Use imperative mood: "add" not "added" or "adds"
- Don't capitalize first letter
- No period at end
- Max 50 characters
- Describe what, not how

## Branch Naming

```
<type>/<short-description>
```

### Examples

| Change | Branch Name |
|--------|-------------|
| Add user avatars | `feat/user-avatars` |
| Fix login redirect | `fix/login-redirect` |
| Refactor auth helpers | `refactor/auth-helpers` |
| Update dependencies | `chore/update-deps` |

### Guidelines

- Use lowercase and hyphens
- Keep under 50 characters
- Be descriptive but concise
- Match the primary commit type
