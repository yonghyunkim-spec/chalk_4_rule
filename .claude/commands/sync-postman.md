---
description: Sync Postman collection with Controller changes
---

# /sync-postman

## Purpose
Detect Controller changes and automatically update Postman collection to keep tests in sync with code.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/postman/postman_generation_rules.md`
- `rule/postman/postman_format_spec.md`

## Execution Steps
1. Read required rule files
2. Read existing Postman collection
3. Analyze current Controller state to identify all endpoints
4. Detect changes:
   - New endpoints (in Controller, not in collection)
   - Modified endpoints (URL, method, body changed)
   - Deleted endpoints (in collection, not in Controller)
5. Regenerate collection using `/generate-postman` logic
6. Overwrite existing collection file

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Package names (space-separated, one or more)
- **Output**: Updated Postman collection files
- **Key Files Modified**:
  - postman/{package}_APIs.postman_collection.json

## Update Strategy
**Complete regeneration** (current implementation):
- Use same logic as `/generate-postman`
- Regenerate entire collection
- Ensures format consistency
- ⚠️ Warning: Custom settings (saved responses, custom variables) will be lost

**Alternative: Partial update** (future enhancement):
- Parse JSON and modify only item array
- Preserve existing auth and variable settings
- Preserve user customizations

## Examples
```bash
# Sync single package
/sync-postman board

# Sync multiple packages
/sync-postman board lecture question

# Chain with API doc sync
/sync-api-doc board && /sync-postman board
```

## Related Commands
- `/generate-postman` - Generate initial collection
- `/sync-api-doc` - Sync API documentation before this command