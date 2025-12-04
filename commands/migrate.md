# Database Migration Helper

Create and manage Supabase database migrations safely.

**Usage**: `/migrate <action>` where action is: `create`, `check`, `push`, `rollback`

## Instructions

### `/migrate create <name>`

1. **Generate migration file**:
   ```bash
   # Filename format: YYYYMMDDHHMMSS_<name>.sql
   TIMESTAMP=$(date +%Y%m%d%H%M%S)
   FILENAME="supabase/migrations/${TIMESTAMP}_$ARGUMENTS.sql"
   ```

2. **Gather context**:
   - List existing tables: Check `supabase/migrations/` for schema
   - Ask user what changes they need:
     - New table?
     - Add/modify columns?
     - Add indexes?
     - Update RLS policies?

3. **Generate migration SQL**:
   ```sql
   -- Migration: <name>
   -- Created: <timestamp>

   -- Up migration
   <SQL statements>

   -- Note: Supabase migrations are forward-only
   -- Document rollback steps in comments if needed
   ```

4. **Add RLS policies** (always for new tables):
   ```sql
   -- Enable RLS
   ALTER TABLE <table> ENABLE ROW LEVEL SECURITY;

   -- Policies
   CREATE POLICY "Users can view own data"
     ON <table> FOR SELECT
     USING (auth.uid() = user_id);
   ```

5. **Generate TypeScript types**:
   ```bash
   npx supabase gen types typescript --local > src/types/database.ts
   ```

### `/migrate check`

1. Check local vs remote status:
   ```bash
   supabase db diff
   ```

2. List pending migrations:
   ```bash
   supabase migration list
   ```

### `/migrate push`

1. **Pre-push checklist**:
   - [ ] Review migration SQL
   - [ ] Backup reminder for production
   - [ ] Test locally first

2. **Push to remote**:
   ```bash
   supabase db push
   ```

3. **Regenerate types**:
   ```bash
   npx supabase gen types typescript --project-id <id> > src/types/database.ts
   ```

### `/migrate rollback`

1. **Warning**: Supabase doesn't support automatic rollback
2. Generate reverse migration if possible
3. Document manual steps needed

## Common Patterns

### Add column with default
```sql
ALTER TABLE users ADD COLUMN status text DEFAULT 'active';
```

### Create junction table
```sql
CREATE TABLE user_roles (
  user_id uuid REFERENCES users(id) ON DELETE CASCADE,
  role_id uuid REFERENCES roles(id) ON DELETE CASCADE,
  PRIMARY KEY (user_id, role_id)
);
```

### Add index for performance
```sql
CREATE INDEX idx_messages_user_id ON messages(user_id);
CREATE INDEX idx_messages_created_at ON messages(created_at DESC);
```
