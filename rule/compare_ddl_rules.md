# DDL 비교 규칙

## 📋 개요

ERD 툴의 DDL과 프로젝트 DDL을 비교하여 차이점을 분석하고 보고하는 규칙을 정의합니다.

## 🎯 적용 대상

- `/compare-ddl` 슬래시 커맨드

## 📁 파일 경로 규칙

### ERD DDL 파일
```
기본값: /Users/kyle/source/00.document/erd_ddl/contents.sql
사용자 지정: ${1} 인자로 제공 가능
```

### Project DDL 파일
```
기본값: src/main/resources/db/migration/V2__init_api.sql
사용자 지정: ${2} 인자로 제공 가능
```

## 🔄 비교 프로세스

### 1. DDL 파일 읽기

**Git 참조 지원**:
- `--erd-ref <commit>`: ERD DDL의 특정 커밋 비교
- `--project-ref <commit>`: Project DDL의 특정 커밋 비교

**읽기 방법**:
```bash
# Git 참조가 있으면
git show ${ref}:${file_path}

# Git 참조가 없으면
Read tool로 현재 파일 읽기
```

### 2. CREATE TABLE 파싱

각 테이블별 정보 추출:
- 테이블명
- 컬럼 목록 (이름, 타입, NULL 허용, DEFAULT, COMMENT)
- PRIMARY KEY 정보
- AUTO_INCREMENT 컬럼
- 테이블 옵션 (ENGINE, CHARSET, COLLATE, COMMENT)

### 3. 비교 항목

#### 테이블 존재 여부
- ERD에만 있는 테이블
- Project에만 있는 테이블
- 공통 테이블

#### 컬럼 차이
- 컬럼 추가/삭제
- 컬럼 타입 변경
- NULL 허용 여부 변경
- DEFAULT 값 변경
- COMMENT 변경

#### PRIMARY KEY 차이
- PRIMARY KEY 존재 여부
- 복합키 구성 변경

#### 테이블 옵션 차이
- ENGINE (Project만)
- CHARSET/COLLATE (Project만)
- 테이블 COMMENT (Project만)

## 📊 정상적인 차이 (무시 가능)

### 형식 차이

다음 차이는 ERD와 Project DDL의 형식 차이로 정상입니다:

1. **PRIMARY KEY**:
   - ERD: ALTER TABLE 또는 없음
   - Project: CREATE TABLE 내부에 `PRIMARY KEY (id)`

2. **테이블 옵션**:
   - ERD: 없음
   - Project: `ENGINE = InnoDB`, `CHARSET = utf8mb4`, `COLLATE = utf8mb4_general_ci`, 테이블 COMMENT

3. **대소문자**:
   - ERD: `int`, `varchar`
   - Project: `INT`, `VARCHAR`

4. **공백/탭**:
   - ERD: 탭으로 구분
   - Project: 공백으로 정렬

### AUTO_INCREMENT 처리

```markdown
⚠️ **중요**: ERD 툴이 생성하는 `DEFAULT AUTO_INCREMENT`는 잘못된 MySQL 문법입니다.

- ERD: `DEFAULT AUTO_INCREMENT` ❌ **MySQL 문법 오류**
- Project: `AUTO_INCREMENT` ✅ **올바른 문법**

이 차이는 **ERROR**로 분류하고 ERD DDL 파일을 수정해야 합니다.

**MySQL 규칙**: AUTO_INCREMENT는 컬럼 속성이며 DEFAULT 키워드와 함께 사용할 수 없습니다.
```

### 복합키 테이블

```markdown
복합키 테이블의 경우:
- ERD: ALTER TABLE ADD CONSTRAINT PRIMARY KEY (col1, col2)
- Project: PRIMARY KEY (col1, col2) in CREATE TABLE

형식 차이이므로 정상입니다.
```

## ⚠️ 주의가 필요한 차이

다음 차이는 실제 스키마 불일치를 나타냅니다:

1. **컬럼 타입 변경**: `VARCHAR(50)` → `VARCHAR(255)`
2. **컬럼 추가/삭제**: 한쪽에만 존재하는 컬럼
3. **NULL 허용 변경**: `NULL` ↔ `NOT NULL`
4. **DEFAULT 값 변경**: 기본값 차이
5. **COMMENT 변경**: 컬럼 설명 차이

## 📝 심각도 분류

### INFO
- COMMENT 변경
- 대소문자 차이
- 공백/탭 차이

### WARNING
- 타입 길이 변경 (VARCHAR(50) → VARCHAR(255))
- 새 컬럼 추가
- DEFAULT 값 변경

### ERROR
- 컬럼 삭제
- NULL 허용 변경 (NOT NULL → NULL)
- PRIMARY KEY 변경
- `DEFAULT AUTO_INCREMENT` 사용 (잘못된 MySQL 문법)

## 📄 보고서 포맷

```markdown
## 📊 ERD DDL vs Project DDL 비교 보고서

**비교 일시**: {timestamp}
**ERD 파일**: {erd_file_path}
**Project 파일**: {project_file_path}

---

### 📈 전체 요약

| 항목 | ERD DDL | Project DDL | 비고 |
|------|---------|-------------|------|
| 총 테이블 수 | {erd_count} | {project_count} | - |
| 공통 테이블 | {common_count} | {common_count} | - |
| ERD에만 존재 | {erd_only_count} | - | ⚠️ |
| Project에만 존재 | - | {project_only_count} | ⚠️ |
| 차이 있는 테이블 | {diff_count} | {diff_count} | 🔍 |

---

### ⚠️ 테이블 존재 차이

#### ERD에만 있는 테이블 ({erd_only_count}개)
{erd_only_list}

**권장**: 이 테이블들이 Project에도 필요한지 확인하세요.

#### Project에만 있는 테이블 ({project_only_count}개)
{project_only_list}

**권장**: ERD 툴에 이 테이블들을 추가하세요.

---

### 🔍 컬럼 구조 차이

{table_by_table_diff}

---

### ✅ 차이 없는 테이블 ({same_count}개)

{same_tables_list}

---

### 🎯 권장 작업

#### ERD 툴 업데이트 필요
{erd_update_list}

#### Project DDL 업데이트 필요
{project_update_list}

#### 동기화 커맨드
- ERD → Project: `/sync-erd-to-project`
- Project → ERD: `/sync-project-to-erd`

---

**비교 완료** ✅
```

## 🔗 참조 문서

- **슬래시 커맨드**: `.claude/commands/compare-ddl.md`
- **외부 DDL**: `/Users/kyle/source/00.document/erd_ddl/`

---

**중요**: 형식 차이와 실제 차이를 명확히 구분하여 보고해야 합니다.
