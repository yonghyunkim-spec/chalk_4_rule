# API 문서 예시

## 📋 개요

이 문서는 템플릿을 정확히 따르는 완전한 API 문서 예시를 제공합니다.

---

## 예시 1: GET (단일 조회) - question_info.md

```markdown
# 문제 단일 조회

## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | GET |
| **Endpoint** | `/admin/v1/questions/{questionId}` |
| **Description** | 특정 문제의 상세 정보를 조회합니다. |
| **Controller / Function** | `QuestionController.getQuestion()` |

## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |

## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| questionId | Long | 문제 고유번호 | ✅ |

## Request Body

> 없음

## Success Response

| name | type | description |
|------|------|-------------|
| id | Long | 문제 고유번호 |
| questionGroupId | Integer | 문제 그룹 고유번호 |
| originQuestionId | Long | 기원 문제 고유번호 (Twin Question인 경우, nullable) |
| questionPropertyId | Long | 문제 속성 고유번호 |
| content | String | 문제 내용 |
| answer | String | 답변 내용 (JSON 포맷) |
| openFlag | String | 오픈 여부 |
| createdAt | String | 생성 일시 (ISO 8601) |
| updatedAt | String | 최종 수정 일시 (ISO 8601) |
| createdById | Integer | 생성자 ID |
| updatedById | Integer | 수정자 ID |

\`\`\`json
{
  "id": 123,
  "questionGroupId": 10,
  "originQuestionId": null,
  "questionPropertyId": 50,
  "content": "다음 중 정답은?",
  "answer": "{\"correctAnswer\": \"A\", \"options\": [\"A\", \"B\", \"C\", \"D\"]}",
  "openFlag": "OPEN",
  "createdAt": "2025-10-29T10:30:00",
  "updatedAt": "2025-10-29T12:15:00",
  "createdById": 1,
  "updatedById": 1
}
\`\`\`

## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "문제가 존재하지 않음",
  "errorCode": 142301
}
\`\`\`

## Example (cURL)

\`\`\`bash
curl -X GET https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json"
\`\`\`

## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142301 | 404 | QUESTION_NOT_EXIST | 문제가 존재하지 않음 |
```

---

## 예시 2: GET (리스트 조회) - question_list.md

```markdown
# 문제 목록 조회

## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | GET |
| **Endpoint** | `/admin/v1/questions/list` |
| **Description** | 문제 목록을 페이징하여 조회합니다. |
| **Controller / Function** | `QuestionController.getQuestionList()` |

## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |

## Path Variable

> 없음

## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| page | Integer | 페이지 번호 (기본값: 1) | ✅ |
| limit | Integer | 페이지당 개수 (기본값: 10) | ✅ |
| questionGroupId | Integer | 문제 그룹 고유번호 (필터) | ⛔ |
| openFlag | String | 오픈 여부 (필터) | ⛔ |
| searchValue | String | 검색어 (문제 내용 검색) | ⛔ |

### openFlag

| Value | Description |
|-------|-------------|
| CLOSE | 비공개 |
| OPEN | 공개 |

## Success Response

> Response는 QuestionSimpleVo 배열 형태입니다.

| name | type | description |
|------|------|-------------|
| id | Long | 문제 고유번호 |
| questionGroupId | Integer | 문제 그룹 고유번호 |
| content | String | 문제 내용 |
| openFlag | String | 오픈 여부 |
| createdAt | String | 생성 일시 (ISO 8601) |
| updatedAt | String | 최종 수정 일시 (ISO 8601) |

\`\`\`json
[
  {
    "id": 123,
    "questionGroupId": 10,
    "content": "다음 중 정답은?",
    "openFlag": "OPEN",
    "createdAt": "2025-10-29T10:30:00",
    "updatedAt": "2025-10-29T12:15:00"
  },
  {
    "id": 124,
    "questionGroupId": 10,
    "content": "다음 문제를 푸시오.",
    "openFlag": "CLOSE",
    "createdAt": "2025-10-29T11:00:00",
    "updatedAt": "2025-10-29T11:00:00"
  }
]
\`\`\`

## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "유효하지 않은 인증 토큰",
  "errorCode": 140003
}
\`\`\`

## Example (cURL)

\`\`\`bash
curl -X GET "https://api.server.com/admin/v1/questions/list?page=1&limit=10&openFlag=OPEN&searchValue=정답" \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json"
\`\`\`

## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
```

---

## 예시 3: POST (추가) - question_add.md

```markdown
# 문제 추가

## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | POST |
| **Endpoint** | `/admin/v1/questions` |
| **Description** | 새로운 문제를 추가합니다. |
| **Controller / Function** | `QuestionController.addQuestion()` |

## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |

## Path Variable

> 없음

## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| questionGroupId | Integer | 문제 그룹 고유번호 | ✅ |
| originQuestionId | Long | 기원 문제 고유번호 (Twin Question인 경우) | ⛔ |
| questionPropertyId | Long | 문제 속성 고유번호 | ✅ |
| content | String | 문제 내용 | ✅ |
| answer | String | 답변 내용 (JSON 포맷) | ✅ |
| openFlag | String | 오픈 여부 | ✅ |

### openFlag

| Value | Description |
|-------|-------------|
| CLOSE | 비공개 |
| OPEN | 공개 |

### Request Example

\`\`\`json
{
  "questionGroupId": 10,
  "originQuestionId": null,
  "questionPropertyId": 50,
  "content": "다음 중 정답은?",
  "answer": "{\"correctAnswer\": \"A\", \"options\": [\"A\", \"B\", \"C\", \"D\"]}",
  "openFlag": "OPEN"
}
\`\`\`

## Success Response

| name | type | description |
|------|------|-------------|
| id | Long | 문제 고유번호 |
| questionGroupId | Integer | 문제 그룹 고유번호 |
| originQuestionId | Long | 기원 문제 고유번호 |
| questionPropertyId | Long | 문제 속성 고유번호 |
| content | String | 문제 내용 |
| answer | String | 답변 내용 (JSON 포맷) |
| openFlag | String | 오픈 여부 |
| createdAt | String | 생성 일시 (ISO 8601) |
| updatedAt | String | 최종 수정 일시 (ISO 8601) |
| createdById | Integer | 생성자 ID |
| updatedById | Integer | 수정자 ID |

\`\`\`json
{
  "id": 125,
  "questionGroupId": 10,
  "originQuestionId": null,
  "questionPropertyId": 50,
  "content": "다음 중 정답은?",
  "answer": "{\"correctAnswer\": \"A\", \"options\": [\"A\", \"B\", \"C\", \"D\"]}",
  "openFlag": "OPEN",
  "createdAt": "2025-10-29T14:00:00",
  "updatedAt": "2025-10-29T14:00:00",
  "createdById": 1,
  "updatedById": 1
}
\`\`\`

## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "문제 그룹이 존재하지 않음",
  "errorCode": 142101
}
\`\`\`

## Example (cURL)

\`\`\`bash
curl -X POST https://api.server.com/admin/v1/questions \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "questionGroupId": 10,
    "questionPropertyId": 50,
    "content": "다음 중 정답은?",
    "answer": "{\"correctAnswer\": \"A\", \"options\": [\"A\", \"B\", \"C\", \"D\"]}",
    "openFlag": "OPEN"
  }'
\`\`\`

## Notes

### Twin Question (변형 문제) 생성 규칙

- Twin Question 생성 시 `originQuestionId`를 반드시 지정해야 합니다.
- Twin Question은 Origin Question과 동일한 `questionGroupId`를 사용해야 합니다.

### 답변 내용 형식

- `answer` 필드는 JSON 문자열 형식이어야 합니다.
- 객관식 문제의 경우: `{"correctAnswer": "A", "options": ["A", "B", "C", "D"]}`
- 주관식 문제의 경우: `{"answer": "정답 내용"}`

## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142101 | 404 | QUESTION_GROUP_NOT_EXIST | 문제 그룹이 존재하지 않음 |
| 142302 | 400 | INVALID_QUESTION_CONTENT | 유효하지 않은 문제 내용 |
| 142303 | 400 | INVALID_ANSWER_FORMAT | 유효하지 않은 답변 형식 |
```

---

## 예시 4: DELETE (삭제) - question_del.md

```markdown
# 문제 삭제

## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | DELETE |
| **Endpoint** | `/admin/v1/questions/{questionId}` |
| **Description** | 특정 문제를 삭제합니다. |
| **Controller / Function** | `QuestionController.delQuestion()` |

## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |

## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| questionId | Long | 문제 고유번호 | ✅ |

## Request Body

> 없음

## Success Response

> HTTP 204 No Content

> 본문 없음

## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "문제가 존재하지 않음",
  "errorCode": 142301
}
\`\`\`

## Example (cURL)

\`\`\`bash
curl -X DELETE https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}"
\`\`\`

## Notes

### 삭제 제약 사항

- Twin Question이 존재하는 Origin Question은 삭제할 수 없습니다.
- 삭제하려면 먼저 모든 Twin Question을 삭제해야 합니다.

## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142301 | 404 | QUESTION_NOT_EXIST | 문제가 존재하지 않음 |
| 142304 | 400 | CANNOT_DELETE_ORIGIN_QUESTION | Origin Question은 Twin Question이 존재하는 동안 삭제할 수 없음 |
```

---

## 핵심 포인트 요약

### 1. Enum 위치
- ✅ **항상 Request Body 섹션 바로 아래**
- 예시 1 (GET): Request Body가 없으므로 Enum 정의 없음
- 예시 2 (GET): openFlag enum 정의가 Request Body 바로 아래
- 예시 3 (POST): openFlag enum 정의가 Request Body 바로 아래

### 2. Error Code 위치
- ✅ **항상 문서 맨 마지막**
- 모든 예시에서 Error Code가 맨 마지막 섹션

### 3. Authorization 헤더
- ✅ description: "JWT 인증 토큰" (Bearer 제외)
- ✅ cURL: `-H "Authorization: {JWT_TOKEN}"` (Bearer 제외)

### 4. page, limit
- ✅ 예시 2에서 page, limit가 기본값이 있어도 **필수(✅)로 표시**

### 5. Success Response
- 예시 1: 단일 객체
- 예시 2: 배열 (첫 줄에 "{Vo}배열 형태" 표시)
- 예시 4: 204 No Content

### 6. Notes
- 예시 1, 2: Notes 없음 (섹션 자체 생략)
- 예시 3, 4: Notes 있음 (Error Code 이전)

---

**이 예시들을 API 문서 생성 시 참고하세요.**
