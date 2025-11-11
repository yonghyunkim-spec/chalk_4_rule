---
description: Generate complete domain package (Entity, VO, Param, Repository, Service, Controller) from DDL
---

# /generate-domain

## Purpose
Generate complete domain package with full CRUD functionality from DDL (CREATE TABLE statement).

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/domain_generation_rules.md`
- `docs/gen-guide/03_DOMAIN_GENERATION.md` (reference)

## Execution Steps
1. Read `rule/domain_generation_rules.md` for complete generation rules
2. Follow the 11-Step Domain Generation Process defined in the rule file
3. Generate 7 files: Entity, VO, Param, Repository, Service, Controller, RepositoryTest
4. Verify outputs at each checkpoint (Entity validation, Repository test, API test)

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Table names (space-separated, one or more)
  - Last argument ending with `.sql` - DDL file path (optional)
- **Output**: Complete domain package (7 files)
- **Key Files Generated**:
  - Entity (JPA + Auditing)
  - VO (Response DTO)
  - Param (Request DTO)
  - Repository (JPA + QueryDSL)
  - Service (Business Logic)
  - Controller (REST API)
  - RepositoryTest (JUnit + QueryDSL tests)

## Tech Stack Requirements
- ✅ QueryDSL for complex queries
- ✅ MapStruct for all mappings
- ✅ Enum for all constants
- ✅ VO return (never Entity)
- ✅ PagingParam inheritance for SearchParam
- ❌ No @Query annotation (use JPA naming convention)
- ❌ No manual mapping (use MapStruct)

## Related Commands
- `/compare-ddl` - Run before to ensure DDL sync
- `/generate-repo-test` - Run after to create additional tests
- `/generate-api-doc` - Generate API documentation
- `/generate-postman` - Generate Postman collection