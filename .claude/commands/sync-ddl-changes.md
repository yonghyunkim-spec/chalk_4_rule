---
description: Sync DDL changes to code (Entity, VO, Param, Repository)
---

# /sync-ddl-changes

## Purpose
Detect DDL (migration SQL) changes and automatically update related code: Entity, VO, Param, and Repository.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/domain_generation_rules.md`
- `rule/compare_ddl_rules.md`

## Execution Steps
1. Read required rule files
2. Detect DDL changes (git diff or specified file):
   - ADD COLUMN
   - DROP COLUMN
   - MODIFY COLUMN
   - CHANGE COLUMN
3. Analyze change types and impacts
4. Update affected files:
   - Entity fields (auto-update)
   - VO fields (auto-update)
   - Param fields (review required)
5. Report manual review items:
   - QueryRepository queries
   - Service logic
   - Controller validation

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Table names (space-separated, one or more)
  - Last argument ending with `.sql` - DDL file path (optional)
- **Output**: Updated Entity, VO, Param files
- **Key Files Modified**:
  - Entity.java, VO.java, Param.java

## Change Types
- **ADD COLUMN**: Add field to Entity, VO, Param (if required)
- **DROP COLUMN**: Remove field from Entity, VO, Param, update queries
- **MODIFY COLUMN**: Update field type/nullable in Entity

## Complexity Warning
⚠️ **High complexity** - Requires:
- ALTER TABLE parsing
- Code comparison
- Dependency impact analysis

**Recommendation**: Complete Phase 1 commands first, then use this for validation.

## Examples
```bash
# Sync single table
/sync-ddl-changes chapter

# Sync multiple tables
/sync-ddl-changes chapter board

# Specify DDL file
/sync-ddl-changes chapter V5__alter_chapter.sql
```

## Related Commands
- `/compare-ddl` - Compare DDL before syncing
- `/generate-domain` - Generate complete domain from scratch
- `/sync-api-doc` - Update API docs after code changes