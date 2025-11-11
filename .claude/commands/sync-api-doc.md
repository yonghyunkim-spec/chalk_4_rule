---
description: Sync API documentation with Controller changes
---

# /sync-api-doc

## Purpose
Detect Controller changes and automatically update API documentation to keep docs in sync with code.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/api_documentation/api_doc_generation_rules.md`
- `rule/api_documentation/api_doc_template.md`

## Execution Steps
1. Read required rule files
2. Read existing API documentation
3. Analyze current Controller state to identify all endpoints
4. Detect changes:
   - New endpoints (in Controller, not in docs)
   - Modified endpoints (URL, params, response changed)
   - Deleted endpoints (in docs, not in Controller)
5. Regenerate documentation using `/generate-api-doc` logic
6. Update change history with today's date and summary

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Package names (space-separated, one or more)
- **Output**: Updated API documentation files
- **Key Files Modified**:
  - docs/api_docs/{package}/{domain}/*.md

## Update Strategy
**Complete regeneration** (recommended):
- Use same logic as `/generate-api-doc`
- Regenerate entire documentation
- Add update date to change history
- Ensures format consistency

## Examples
```bash
# Sync single package
/sync-api-doc board

# Sync multiple packages
/sync-api-doc board lecture question
```

## Related Commands
- `/generate-api-doc` - Generate initial documentation
- `/sync-postman` - Sync Postman collection after this command