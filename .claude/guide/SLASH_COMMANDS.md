# Slash Commands 가이드

Claude Code에서 사용 가능한 슬래시 커맨드 목록과 사용법입니다.

## 📋 개요

슬래시 커맨드는 `.claude/commands/` 폴더에 정의된 자동화 도구입니다. 각 커맨드는 특정 작업을 빠르게 수행할 수 있도록 도와줍니다.

## 🎯 Phase 1: 생성 커맨드

### `/generate-domain`

**용도**: DDL(CREATE TABLE)을 기반으로 완전한 CRUD 도메인 패키지를 자동 생성합니다.

**생성되는 파일**:
- Constants (Enum)
- Entity
- VO (Vo + SimpleVo)
- Param (SearchParam, AddParam, ModParam)
- Repository (JpaRepository + QueryRepository)
- Service
- Controller
- Mapper (MapStruct)

**입력 형식**:
```bash
/generate-domain {테이블명} [테이블명2 ...] [DDL파일명.sql]
```

**예시**:
```bash
# 단일 테이블
/generate-domain chapter

# 복수 테이블
/generate-domain chapter board

# DDL 파일 지정
/generate-domain chapter V2__init_api.sql

# 복수 테이블 + DDL 파일
/generate-domain chapter board V2__init_api.sql
```

**동작**:
1. DDL 파일 자동 탐색 (제공 → git staging → 자동 검색)
2. DDL 분석 (테이블 구조, PK, 컬럼, Enum 식별)
3. Enum 값 사용자에게 질문
4. 16단계 프로세스로 모든 파일 생성
5. ErrorCode, AdminAction, AdminPrivilege 자동 추가

---

### `/generate-api-doc`

**용도**: Controller를 분석하여 API 문서를 자동 생성합니다.

**생성 위치**: `/Users/kyle/source/00.document/admin_docs/{package}/{domain}/{domain}_{action}.md`

**입력 형식**:
```bash
/generate-api-doc {패키지명} [패키지명2 ...]
```

**예시**:
```bash
# 단일 패키지
/generate-api-doc board

# 복수 패키지
/generate-api-doc board lecture question
```

**생성되는 문서**:
- `{domain}_info.md` - 단일 조회
- `{domain}_list.md` - 목록 조회
- `{domain}_list_count.md` - 목록 개수
- `{domain}_add.md` - 추가
- `{domain}_mod.md` - 수정
- `{domain}_del.md` - 삭제

**동작**:
1. Controller 분석 (모든 endpoint)
2. Param/VO 구조 분석
3. Enum 파일 읽기 (실제 값 확인)
4. ErrorCode 분석
5. 각 API별 개별 문서 생성
6. ADMIN_API.md 목차 업데이트

---

### `/generate-postman`

**용도**: Controller를 분석하여 Postman 컬렉션을 자동 생성합니다.

**생성 위치**: `/Users/kyle/source/00.document/admin_docs/postman/{package}_APIs.postman_collection.json`

**입력 형식**:
```bash
/generate-postman {패키지명} [패키지명2 ...]
```

**예시**:
```bash
# 단일 패키지
/generate-postman board

# 복수 패키지
/generate-postman board lecture question
```

**생성되는 Request**:
- `{domain} info` - GET 단일 조회
- `{domain} list` - GET 목록 조회
- `{domain} list count` - GET 목록 개수
- `{domain} add` - POST 추가
- `{domain} modify` - PUT 수정
- `{domain} delete` - DELETE 삭제

**동작**:
1. Controller 분석
2. Param/VO 구조 분석
3. Enum 파일 읽기
4. Request Body 예시 생성
5. Postman v2.1 포맷으로 컬렉션 생성

**네이밍 규칙**:
- 환경 변수: `{{admin local}}`, `{{jwt token}}`
- page는 1부터 시작
- Enum은 문자열로 표현 (예: "HIDDEN")

---

### `/generate-repo-test`

**용도**: Repository와 Entity를 분석하여 테스트 클래스를 자동 생성합니다.

**생성 위치**: `src/test/java/com/firsthabit/chalk/{package}/repository/{Domain}RepositoryTest.java`

**입력 형식**:
```bash
/generate-repo-test {패키지명} [패키지명2 ...]
```

**예시**:
```bash
# 단일 패키지
/generate-repo-test board

# 복수 패키지
/generate-repo-test board lecture question
```

**생성되는 테스트**:
- `save_{domain}_then_saved` - 저장 테스트
- `findById_existing{domain}_then_found` - 조회 테스트
- `delete_{domain}_then_deleted` - 삭제 테스트
- JPA Repository 커스텀 메서드 테스트
- QueryRepository 메서드 테스트 (있으면)

**동작**:
1. Repository 파일 찾기 및 분석
2. Entity 파일 읽기
3. Enum 파일 읽기 (실제 값 사용)
4. Given-When-Then 패턴으로 테스트 생성
5. QueryDsLTestConfig 업데이트 (QueryRepository 있으면)

**주의사항**:
- Enum 값은 절대 추론하지 않음
- 필수 필드(nullable=false) 모두 설정
- createdAt, updatedAt 수동 설정 필수

---

## 🔄 Phase 2: 동기화 커맨드

### `/sync-ddl-changes`

**용도**: DDL 변경사항을 감지하고 관련 코드를 자동 업데이트합니다.

**입력 형식**:
```bash
/sync-ddl-changes {테이블명} [테이블명2 ...] [DDL파일명.sql]
```

**예시**:
```bash
/sync-ddl-changes chapter
/sync-ddl-changes chapter V5__alter_chapter.sql
```

**동작**:
1. DDL 변경사항 감지 (ADD, DROP, MODIFY COLUMN)
2. Entity, VO, Param 자동 업데이트
3. QueryRepository 검토 필요 사항 알림

**상태**: ⚠️ 복잡도 높음 - 추후 구현 권장

---

### `/sync-api-doc`

**용도**: Controller 변경사항을 감지하고 API 문서를 업데이트합니다.

**입력 형식**:
```bash
/sync-api-doc {패키지명} [패키지명2 ...]
```

**예시**:
```bash
/sync-api-doc board
/sync-api-doc board lecture
```

**동작**:
1. 기존 API 문서 읽기
2. Controller 최신 상태 분석
3. 변경사항 감지 (추가/수정/삭제)
4. 문서 완전 재생성 (전략 1)

---

### `/sync-postman`

**용도**: Controller 변경사항을 감지하고 Postman 컬렉션을 업데이트합니다.

**입력 형식**:
```bash
/sync-postman {패키지명} [패키지명2 ...]
```

**예시**:
```bash
/sync-postman board
/sync-postman board lecture
```

**동작**:
1. 기존 Postman 컬렉션 읽기
2. Controller 최신 상태 분석
3. 변경사항 감지
4. 컬렉션 완전 재생성

**주의**: 사용자 커스텀 설정(saved responses) 손실 가능

---

## 🔄 Phase 3: DDL 동기화 커맨드

ERD 툴의 DDL과 프로젝트 DDL을 양방향으로 동기화하는 커맨드입니다.

### `/sync-erd-to-project`

**용도**: ERD 툴에서 export한 DDL을 프로젝트 DDL 형식으로 변환하여 반영합니다.

**입력 형식**:
```bash
/sync-erd-to-project [erd_file] [project_file]
```

**예시**:
```bash
# 기본 파일 사용
/sync-erd-to-project

# ERD 파일만 지정
/sync-erd-to-project /path/to/erd.sql

# 두 파일 모두 지정
/sync-erd-to-project /path/to/erd.sql V3__update.sql
```

**기본값**:
- ERD 파일: `/Users/kyle/source/00.document/erd_ddl/contents.sql`
- Project 파일: `src/main/resources/db/migration/V2__init_api.sql`

**주요 변환**:
- `DEFAULT AUTO_INCREMENT` → `AUTO_INCREMENT`
- 소문자 타입 → 대문자 타입 (int → INT)
- 탭 구분 → 공백 정렬
- PRIMARY KEY 자동 추가 (id + AUTO_INCREMENT → PRIMARY KEY (id))
- 테이블 옵션 추가 (ENGINE, CHARSET, COLLATE, COMMENT)

**동작**:
1. ERD DDL 파일 읽기 및 파싱
2. CREATE TABLE 문 변환
3. 복합키 테이블 확인 (사용자 질문)
4. ENUM 타입 확인 (사용자 질문)
5. 테이블 한글명 확인 (사용자 질문)
6. Project DDL 파일 업데이트

---

### `/sync-project-to-erd`

**용도**: 프로젝트 DDL 변경사항을 감지하고 ERD DDL 형식으로 역변환하여 반영합니다.

**입력 형식**:
```bash
/sync-project-to-erd [git_ref] [erd_file]
```

**예시**:
```bash
# Git staging area의 변경사항 사용
/sync-project-to-erd

# 특정 commit의 변경사항 사용
/sync-project-to-erd abc1234

# ERD 파일 지정
/sync-project-to-erd staging /path/to/erd.sql
```

**기본값**:
- Git 참조: staging (또는 working directory)
- ERD 파일: `/Users/kyle/source/00.document/erd_ddl/contents.sql`

**주요 변환**:
- `AUTO_INCREMENT` (PRIMARY KEY 앞) → 컬럼에 `AUTO_INCREMENT` 추가
- 대문자 타입 → 소문자 타입 (INT → int)
- 공백 정렬 → 탭 구분
- PRIMARY KEY 라인 제거
- 테이블 옵션 제거 (ENGINE, CHARSET, COLLATE, 테이블 COMMENT)

**동작**:
1. Git 변경사항 감지 (staging, commit, 또는 working directory)
2. 변경된 SQL 파일에서 테이블 추출
3. Project DDL → ERD DDL 형식 변환
4. ERD DDL 파일 업데이트
5. 변경 목록 저장 (`changes_${timestamp}.md`)

**일반적인 워크플로우**:
```bash
# 1. Project DDL 수정
# 2. Git staging
git add src/main/resources/db/migration/*.sql

# 3. ERD DDL 동기화
/sync-project-to-erd

# 4. ERD 툴에서 업데이트
# (changes_*.md 파일 참조)
```

---

### `/compare-ddl`

**용도**: ERD DDL과 Project DDL을 비교하여 차이점을 상세히 보고합니다.

**입력 형식**:
```bash
/compare-ddl [erd_file] [project_file]
```

**예시**:
```bash
# 기본 파일 비교
/compare-ddl

# 특정 파일 비교
/compare-ddl /path/to/erd.sql V3__update.sql
```

**기본값**:
- ERD 파일: `/Users/kyle/source/00.document/erd_ddl/contents.sql`
- Project 파일: `src/main/resources/db/migration/V2__init_api.sql`

**비교 항목**:
1. **테이블 존재 여부**:
   - ERD에만 있는 테이블
   - Project에만 있는 테이블
   - 공통 테이블

2. **컬럼 차이**:
   - 컬럼 추가/삭제
   - 컬럼 타입 변경
   - NULL 허용 여부 변경
   - DEFAULT 값 변경
   - COMMENT 변경

3. **PRIMARY KEY 차이**

4. **테이블 옵션 차이** (PROJECT만)

**심각도 분류**:
- **INFO**: COMMENT 변경, 대소문자 차이 등 무시 가능
- **WARNING**: 타입 길이 변경, 새 컬럼 추가 등 확인 필요
- **ERROR**: 컬럼 삭제, NULL 허용 변경 등 데이터 영향 가능

**동작**:
1. ERD DDL 파일 읽기 및 파싱
2. Project DDL 파일 읽기 및 파싱
3. 테이블 목록 비교
4. 각 테이블의 컬럼 구조 비교
5. 차이점 분석 및 분류
6. 상세 비교 보고서 생성

**비교 → 동기화 워크플로우**:
```bash
# 1. DDL 비교
/compare-ddl

# 2. 차이점 확인 후 동기화 결정

# 3-A. ERD 변경사항을 Project에 반영
/sync-erd-to-project

# 3-B. Project 변경사항을 ERD에 반영
/sync-project-to-erd

# 4. 다시 비교하여 검증
/compare-ddl
```

---

## 💡 일반적인 워크플로우

### 새 도메인 추가 시

```bash
# 1. DDL 작성 후
/generate-domain {domain}

# 2. API 문서 생성
/generate-api-doc {package}

# 3. Postman 컬렉션 생성
/generate-postman {package}

# 4. Repository 테스트 생성
/generate-repo-test {package}

# 5. 테스트 실행
./gradlew test --tests "{Domain}RepositoryTest"
```

### 기존 도메인 수정 시

```bash
# 1. DDL 변경 반영
/sync-ddl-changes {domain} {ddl_file.sql}

# 2. API 문서 동기화
/sync-api-doc {package}

# 3. Postman 동기화
/sync-postman {package}
```

### ERD와 프로젝트 DDL 동기화

```bash
# ERD 변경 → 프로젝트 반영
# 1. ERD 툴에서 DDL export
# 2. contents.sql 업데이트
# 3. 프로젝트로 동기화
/sync-erd-to-project

# 4. 생성된 도메인 패키지 생성
/generate-domain {new_table}

# 프로젝트 변경 → ERD 반영
# 1. 프로젝트 DDL 수정
# 2. Git staging
git add src/main/resources/db/migration/*.sql

# 3. ERD로 동기화
/sync-project-to-erd

# 4. ERD 툴에서 업데이트 (changes_*.md 참조)

# DDL 검증
# 정기적으로 두 DDL이 동기화되어 있는지 확인
/compare-ddl
```

---

## 🎯 사용 팁

1. **Enum 값 준비**: `/generate-domain` 실행 전에 Enum 값들을 미리 정리해두세요.

2. **순차 실행**: 생성 커맨드는 순서대로 실행하세요 (domain → api-doc → postman → test).

3. **복수 패키지**: 여러 패키지를 한 번에 처리할 수 있습니다.

4. **검증 필수**: 생성 후 반드시 코드 리뷰와 테스트를 수행하세요.

5. **board 패키지 참조**: 생성된 코드가 맞는지 board 패키지와 비교해보세요.

6. **DDL 동기화**: ERD 툴과 프로젝트 DDL을 정기적으로 동기화하고 비교하세요.

7. **변경 로그**: `/sync-project-to-erd` 실행 시 생성되는 `changes_*.md` 파일을 ERD 업데이트 시 참조하세요.

---

**참고**: 모든 커맨드는 `.claude/commands/` 폴더에서 상세 내용을 확인할 수 있습니다.
