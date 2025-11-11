---
description: Compare ERD DDL and Project DDL to find differences
---

# /compare-ddl

## Purpose
Compare ERD DDL with Project DDL to identify schema differences and synchronization needs.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/compare_ddl_rules.md`

## Execution Steps
1. Read `rule/compare_ddl_rules.md` for detailed comparison rules
2. Parse ERD DDL and Project DDL following the parsing rules
3. Compare tables, columns, PRIMARY KEYs, and options
4. Generate comparison report with severity classification (INFO/WARNING/ERROR)

## Quick Context
- **Input**:
  - `${1}` - ERD DDL file path (optional, default: `docs/erd_ddl/contents.sql`)
  - `${2}` - Project DDL file path (optional, default: `src/main/resources/db/migration/V2__init_api.sql`)
  - `--erd-ref <commit>` - ERD DDL git reference (optional)
  - `--project-ref <commit>` - Project DDL git reference (optional)
- **Output**: Comparison report with schema differences
- **Key Files Read**: ERD DDL, Project DDL

## Examples
```bash
# Compare with default paths
/compare-ddl

# Specify ERD file
/compare-ddl /path/to/erd.sql

# Compare specific git commits
/compare-ddl --erd-ref HEAD~1 --project-ref abc1234
```

## Related Commands
- `/sync-erd-to-project` - Sync ERD changes to Project DDL
- `/sync-ddl-changes` - Apply DDL changes to code