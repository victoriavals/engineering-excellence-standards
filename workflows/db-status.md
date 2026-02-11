---
description: Check database connection and health status
---

# Database Status Workflow

**Purpose:** Verify database connection, check health, dan troubleshoot connection issues for ANY database system.

**Use Cases:**
- Debug database connection problems
- Verify database setup (PostgreSQL, MySQL, MongoDB, Supabase, Firebase, etc.)
- Monitor database health
- Check data integrity

---

## Status Check Steps

### 1. Detect Database Type

Ask user: "Apa database yang kamu pakai?"

Options:
- **PostgreSQL** (includes Supabase)
- **MySQL/MariaDB**
- **MongoDB**
- **SQLite**
- **Firebase**
- **Redis**
- **Other** (user specifies)

---

### 2. Load Connection Configuration

**Check environment variables or config files:**

```bash
# Check .env file
if (Test-Path ".env") {
    Get-Content .env | Select-String "DATABASE|DB_|MONGO|REDIS|FIREBASE"
} else {
    Write-Host "⚠️ No .env file found"
}
```

**Common variables to check:**
- `DATABASE_URL`
- `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_NAME`
- `MONGODB_URI`
- `REDIS_URL`
- `FIREBASE_CONFIG`

---

### 3. Test Connection

Based on database type:

**PostgreSQL (via psql):**
```bash
// turbo
psql $env:DATABASE_URL -c "SELECT version();"
```

**MySQL:**
```bash
mysql -h $env:DB_HOST -u $env:DB_USER -p$env:DB_PASS -e "SELECT VERSION();"
```

**MongoDB:**
```bash
mongosh $env:MONGODB_URI --eval "db.runCommand({ connectionStatus: 1 })"
```

**SQLite:**
```bash
sqlite3 database.db "SELECT sqlite_version();"
```

**Generic (HTTP API like Supabase/Firebase):**
```powershell
$response = Invoke-WebRequest -Uri "$env:DATABASE_URL" -Headers @{
    "Authorization" = "Bearer $env:DATABASE_KEY"
}
Write-Host "Status: $($response.StatusCode)"
```

---

### 4. Check Table/Collection Count

```bash
# PostgreSQL/MySQL
[database command] -c "SELECT COUNT(*) FROM information_schema.tables;"

# MongoDB
mongosh --eval "db.getCollectionNames().length"

# SQLite
sqlite3 database.db ".tables" | Measure-Object -Line
```

---

### 5. Get Record Statistics

Ask: "Mau check record count table mana?" (atau check all tables)

**PostgreSQL:**
```sql
SELECT schemaname, tablename, n_tup_ins as inserts, n_tup_upd as updates
FROM pg_stat_user_tables
ORDER BY n_tup_ins DESC;
```

**MySQL:**
```sql
SELECT TABLE_NAME, TABLE_ROWS 
FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'your_database';
```

**MongoDB:**
```javascript
db.stats()
db.collection_name.count()
```

---

### 6. Test Write Permission (Optional)

Ask: "Mau test write permission (insert test record)?"

**Generic Approach:**
```
1. Create test table/collection
2. Insert test record
3. Read it back
4. Delete test record
5. Drop test table/collection
```

Adapt command based on database type.

---

### 7. Generate Health Report

```markdown
# Database Health Report

## Connection Status
✅ Database connection successful
✅ Authentication working
✅ Database type: [PostgreSQL/MySQL/MongoDB/etc.]

## Database Info
- Version: [version]
- Host: [host:port]
- Database name: [name]

## Statistics
- Total tables/collections: [N]
- Total records (approx): [N]
- Database size: [XX]MB
- Latest activity: [timestamp]

## Permissions
✅ Read permission: OK
✅ Write permission: OK (if tested)
✅ Connection pool: [active/max connections]

## Overall Health
🟢 HEALTHY - All checks passed

---

Next Steps:
1. Monitor connection pool usage
2. Set up automated health checks
3. Review slow queries (if applicable)
```

---

## Troubleshooting Tips

**Connection Failed:**
1. Check `.env` has correct credentials
2. Verify host/port accessible (telnet/nc)
3. Check firewall rules
4. Verify database server is running
5. Test from database client tool first

**Auth Failed:**
1. Verify username/password correct
2. Check user has proper permissions
3. Verify SSL/TLS requirements
4. Check IP whitelist (cloud databases)

**Timeout:**
1. Check network latency
2. Increase connection timeout
3. Check database server load
4. Verify connection pool settings

---

## Database-Specific Commands Reference

**PostgreSQL:**
- Connect: `psql $DATABASE_URL`
- List tables: `\dt`
- Table size: `SELECT pg_size_pretty(pg_total_relation_size('table_name'));`

**MySQL:**
- Connect: `mysql -h HOST -u USER -p`
- List tables: `SHOW TABLES;`
- Database size: `SELECT table_schema, SUM(data_length + index_length) / 1024 / 1024 AS "Size (MB)" FROM information_schema.TABLES GROUP BY table_schema;`

**MongoDB:**
- Connect: `mongosh URI`
- Collections: `show collections`
- Stats: `db.stats()`

**This workflow helps diagnose ANY database connection issues!**
