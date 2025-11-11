---
description: Safely remove a domain and restore project to previous state
---

# /remove-domain

## Purpose
Safely remove a generated domain and restore project to previous state, including DDL, code, tests, docs, and Postman collection.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/compare_ddl_rules.md` (for DDL removal)
- `rule/domain_generation_rules.md` (for understanding domain structure)

## Execution Steps
1. Read required rule files to understand domain structure
2. **Phase 6 → 1** (reverse order): Remove domain components
   - Phase 6: Clean up sync changes in DDL (field modifications)
   - Phase 5: Delete Postman collection file
   - Phase 4: Delete API documentation folder
   - Phase 3: Delete test code package
   - Phase 2: Delete domain package
   - Phase 1: Remove table from DDL
3. Clean up configuration files (QueryDslTestConfig, etc.)
4. Verify removal completeness

## Quick Context
- **Input**:
  - `${1}` - Domain name (single domain, e.g., test, board, lecture)
- **Output**: Domain completely removed from project
- **Key Files Deleted**:
  - DDL: Table definition removed from V2__init_api.sql
  - Code: src/main/java/com/firsthabit/chalk/{domain}/
  - Tests: src/test/java/com/firsthabit/chalk/{domain}/
  - Docs: docs/api_docs/{domain}/
  - Postman: postman/{domain}_APIs.postman_collection.json

## Removal Strategy
**Practical approach**: Delete packages/files entirely (Phase 5-2), then clean DDL (Phase 1).
- No need to reverse individual field additions from sync operations
- Reason: Entire packages are deleted, so individual field removal is unnecessary

## Safety Notes
- ⚠️ **Destructive operation** - ensure backup before removal
- ⚠️ **No undo** - files are permanently deleted
- ✅ **Verification** - confirms complete removal after execution

## Related Commands
- `/create-domain` - Create domain (opposite operation)
- `/compare-ddl` - Verify DDL state after removal