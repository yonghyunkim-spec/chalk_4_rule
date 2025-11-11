---
description: Generate Repository test classes automatically
---

# /generate-repo-test

## Purpose
Generate Repository test classes with comprehensive QueryDSL test cases automatically.

## Required Rule Files
**Before executing this command, you MUST read:**
- `rule/repository_test_rules.md`
- `rule/domain_generation_rules.md` (for Repository patterns)

## Execution Steps
1. Read required rule files
2. Identify Repository classes to test (JpaRepository and QueryRepository)
3. Analyze Entity and Enum classes for test data
4. Generate test classes following @DataJpaTest patterns
5. Update QueryDslTestConfig if QueryRepository exists
6. Run tests and verify all pass

## Quick Context
- **Input**:
  - `${1} ${2} ${3}` - Package names (space-separated, one or more)
- **Output**: RepositoryTest classes
- **Key Files Generated**:
  - src/test/.../RepositoryTest.java (@DataJpaTest)
- **Key Files Modified**:
  - QueryDslTestConfig.java (if QueryRepository exists)

## Tech Stack Requirements
- ✅ @DataJpaTest annotation
- ✅ TestEntityManager for setup
- ✅ QueryDSL test cases
- ✅ Actual enum values (never hardcoded)
- ✅ All Repository methods tested

## Related Commands
- `/generate-domain` - Run before to create Repository
- `./gradlew test --tests "*RepositoryTest"` - Run generated tests