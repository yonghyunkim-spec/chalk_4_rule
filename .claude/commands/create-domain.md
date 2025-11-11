---
description: Create complete domain from ERD DDL to code, tests, docs, and Postman (Phase 1-5 workflow)
---

# /create-domain

## Purpose
Create complete domain package from ERD DDL through 5-phase workflow: DDL sync → Code generation → Tests → API docs → Postman collection.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/compare_ddl_rules.md` (Phase 1: DDL sync)
- `rule/domain_generation_rules.md` (Phase 2: Code generation)
- `rule/repository_test_rules.md` (Phase 3: Tests)
- `rule/api_documentation/api_doc_generation_rules.md` (Phase 4: API docs)
- `rule/postman/postman_generation_rules.md` (Phase 5: Postman)

## Execution Steps
1. Read all required rule files for 5-phase workflow
2. **Phase 1**: Sync ERD DDL to Project DDL (add table to V2__init_api.sql)
3. **Phase 2**: Generate domain package (Entity, VO, Param, Repository, Service, Controller)
4. **Phase 3**: Generate test code (RepositoryTest, QueryRepositoryTest)
5. **Phase 4**: Generate API documentation (6 API docs + ADMIN_API.md update)
6. **Phase 5**: Generate Postman collection (6 requests: info, list, count, add, mod, del)

## Quick Context
- **Input**:
  - `${1}` - Domain name (required, e.g., test, board, chapter)
  - `--full` - Execute all phases 1-5 (default)
  - `--phase N` - Execute specific phase only
  - `--skip-phase N` - Skip specific phase
- **Output**: Complete domain package (~25 files)
- **Estimated Time**: 30 minutes (vs 10.5 hours manual work, 95% time saved)

## 5-Phase Workflow
1. **Phase 1**: DDL Preparation → `/compare-ddl` + `/sync-erd-to-project`
2. **Phase 2**: Domain Package → `/generate-domain`
3. **Phase 3**: Test Code → `/generate-repo-test`
4. **Phase 4**: API Documentation → `/generate-api-doc`
5. **Phase 5**: Postman Collection → `/generate-postman`

## Examples
```bash
# Execute all phases 1-5
/create-domain test

# Skip Phase 1 (DDL already prepared)
/create-domain test --skip-phase 1

# Execute Phase 2 only (domain package only)
/create-domain test --phase 2
```

## Related Commands
- `/compare-ddl` - Compare DDL before starting
- `/remove-domain` - Remove domain if needed to rollback
