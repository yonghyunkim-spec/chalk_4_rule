# 조인 관계 API 문서화 전용 규칙

## 📋 개요

이 문서는 **조인 테이블을 통한 관계 관리 API**를 문서화할 때 적용하는 특수 규칙을 정의합니다.

일반 CRUD API(`Question`, `Keyword`, `Chapter`)와 달리, 조인 관계 API(`QuestionKeyword`, `QuestionChapter`)는 두 엔티티 간의 **관계(Relation)**를 관리하며, 다음과 같은 특징이 있습니다:

- 복합키(Composite Key) 사용
- 양방향 조회 API (A→B, B→A)
- 조인된 데이터를 포함한 VO 반환
- 벌크 추가 기능 (1:N 관계 일괄 생성)

## 🎯 적용 대상

- QuestionKeywordController
- QuestionChapterController
- QuestionLectureSetController
- 기타 조인 테이블 기반 관계 관리 API

## 🔍 핵심 차이점 요약

### 1. 엔티티 구조 차이

| 항목 | 일반 CRUD API | 조인 관계 API |
|------|--------------|--------------|
| **엔티티 예시** | Question, Keyword, Chapter | QuestionKeyword, QuestionChapter |
| **Primary Key** | 단일 키 (Long id) | 복합 키 (questionId + keywordId) |
| **주요 속성** | 엔티티 고유 데이터 (title, content 등) | 관계 메타데이터 (createdAt, createdById) |
| **비즈니스 의미** | 실체가 있는 개체 (Question은 문제) | 두 개체 간의 연결 (Question과 Keyword의 관계) |

### 2. API 엔드포인트 패턴 차이

#### 일반 CRUD API
```
GET    /admin/v1/questions/{questionId}    # 단일 조회
GET    /admin/v1/questions/list            # 목록 조회
POST   /admin/v1/questions                 # 추가
PUT    /admin/v1/questions/{questionId}    # 수정
DELETE /admin/v1/questions/{questionId}    # 삭제
```

**특징**: 단방향, 단일 ID, 수정 API 존재

#### 조인 관계 API
```
# 양방향 조회
GET    /admin/v1/questions/{questionId}/keywords/details    # Question → Keyword
GET    /admin/v1/keywords/{keywordId}/questions/details     # Keyword → Question
GET    /admin/v1/question-keywords                          # 전체 관계 조회

# 추가
POST   /admin/v1/question-keywords                          # 단일 관계 추가
POST   /admin/v1/question-keywords/list                     # 벌크 추가 (1:N)

# 삭제
DELETE /admin/v1/questions/{questionId}/keywords/{keywordId}  # 복합키로 삭제
```

**특징**: 양방향 조회, 복합키, 수정 API 없음, 벌크 추가 제공

### 3. Response VO 차이

#### 일반 CRUD API Response
```json
{
  "questionId": 123,
  "questionGroupId": 5,
  "content": "문제 내용",
  "answer": "정답",
  "createdAt": "2025-11-12T10:00:00"
}
```
**특징**: 해당 엔티티의 필드만 포함

#### 조인 관계 API Response
```json
{
  "questionId": 123,           // 관계 PK 1
  "keywordId": 10,             // 관계 PK 2
  "createdAt": "...",          // 관계 메타데이터
  "createdById": 1,            // 관계 메타데이터
  "keywordTitle": "조건문",     // 조인된 Keyword 데이터
  "keywordContent": "...",     // 조인된 Keyword 데이터
  "keywordKeywordType": "CONCEPT"  // 조인된 Keyword 데이터
}
```
**특징**: 관계 메타데이터 + 조인된 엔티티 데이터, **prefix 패턴** 사용

### 4. Service 로직 차이

**일반 CRUD**:
- 단일 엔티티 생성 및 저장
- 자체 데이터만 검증

**조인 관계 (벌크 추가)**:
- **양쪽 엔티티 검증**: 부모(Question)와 자식(Keyword) 모두 존재 확인
- **IN 절 최적화**: 자식 배열을 단일 쿼리로 일괄 조회
- **중복 체크**: 이미 존재하는 관계인지 확인
- **배치 저장**: saveAll()로 한번에 저장
- **트랜잭션**: All-or-Nothing (일부만 성공 불가)

### 5. 파일 네이밍 패턴 차이

**일반 CRUD**:
```
question_info.md              # GET /{id}
question_list.md              # GET /list
question_add.md               # POST /
question_mod.md               # PUT /{id}
question_del.md               # DELETE /{id}
```

**조인 관계**:
```
question_keyword_get_keywords_by_question.md    # GET /questions/{id}/keywords
question_keyword_get_questions_by_keyword.md    # GET /keywords/{id}/questions
question_keyword_get_all.md                     # GET /question-keywords
question_keyword_add.md                         # POST /question-keywords
question_keyword_list_add.md                    # POST /question-keywords/list
question_keyword_del.md                         # DELETE /questions/{qid}/keywords/{kid}
```

**네이밍 규칙**: `{domain1}_{domain2}_get_{target}_by_{criteria}`
- target: 조회하는 대상 (keywords, questions)
- criteria: 조회 기준 (question, keyword)
- 예: `get_keywords_by_question` = Question 기준으로 Keyword 조회

## 📝 조인 관계 API 문서화 특수 규칙

### 1. 기본 정보 섹션 작성

```markdown
## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | GET |
| **Endpoint** | `/admin/v1/questions/{questionId}/keywords/details` |
| **Description** | 특정 문제에 연결된 키워드 목록을 조회합니다. |
| **Controller / Function** | `QuestionKeywordController.getKeywordsWithDetailsByQuestionId()` |
```

**주의사항**:
- Description에 **"연결된"** 또는 **"관계"** 키워드 명시
- 양방향 조회는 방향성 명확히 표현 (A → B)

### 2. Path Variable - 복합키 처리

#### 단일 ID (양방향 조회)

```markdown
## Path Variable

| name | type | description | required |
|------|------|-------------|----------|
| questionId | number | 문제 고유번호 (Long) | ✅ |
```

#### 복합 ID (삭제)

```markdown
## Path Variable

| name | type | description | required |
|------|------|-------------|----------|
| questionId | number | 문제 고유번호 (Long) | ✅ |
| keywordId | number | 키워드 고유번호 (Long) | ✅ |
```

**주의사항**:
- 복합키를 사용하는 삭제 API는 **두 개의 Path Variable** 필요
- 타입이 다를 수 있음 (예: questionId는 Long, chapterId는 Integer)

### 3. Request Body - 벌크 추가 패턴

```markdown
## Request Body

| name | type | description | required |
|------|------|-------------|----------|
| questionId | number | 문제 고유번호 (Long) | ✅ |
| keywordIds | array | 키워드 고유번호 배열 (number 배열) | ✅ |

### Request Example

\```json
{
  "questionId": 1,
  "keywordIds": [100, 200, 300]
}
\```
```

**주의사항**:
- 부모 1개 (questionId) + 자식 배열 (keywordIds) 구조
- 배열 타입은 `array`로 표기, description에 요소 타입 명시

### 4. Success Response - 조인된 VO 문서화

```markdown
## Success Response

> Response는 QuestionKeywordVo 배열 형태입니다.

\```json
[
  {
    "questionId": 123,
    "keywordId": 10,
    "createdAt": "2025-11-12T10:30:00",
    "createdById": 1,
    "keywordTitle": "조건문",
    "keywordContent": "조건에 따라 다른 코드를 실행하는 제어문",
    "keywordKeywordType": "CONCEPT"
  }
]
\```

### QuestionKeywordVo

| name | type | description |
|------|------|-------------|
| questionId | number | 문제 고유번호 (Long) |
| keywordId | number | 키워드 고유번호 (Long) |
| createdAt | string | 생성 일시 (ISO 8601) |
| createdById | number | 생성자 ID (Integer) |
| keywordTitle | string | 키워드 제목 |
| keywordContent | string | 키워드 내용 |
| keywordKeywordType | string | 키워드 타입 |

### keywordKeywordType

키워드 타입

| Value | Description |
|-------|-------------|
| CONCEPT | 개념 |
| TYPE | 유형 |
| DEMONSTRATION | 시연 |
```

**조인 관계 특화 문서화 규칙**:

1. **필드 분류**
   - **관계 메타데이터**: questionId, keywordId, createdAt, createdById
   - **조인된 데이터**: prefix가 있는 필드 (keywordTitle, keywordContent, keywordKeywordType)

2. **Prefix 패턴**
   - 조인된 엔티티 필드는 엔티티명이 prefix
   - 예: `keyword`로 시작하면 Keyword 엔티티에서 조인된 필드
   - 예: `question`으로 시작하면 Question 엔티티에서 조인된 필드

3. **⚠️ 필수 준수 사항** (기본 규칙과 동일하지만 조인 관계에서 특히 중요)

   - **VO 파일 반드시 읽기**
     - 조인 관계 VO는 복잡하므로 **추론 절대 금지**
     - 관계 메타데이터 + 조인된 엔티티 데이터가 섞여 있어 혼란 발생 가능

   - **Enum 파일 반드시 읽기**
     - 조인된 엔티티의 Enum도 반드시 실제 파일 확인
     - 예: `keywordKeywordType`은 Keyword 엔티티의 Enum

   - **실제로 없는 필드 추가 절대 금지**
     - VO에 없는 필드를 추측해서 추가하지 말 것
     - 예: `questionOpenFlag`가 KeywordQuestionVo에 없으면 문서에도 없어야 함

   > 💡 자세한 내용: `api_doc_generation_rules.md`의 VO/Enum 처리 규칙 참고

### 5. Fail Response - 조인 관계 특화 에러

```markdown
## Fail Response

### 부모 엔티티가 존재하지 않는 경우

\```json
{
  "timestamp": "2025-11-12T10:00:00",
  "message": "존재하지 않는 문제",
  "errorCode": 142101,
  "notExistQuestion": 1001
}
\```
```

**조인 관계 특화 에러 4가지**:

1. **부모 엔티티 없음** (QUESTION_NOT_EXIST 등)
2. **자식 엔티티 없음** (벌크 추가 시 배열 중 일부 없음, KEYWORD_NOT_EXIST 등)
3. **관계 중복** (DUPLICATE_QUESTION_KEYWORD)
4. **관계 없음** (QUESTION_KEYWORD_NOT_EXIST, 삭제 시)

**additionalInfo 패턴**:
- 단일값: `"notExistQuestion": 1001`
- 배열값: `"notExistKeyword": [200, 300]`
- 중복 상황: `"questionId": 1, "existingKeywordIds": [100, 200]`

### 6. Notes 섹션 - 벌크 추가 설명

벌크 추가(`/list`) API가 있는 경우 Notes 섹션 필수 포함:

```markdown
## Notes

### 벌크 추가 처리 과정

1. 부모 엔티티(questionId) 존재 여부 검증
2. 자식 엔티티(keywordIds) 일괄 검증 (IN 절 사용)
3. 중복 관계 체크
4. 새로운 관계 엔티티 생성 (부모 1개 × 자식 N개)
5. 배치 저장 (saveAll, 한번의 INSERT 쿼리)
6. VO로 변환하여 반환

### 트랜잭션 및 중복 처리

- 모든 관계가 성공하거나 모두 실패 (원자성 보장)
- 배열에 중복된 값이 있으면 자동 제거
- 이미 DB에 존재하는 관계는 에러 발생
```

### 7. Error Code 섹션

```markdown
## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142101 | 404 | QUESTION_NOT_EXIST | 존재하지 않는 문제 |
| 142701 | 400 | DUPLICATE_QUESTION_KEYWORD | 중복된 문제-키워드 관계 |
| 144001 | 404 | KEYWORD_NOT_EXIST | 존재하지 않는 키워드 |
```

**조인 관계 에러 순서**:
1. 공통 인증/권한 에러 (140xxx)
2. 부모 엔티티 에러 (예: QUESTION_NOT_EXIST)
3. 자식 엔티티 에러 (예: KEYWORD_NOT_EXIST)
4. 관계 관련 에러 (예: DUPLICATE_QUESTION_KEYWORD)

## 🔍 검증 체크리스트 (조인 관계 API 전용)

### 엔드포인트 및 네이밍
- [ ] 양방향 조회 API가 각각 문서화되었는가?
  - [ ] A → B 조회 (`/questions/{id}/keywords`)
  - [ ] B → A 조회 (`/keywords/{id}/questions`)
- [ ] 파일명이 `get_{target}_by_{criteria}` 패턴을 따르는가?
- [ ] 전체 관계 조회 API가 있는가? (`/question-keywords`)
- [ ] 벌크 추가 API가 별도 문서화되었는가? (`/question-keywords/list`)

### Path Variable 및 Request Body
- [ ] 삭제 API의 Path Variable이 복합키(두 개 ID)인가?
- [ ] 벌크 추가의 Request Body가 부모 1개 + 자식 배열 구조인가?
- [ ] 배열 필드의 타입이 `array`로 표기되었는가?
- [ ] 배열 필드의 description에 요소 타입이 명시되었는가?

### Response VO 문서화
- [ ] **VO 파일을 실제로 읽었는가?** (추론 금지!)
- [ ] 관계 메타데이터 필드가 포함되었는가? (questionId, keywordId, createdAt, createdById)
- [ ] 조인된 엔티티 필드가 포함되었는가? (prefix 있는 필드)
- [ ] 조인된 필드의 prefix가 정확한가? (keywordTitle, questionContent 등)
- [ ] **VO에 없는 필드를 추측해서 추가하지 않았는가?**
- [ ] 조인된 엔티티의 Enum이 subsection으로 문서화되었는가?

### Fail Response
- [ ] 부모 엔티티 없음 에러가 포함되었는가?
- [ ] 자식 엔티티 없음 에러가 포함되었는가? (벌크 추가 시)
- [ ] 중복 관계 에러가 포함되었는가? (추가 시)
- [ ] 관계 없음 에러가 포함되었는가? (삭제 시)
- [ ] additionalInfo 형식이 정확한가? (단일값 vs 배열)

### Notes 섹션
- [ ] 벌크 추가 설명이 포함되었는가?
- [ ] 처리 과정이 단계별로 설명되었는가?
- [ ] IN 절, saveAll 등 성능 최적화 방법이 언급되었는가?
- [ ] 트랜잭션 동작이 설명되었는가?
- [ ] 중복 제거 정책이 설명되었는가?

### Content-Type 헤더
- [ ] GET/DELETE는 Content-Type이 **없는가**?
- [ ] POST/PUT/PATCH는 Content-Type이 **있는가**?

## 🚨 조인 관계 API 문서화 시 자주 발생하는 실수

### 1. VO에 없는 필드를 추측해서 추가

```markdown
# ❌ 잘못된 예 (실제로 없는 필드 추가)
### KeywordQuestionVo

| questionId | number | 문제 고유번호 (Long) |
| keywordId | number | 키워드 고유번호 (Long) |
| questionOpenFlag | string | 오픈 여부 |  ← 실제 VO에 없음!

# ✅ 올바른 예 (VO 파일을 읽고 실제 필드만 작성)
### KeywordQuestionVo

| questionId | number | 문제 고유번호 (Long) |
| keywordId | number | 키워드 고유번호 (Long) |
| createdAt | string | 생성 일시 (ISO 8601) |
| createdById | number | 생성자 ID (Integer) |
// questionOpenFlag 없음!
```

### 2. 파일명 패턴 혼동

```markdown
# ❌ 잘못된 예 (방향성 반대)
question_keyword_get_questions_by_keyword.md
→ Endpoint: /questions/{id}/keywords  ← 불일치!

# ✅ 올바른 예 (파일명과 엔드포인트 일치)
question_keyword_get_keywords_by_question.md
→ Endpoint: /questions/{id}/keywords  ← 일치!
```

**규칙**: `get_{target}_by_{criteria}`
- target: 가져오는 것 (keywords)
- criteria: 기준 (question)

### 3. GET/DELETE에 Content-Type 추가

```markdown
# ❌ 잘못된 예 (GET인데 Content-Type 포함)
| **Method** | GET |
| **Endpoint** | `/admin/v1/questions/{id}/keywords` |

## Request Header

| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | application/json | ✅ |  ← 불필요!

# ✅ 올바른 예 (GET은 Content-Type 없음)
| **Method** | GET |

## Request Header

| Authorization | String | JWT 인증 토큰 | ✅ |
```

### 4. 복합키 Path Variable 누락

```markdown
# ❌ 잘못된 예 (삭제인데 ID 하나만)
DELETE /admin/v1/questions/{questionId}/keywords

## Path Variable

| questionId | number | 문제 고유번호 | ✅ |
// keywordId 누락!

# ✅ 올바른 예 (복합키 두 개 모두 포함)
DELETE /admin/v1/questions/{questionId}/keywords/{keywordId}

## Path Variable

| questionId | number | 문제 고유번호 | ✅ |
| keywordId | number | 키워드 고유번호 | ✅ |
```

### 5. 조인된 필드의 Enum 누락

```markdown
# ❌ 잘못된 예 (Enum subsection 없음)
### QuestionKeywordVo

| keywordKeywordType | string | 키워드 타입 |

// keywordKeywordType Enum 정의 없음!

# ✅ 올바른 예 (Enum subsection 추가)
### QuestionKeywordVo

| keywordKeywordType | string | 키워드 타입 |

### keywordKeywordType

키워드 타입

| Value | Description |
|-------|-------------|
| CONCEPT | 개념 |
| TYPE | 유형 |
| DEMONSTRATION | 시연 |
```

### 6. 벌크 추가 Request Body 구조 틀림

```markdown
# ❌ 잘못된 예 (배열 구조 틀림)
{
  "relations": [
    {"questionId": 1, "keywordId": 100},
    {"questionId": 1, "keywordId": 200}
  ]
}

# ✅ 올바른 예 (부모 1개 + 자식 배열)
{
  "questionId": 1,
  "keywordIds": [100, 200, 300]
}
```

## 📊 조인 관계 API 6가지 패턴 요약

| API 유형 | Method | Endpoint | Response |
|---------|--------|----------|----------|
| A → B 조회 | GET | `/questions/{id}/keywords/details` | Keyword 정보가 포함된 VO 배열 |
| B → A 조회 | GET | `/keywords/{id}/questions/details` | Question 정보가 포함된 VO 배열 |
| 전체 관계 조회 | GET | `/question-keywords` | 전체 관계 VO 배열 |
| 단일 관계 추가 | POST | `/question-keywords` | 200 OK (본문 없음) |
| 벌크 관계 추가 | POST | `/question-keywords/list` | 생성된 관계 VO 배열 |
| 관계 삭제 | DELETE | `/questions/{qid}/keywords/{kid}` | 204 No Content |

## 🔗 관련 문서

- **기본 규칙**: `rule/api_documentation/api_doc_generation_rules.md`
- **템플릿**: `rule/api_documentation/api_doc_template.md`
- **예시**: `rule/api_documentation/api_doc_examples.md`

---

**작성일**: 2025-11-12
**버전**: 1.1.0 (최적화)
**적용 대상**: QuestionKeyword, QuestionChapter API 문서화
