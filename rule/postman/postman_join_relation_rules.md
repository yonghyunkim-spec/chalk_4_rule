# 조인 관계 API Postman 생성 규칙

## 📋 개요

조인 관계 API (QuestionKeyword, QuestionChapter 등)는 일반 CRUD API와 다른 특수한 구조를 가지므로, 별도의 Postman 생성 규칙이 필요합니다.

이 문서는 조인 테이블을 통해 두 엔티티 간의 M:N 관계를 관리하는 API의 Postman 컬렉션 생성 규칙을 정의합니다.

---

## 🎯 적용 대상

### Controller 패턴

다음과 같은 패턴의 Controller에 적용:
- `QuestionKeywordController` - 문제-키워드 관계
- `QuestionChapterController` - 문제-챕터 관계
- `QuestionLectureSetController` - 문제-강의세트 관계 (향후)
- 기타 `{Entity1}{Entity2}Controller` 패턴

### 특징

- 복합키(Composite Key) 사용: `(entity1Id, entity2Id)`
- 양방향 조회 API: A→B, B→A
- 벌크 추가 API: `/list` 엔드포인트
- 수정(PUT) API 없음: 관계는 추가/삭제만 가능

---

## 🔍 일반 CRUD API와의 차이점

| 항목 | 일반 CRUD API | 조인 관계 API |
|------|--------------|--------------|
| **Primary Key** | 단일 키 (Long id) | 복합 키 (entity1Id + entity2Id) |
| **조회 방향** | 단방향 | 양방향 (A→B, B→A) |
| **수정 API** | 있음 (PUT) | ❌ 없음 |
| **벌크 추가** | 일반적으로 없음 | ✅ 있음 (`/list` 엔드포인트) |
| **Response** | 단일 엔티티 데이터 | 관계 + 조인된 데이터 |
| **Request Name** | `{resource} {action}` | `{resource1}-{resource2} {action}` |

---

## 📝 조인 관계 API 6가지 패턴

모든 조인 관계 API는 다음 6가지 패턴을 따릅니다:

### 1. A→B 조회 (양방향 1)

**예시**: 특정 문제에 연결된 키워드 목록 조회

```json
{
  "name": "question-keyword keywords by question",
  "request": {
    "method": "GET",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ],
    "url": {
      "raw": "{{server url}}/admin/v1/questions/{questionId}/keywords/details",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "{questionId}", "keywords", "details"]
    },
    "description": "특정 문제에 연결된 키워드 목록 조회 API"
  }
}
```

**네이밍 규칙**:
- `{domain1}-{domain2} {target} by {criteria}`
- `keywords by question`: "question을 기준으로 keywords를 가져온다"

### 2. B→A 조회 (양방향 2)

**예시**: 특정 키워드에 연결된 문제 목록 조회

```json
{
  "name": "question-keyword questions by keyword",
  "request": {
    "method": "GET",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ],
    "url": {
      "raw": "{{server url}}/admin/v1/keywords/{keywordId}/questions/details",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "keywords", "{keywordId}", "questions", "details"]
    },
    "description": "특정 키워드에 연결된 문제 목록 조회 API"
  }
}
```

**네이밍 규칙**:
- `questions by keyword`: "keyword를 기준으로 questions를 가져온다"

**⚠️ 양방향 조회 대칭성**:
```
keywords by question ↔ questions by keyword
chapters by question ↔ questions by chapter
```
- 완벽한 대칭 구조를 유지해야 함
- 네이밍만으로 조회 방향이 명확해야 함

### 3. 전체 관계 조회

**예시**: 모든 문제-키워드 관계 조회

```json
{
  "name": "question-keyword all relationships",
  "request": {
    "method": "GET",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ],
    "url": {
      "raw": "{{server url}}/admin/v1/question-keywords",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "question-keywords"]
    },
    "description": "모든 문제-키워드 관계 조회 API"
  }
}
```

**네이밍 규칙**:
- `{domain1}-{domain2} all relationships`

### 4. 단일 관계 추가

**예시**: 문제-키워드 관계 1:1 추가

```json
{
  "name": "question-keyword add",
  "request": {
    "method": "POST",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      },
      {
        "key": "Content-Type",
        "value": "application/json"
      }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"questionId\": 1, // 문제 고유번호\n  \"keywordId\": 1 // 키워드 고유번호\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/question-keywords",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "question-keywords"]
    },
    "description": "문제-키워드 관계 추가 API"
  }
}
```

**Request Body 규칙**:
- 두 엔티티의 ID만 포함
- adminId 제외 (Controller에서 설정)
- 각 필드에 주석 필수

### 5. 벌크 관계 추가 (⭐ 핵심)

**예시**: 1개 문제에 여러 키워드를 한번에 연결

```json
{
  "name": "question-keyword add list",
  "request": {
    "method": "POST",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      },
      {
        "key": "Content-Type",
        "value": "application/json"
      }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"questionId\": 1, // 문제 고유번호 (부모값 1개)\n  \"keywordIds\": [1, 2, 3] // 키워드 고유번호 리스트 (자식값 배열)\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/question-keywords/list",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "question-keywords", "list"]
    },
    "description": "문제-키워드 관계 벌크 추가 API (하나의 문제에 여러 키워드를 한번에 연결)"
  }
}
```

**벌크 추가 규칙**:
- ✅ URL 패턴: `/{resource}/list`
- ✅ 부모 ID: 단수형 필드 (`questionId`)
- ✅ 자식 IDs: 복수형 배열 (`keywordIds`, `chapterIds`)
- ✅ 주석: "(부모값 1개)", "(자식값 배열)" 명시
- ✅ Description: "하나의 {부모}에 여러 {자식}을 한번에 연결" 형태

**⚠️ 자주 발생하는 실수**:
```json
// ❌ 잘못된 예: 필드명이 복수형이 아님
{
  "questionId": 1,
  "keywordId": [1, 2, 3]  // ❌ keywordId (단수)
}

// ✅ 올바른 예: 배열 필드는 복수형
{
  "questionId": 1,
  "keywordIds": [1, 2, 3]  // ✅ keywordIds (복수)
}
```

### 6. 관계 삭제 (복합키)

**예시**: 문제-키워드 관계 삭제

```json
{
  "name": "question-keyword delete",
  "request": {
    "method": "DELETE",
    "header": [
      {
        "key": "Authorization",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ],
    "url": {
      "raw": "{{server url}}/admin/v1/questions/{questionId}/keywords/{keywordId}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "{questionId}", "keywords", "{keywordId}"]
    },
    "description": "문제-키워드 관계 삭제 API"
  }
}
```

**복합키 Path Variable 규칙**:
- ✅ 두 개의 Path Variable 사용
- ✅ RESTful 계층 구조: `/{entity1}/{id1}/{entity2}/{id2}`
- ✅ 순서: 주 엔티티(부모) 먼저, 관련 엔티티(자식) 나중
- ✅ Content-Type 헤더 없음 (DELETE는 Request Body 없음)

**⚠️ 자주 발생하는 실수**:
```json
// ❌ 잘못된 예: DELETE에 Content-Type 포함
"header": [
  {
    "key": "Authorization",
    "value": "{{jwt token}}"
  },
  {
    "key": "Content-Type",  // ❌ DELETE는 불필요!
    "value": "application/json"
  }
]

// ✅ 올바른 예: DELETE는 Authorization만
"header": [
  {
    "key": "Authorization",
    "value": "{{jwt token}}"
  }
]
```

---

## 🏷️ 네이밍 규칙

### Collection 이름

**형식**: `{Entity1}{Entity2} APIs`

**예시**:
- `QuestionKeyword APIs`
- `QuestionChapter APIs`
- `QuestionLectureSet APIs`

### Request 이름

**형식**: `{domain1}-{domain2} {action}`

| 패턴 | Request Name 형식 | 예시 |
|------|-------------------|------|
| A→B 조회 | `{domain1}-{domain2} {targets} by {criteria}` | `question-keyword keywords by question` |
| B→A 조회 | `{domain1}-{domain2} {targets} by {criteria}` | `question-keyword questions by keyword` |
| 전체 관계 조회 | `{domain1}-{domain2} all relationships` | `question-keyword all relationships` |
| 단일 추가 | `{domain1}-{domain2} add` | `question-keyword add` |
| 벌크 추가 | `{domain1}-{domain2} add list` | `question-keyword add list` |
| 삭제 | `{domain1}-{domain2} delete` | `question-keyword delete` |

**규칙**:
- 소문자 + 하이픈 + 공백
- domain1과 domain2는 하이픈(`-`)으로 연결
- action 앞은 공백

**잘못된 예**:
```
❌ QuestionKeyword Add
❌ question_keyword_add
❌ questionKeyword add
❌ question keyword add  (하이픈 누락)
```

**올바른 예**:
```
✅ question-keyword add
✅ question-chapter delete
✅ question-keyword add list
```

---

## 📄 Request Body 규칙

### 단일 추가

**구조**:
```json
{
  "entity1Id": 1,  // 첫 번째 엔티티 ID
  "entity2Id": 1   // 두 번째 엔티티 ID
  // adminId 제외 (Controller에서 설정)
}
```

**주석 규칙**:
```json
{
  "questionId": 1, // 문제 고유번호
  "keywordId": 1 // 키워드 고유번호
}
```

### 벌크 추가 (⭐ 중요)

**구조**:
```json
{
  "entity1Id": 1,          // 부모 1개 (단수형)
  "entity2Ids": [1, 2, 3]  // 자식 N개 (복수형 배열)
  // adminId 제외
}
```

**주석 규칙** (필수!):
```json
{
  "questionId": 1, // 문제 고유번호 (부모값 1개)
  "keywordIds": [1, 2, 3] // 키워드 고유번호 리스트 (자식값 배열)
}
```

**⚠️ 핵심 포인트**:
- 부모 ID: 단수형 필드명 (`questionId`)
- 자식 IDs: **복수형** 필드명 (`keywordIds`, `chapterIds`)
- 주석에 **(부모값 1개)**, **(자식값 배열)** 명시 필수
- 1:N 관계 구조를 명확히 표현

---

## 🔍 검증 체크리스트

### 기본 구조

- [ ] Collection 이름: `{Entity1}{Entity2} APIs` 형식
- [ ] Collection-level auth 제거됨
- [ ] variable에 `server url`, `jwt token` 설정
- [ ] 6가지 API 패턴 모두 포함

### 양방향 조회

- [ ] A→B 조회 API 존재
- [ ] B→A 조회 API 존재
- [ ] 네이밍이 대칭 구조: `{target} by {criteria}` ↔ 반대
- [ ] URL 경로가 올바름: `/{criteria}/{id}/{target}/details`

### 벌크 추가

- [ ] URL 패턴: `/{resource}/list`
- [ ] Request Body에 부모 ID (단수형)
- [ ] Request Body에 자식 IDs (복수형 배열)
- [ ] 주석에 "(부모값 1개)", "(자식값 배열)" 명시
- [ ] Description에 "하나의 {부모}에 여러 {자식}" 표현

### 복합키 삭제

- [ ] 두 개의 Path Variable 사용
- [ ] RESTful 계층 구조: `/{entity1}/{id1}/{entity2}/{id2}`
- [ ] Authorization 헤더 존재
- [ ] Content-Type 헤더 **없음** (⚠️ 중요)

### Request 이름

- [ ] 모두 소문자 + 하이픈 + 공백
- [ ] `{domain1}-{domain2}` 패턴
- [ ] 양방향 조회: `{target} by {criteria}` 형식
- [ ] 전체 관계: `all relationships`
- [ ] 벌크 추가: `add list`

### Header

- [ ] 모든 request에 Authorization 헤더
- [ ] GET, DELETE는 Content-Type **없음**
- [ ] POST만 Content-Type 있음
- [ ] `{{jwt token}}` 변수 사용

---

## 🚨 자주 발생하는 실수

### 1. DELETE에 Content-Type 포함

**❌ 잘못된 예**:
```json
{
  "method": "DELETE",
  "header": [
    {"key": "Authorization", "value": "{{jwt token}}"},
    {"key": "Content-Type", "value": "application/json"}  // ❌
  ]
}
```

**✅ 올바른 예**:
```json
{
  "method": "DELETE",
  "header": [
    {"key": "Authorization", "value": "{{jwt token}}"}
  ]
}
```

### 2. 벌크 추가 필드명 단수형 사용

**❌ 잘못된 예**:
```json
{
  "questionId": 1,
  "keywordId": [1, 2, 3]  // ❌ 배열인데 단수형
}
```

**✅ 올바른 예**:
```json
{
  "questionId": 1,
  "keywordIds": [1, 2, 3]  // ✅ 배열은 복수형
}
```

### 3. 양방향 조회 네이밍 비대칭

**❌ 잘못된 예**:
```
question-keyword keyword list by question
question-keyword question info by keyword
```
- "list" vs "info" 비대칭
- "keyword" vs "question" 단수/복수 불일치

**✅ 올바른 예**:
```
question-keyword keywords by question
question-keyword questions by keyword
```
- 완벽한 대칭 구조
- 복수형 일관성

### 4. 복합키 Path Variable 순서 오류

**❌ 잘못된 예**:
```
/admin/v1/keywords/{keywordId}/questions/{questionId}
```
- 주 엔티티가 나중에 옴

**✅ 올바른 예**:
```
/admin/v1/questions/{questionId}/keywords/{keywordId}
```
- 주 엔티티(Question)가 먼저 옴
- RESTful 계층 구조 표현

### 5. adminId 포함

**❌ 잘못된 예**:
```json
{
  "adminId": 1,  // ❌ 포함하면 안 됨
  "questionId": 1,
  "keywordId": 1
}
```

**✅ 올바른 예**:
```json
{
  "questionId": 1,
  "keywordId": 1
  // adminId 제외 (Controller에서 설정)
}
```

### 6. 벌크 추가 주석 누락

**❌ 잘못된 예**:
```json
{
  "questionId": 1,
  "keywordIds": [1, 2, 3]
}
```
- 주석 없어서 1:N 구조가 불명확

**✅ 올바른 예**:
```json
{
  "questionId": 1, // 문제 고유번호 (부모값 1개)
  "keywordIds": [1, 2, 3] // 키워드 고유번호 리스트 (자식값 배열)
}
```
- "(부모값 1개)", "(자식값 배열)" 주석으로 구조 명확화

---

## 📊 패턴 요약표

| API | HTTP Method | URL 패턴 | Request Name | Request Body | 특징 |
|-----|-------------|----------|--------------|--------------|------|
| A→B 조회 | GET | `/{entity1s}/{id1}/{entity2s}/details` | `{e1}-{e2} {e2s} by {e1}` | 없음 | 양방향 1 |
| B→A 조회 | GET | `/{entity2s}/{id2}/{entity1s}/details` | `{e1}-{e2} {e1s} by {e2}` | 없음 | 양방향 2 |
| 전체 관계 | GET | `/{entity1}-{entity2s}` | `{e1}-{e2} all relationships` | 없음 | 전체 조회 |
| 단일 추가 | POST | `/{entity1}-{entity2s}` | `{e1}-{e2} add` | 2개 ID | 1:1 |
| 벌크 추가 | POST | `/{entity1}-{entity2s}/list` | `{e1}-{e2} add list` | 1개 ID + 배열 | 1:N |
| 관계 삭제 | DELETE | `/{entity1s}/{id1}/{entity2s}/{id2}` | `{e1}-{e2} delete` | 없음 | 복합키 |

---

## 🔗 관련 문서

- **기본 Postman 생성 규칙**: `rule/postman/postman_generation_rules.md`
- **Postman 포맷 규격**: `rule/postman/postman_format_spec.md`
- **Postman 예시**: `rule/postman/postman_examples.md`
- **조인 관계 API 문서화 규칙**: `rule/api_documentation/api_doc_join_relation_rules.md`

---

## 💡 핵심 원칙

1. **양방향 조회 대칭성**: 네이밍과 구조가 완벽히 대칭이어야 함
2. **복합키 표현**: DELETE URL에서 두 ID를 모두 Path Variable로 표현
3. **벌크 추가 명확성**: 부모 단수 + 자식 복수 배열, 주석 필수
4. **Content-Type 규칙**: Request Body가 있을 때만 (POST)
5. **Controller 일치**: 실제 Controller 코드와 100% 일치해야 함

---

**중요**: 조인 관계 API는 일반 CRUD와 완전히 다른 패턴이므로, 이 규칙을 **엄격히 준수**해야 합니다.
