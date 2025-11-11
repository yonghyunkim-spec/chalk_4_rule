---
description: Sync ERD DDL to Project DDL format with auto-conversion
---

# /sync-erd-to-project

## Purpose
Sync ERD DDL to Project DDL format with automatic format conversion and table additions.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/compare_ddl_rules.md`

## Execution Steps
1. Read `rule/compare_ddl_rules.md` for DDL comparison and format conversion rules
2. Compare ERD DDL with Project DDL to identify new/changed tables
3. Convert ERD format to Project format:
   - Add PRIMARY KEY clause inside CREATE TABLE
   - Add table options (ENGINE, CHARSET, COLLATE, COMMENT)
   - Normalize spacing and indentation
   - Convert data types to uppercase
4. Add or update tables in Project DDL
5. Verify synchronization completeness

## Quick Context
- **Input**:
  - `${1}` - ERD DDL file path (optional, default: docs/erd_ddl/contents.sql)
  - `${2}` - Project DDL file path (optional, default: src/main/resources/db/migration/V2__init_api.sql)
- **Output**: Project DDL updated with ERD changes
- **Key Files Modified**: Project DDL file (V2__init_api.sql)

## Format Conversion
**ERD format** → **Project format**:
- Add `PRIMARY KEY (id)` inside CREATE TABLE
- Add `ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci`
- Convert `int` → `INT`, `varchar` → `VARCHAR`
- Add table COMMENT
- Normalize indentation and spacing

## Examples
```bash
# Use default paths
/sync-erd-to-project

# Specify ERD file only
/sync-erd-to-project /path/to/erd.sql

# Specify both files
/sync-erd-to-project /path/to/erd.sql V3__update.sql
```

## Related Commands
- `/compare-ddl` - Compare DDL before syncing
- `/sync-ddl-changes` - Sync DDL changes to code after this command