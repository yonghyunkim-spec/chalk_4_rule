---
description: Generate API documentation from Controller classes
---

# /generate-api-doc

## Purpose
Generate comprehensive API documentation from Controller classes automatically.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/api_documentation/api_doc_generation_rules.md`
- `rule/api_documentation/api_doc_template.md`
- `rule/api_documentation/api_doc_examples.md`

## Execution Steps
1. Read all API documentation rule files
2. Analyze Controller classes to extract endpoints
3. Generate documentation following the template for each API
4. Include request/response examples with actual enum values
5. Update ADMIN_API.md table of contents

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Package names (space-separated, one or more)
- **Output**: API documentation markdown files
- **Key Files Generated**:
  - docs/api_docs/{package}/{domain}/{domain}_{action}.md (per API)
  - docs/api_docs/ADMIN_API.md (table of contents)

## Document Structure
Per Controller → Multiple API files:
- {domain}_info.md - Single entity retrieval
- {domain}_list.md - List retrieval
- {domain}_list_count.md - Count
- {domain}_add.md - Create
- {domain}_mod.md - Update
- {domain}_del.md - Delete

## Related Commands
- `/generate-domain` - Run before to create Controller
- `/sync-api-doc` - Update existing API docs after Controller changes
- `/generate-postman` - Generate Postman collection from same Controllers