---
description: Generate Postman collection from Controller classes
---

# /generate-postman

## Purpose
Generate Postman v2.1 collection from Controller classes for API testing.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/postman/postman_generation_rules.md`
- `rule/postman/postman_format_spec.md`
- `rule/postman/postman_examples.md`

## Execution Steps
1. Read all Postman rule files
2. Analyze Controller classes to extract endpoints
3. Generate collection following Postman v2.1 spec
4. Include all CRUD operations with example request bodies
5. Set up environment variables and auth configuration

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Package names (space-separated, one or more)
- **Output**: Postman collection JSON files
- **Key Files Generated**:
  - {package}_APIs.postman_collection.json (per package)

## Collection Structure
Per Controller → Requests:
- {resource} info (GET single)
- {resource} list (GET list)
- {resource} list count (GET count)
- {resource} add (POST)
- {resource} modify (PUT)
- {resource} delete (DELETE)

## Environment Variables
- {{admin local}} - http://localhost:8080
- {{jwt token}} - Bearer token for authentication

## Related Commands
- `/generate-domain` - Run before to create Controller
- `/sync-postman` - Update existing collection after Controller changes
- `/generate-api-doc` - Generate documentation from same Controllers