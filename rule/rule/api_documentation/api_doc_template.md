# API 문서 템플릿

## 📋 개요

이 문서는 API 문서의 **강제 템플릿**입니다. 모든 API 문서는 이 템플릿을 **정확히** 따라야 합니다.

## 🚨 중요 규칙

- ✅ **이 템플릿의 섹션 순서를 절대 변경하지 마세요**
- ✅ **각 섹션의 포맷을 정확히 따르세요**
- ✅ **표시된 주의사항을 반드시 지키세요**

---

## 📄 템플릿 (전체)

```markdown
# {API 이름}

## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | {HTTP_METHOD} |
| **Endpoint** | `{endpoint}` |
| **Description** | {설명} |
| **Controller / Function** | `{ControllerName}.{methodName}()` |

## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |

⚠️ Authorization description: "JWT 인증 토큰" (Bearer 문구 제외)

## Path Variable

{path_variable_table_or_없음}

## Request Body

{request_body_table_or_없음}

{enum_definitions_if_exists}

{request_example_if_body_exists}

## Success Response

{success_response_table_and_example}

## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "{에러 메시지}",
  "errorCode": {에러코드}
}
\`\`\`

## Example (cURL)

\`\`\`bash
{curl_example}
\`\`\`

{notes_if_needed}

## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 ({DOMAIN}_MANAGE 권한 필요) |
| {code} | {status} | {message} | {explain} |
```

---

## 📑 섹션별 상세 템플릿

### 1. 기본 정보

```markdown
## 기본 정보

| 항목 | 내용 |
|------|------|
| **Method** | GET |
| **Endpoint** | `/admin/v1/questions/{questionId}` |
| **Description** | 특정 문제의 상세 정보를 조회합니다. |
| **Controller / Function** | `QuestionController.getQuestion()` |
```

**변수 설명**:
- `{HTTP_METHOD}`: GET, POST, PUT, DELETE
- `{endpoint}`: 실제 API endpoint
- `{설명}`: API 기능 설명 (한 문장, 마침표 포함)
- `{ControllerName}`: Controller 클래스명
- `{methodName}`: Controller 메서드명

**제외 항목**:
- Permission (표시 안 함)
- Tag (표시 안 함)

---

### 2. Request Header

```markdown
## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |
```

**⚠️ 중요 규칙**:
- Authorization description: **반드시** "JWT 인증 토큰"
- **절대** "Bearer {token}" 문구 포함 금지
- Content-Type은 항상 `application/json`

**잘못된 예**:
```markdown
| Authorization | String | Bearer {token} 형식의 JWT 인증 토큰 | ✅ |  # 잘못됨!
```

---

### 3. Path Variable

#### Case 1: Path Variable이 있는 경우

```markdown
## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| questionId | Long | 문제 고유번호 | ✅ |
| chapterId | Long | 챕터 고유번호 | ✅ |
```

**변수 설명**:
- `{name}`: Path Variable 이름 (camelCase)
- `{type}`: Java 타입 (Long, Integer, String 등)
- `{description}`: 설명
- `{required}`: 항상 ✅ (Path Variable은 모두 필수)

#### Case 2: Path Variable이 없는 경우

```markdown
## Path Variable

> 없음
```

**⚠️ 중요**: 섹션 자체는 반드시 포함, "> 없음"으로 표시

---

### 4. Request Body

#### Case 1: Request Body가 있는 경우

```markdown
## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| title | String | 문제 제목 | ✅ |
| content | String | 문제 내용 | ✅ |
| questionType | String | 문제 유형 | ✅ |
| difficulty | String | 난이도 | ✅ |
| score | Integer | 점수 | ✅ |
| timeLimit | Integer | 제한시간 (초) | ⛔ |
```

**변수 설명**:
- `{name}`: 필드 이름 (camelCase)
- `{type}`: API 타입 (String, Integer, Long, Boolean 등)
  - Enum 필드는 String으로 표시
  - LocalDateTime은 String으로 표시
- `{description}`: 설명
- `{required}`: ✅ (필수) 또는 ⛔ (선택)

**⚠️ 리스트 조회 API의 page, limit**:
```markdown
| page | Integer | 페이지 번호 (기본값: 1) | ✅ |
| limit | Integer | 페이지당 개수 (기본값: 10) | ✅ |
```
- 기본값이 있어도 **반드시 ✅ (필수)로 표시**

**잘못된 예**:
```markdown
| page | Integer | 페이지 번호 (기본값: 1) | ⛔ |  # 잘못됨!
| size | Integer | 페이지당 개수 (기본값: 10) | ⛔ |  # 잘못됨!
```

#### Enum 정의 (Request Body 바로 아래)

**⚠️ 중요: Enum 정의는 항상 Request Body 섹션 바로 아래에 위치**

```markdown
### questionType

| Value | Description |
|-------|-------------|
| SUBJECTIVE | 주관식 |
| OBJECTIVE | 객관식 |
| MIXED | 혼합형 |

### difficulty

| Value | Description |
|-------|-------------|
| BASE | 기초 |
| BASIC | 기본 |
| TOP | 상급 |
| TOP_PLUS | 최상급 |
| SEMI_CONTEST | 준경시 |
```

**Enum 정의 규칙**:
- Enum 필드 하나당 하나의 sub-section (###)
- Value는 Enum 상수 이름 (UPPERCASE_SNAKE_CASE)
- Description은 Enum의 description 필드 값

**잘못된 위치**:
```markdown
# ❌ 문서 하단 부록에 위치
## Notes
...

## 부록: Enum 정의
### questionType
...

# ❌ Error Code 이전에 위치
## Error Code
...

### Enum 정의
...
```

#### Request Example

Request Body가 있으면 **반드시** 예시 포함:

```markdown
### Request Example

\`\`\`json
{
  "title": "테스트 문제",
  "content": "문제 내용입니다.",
  "questionType": "SUBJECTIVE",
  "difficulty": "BASIC",
  "score": 100,
  "timeLimit": 300
}
\`\`\`
```

**예시 작성 규칙**:
- Enum 값: 문자열 (예: "SUBJECTIVE")
- 날짜/시간: ISO 8601 형식 (예: "2024-01-01T00:00:00")
- 의미 있는 값 사용 (예: "테스트 문제", 100, 300)

#### Case 2: Request Body가 없는 경우

```markdown
## Request Body

> 없음
```

**⚠️ 중요**:
- 섹션 자체는 반드시 포함
- Request Example은 생략

---

### 5. Success Response

#### Case 1: 단일 객체 응답

```markdown
## Success Response

| name | type | description |
|------|------|-------------|
| questionId | Long | 문제 고유번호 |
| title | String | 문제 제목 |
| content | String | 문제 내용 |
| questionType | String | 문제 유형 |
| difficulty | String | 난이도 |
| score | Integer | 점수 |
| createdAt | String | 생성일시 (ISO 8601) |
| updatedAt | String | 수정일시 (ISO 8601) |

\`\`\`json
{
  "questionId": 123,
  "title": "테스트 문제",
  "content": "문제 내용입니다.",
  "questionType": "SUBJECTIVE",
  "difficulty": "BASIC",
  "score": 100,
  "createdAt": "2025-10-29T10:30:00",
  "updatedAt": "2025-10-29T10:30:00"
}
\`\`\`
```

**변수 설명**:
- `{name}`: 필드 이름 (camelCase)
- `{type}`: API 타입
- `{description}`: 설명

#### Case 2: 배열 응답 (리스트)

```markdown
## Success Response

> Response는 QuestionSimpleVo 배열 형태입니다.

| name | type | description |
|------|------|-------------|
| questionId | Long | 문제 고유번호 |
| title | String | 문제 제목 |
| questionType | String | 문제 유형 |
| createdAt | String | 생성일시 (ISO 8601) |

\`\`\`json
[
  {
    "questionId": 123,
    "title": "문제 1",
    "questionType": "SUBJECTIVE",
    "createdAt": "2025-10-29T10:30:00"
  },
  {
    "questionId": 124,
    "title": "문제 2",
    "questionType": "OBJECTIVE",
    "createdAt": "2025-10-29T10:31:00"
  }
]
\`\`\`
```

**⚠️ 중요**: 첫 줄에 "{Vo클래스명} 배열 형태입니다." 표시

#### Case 3: Count 응답

```markdown
## Success Response

| name | type | description |
|------|------|-------------|
| count | Long | 총 개수 |

\`\`\`json
{
  "count": 42
}
\`\`\`
```

#### Case 4: 204 No Content (DELETE)

```markdown
## Success Response

> HTTP 204 No Content

> 본문 없음
```

---

### 6. Fail Response

**모든 API 공통 포맷**:

```markdown
## Fail Response

\`\`\`json
{
  "timestamp": "2025-10-29T13:05:10",
  "message": "존재하지 않는 문제입니다.",
  "errorCode": 142001
}
\`\`\`
```

**변수 설명**:
- `{timestamp}`: 예시 날짜/시간 (ISO 8601)
- `{message}`: 대표적인 에러 메시지 (실제 ErrorCode의 메시지)
- `{errorCode}`: 대표적인 에러 코드

---

### 7. Example (cURL)

#### GET (단일 조회)

```markdown
## Example (cURL)

\`\`\`bash
curl -X GET https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json"
\`\`\`
```

**⚠️ 중요**: `"Authorization: {JWT_TOKEN}"` (Bearer 제외)

**잘못된 예**:
```bash
curl -X GET https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: Bearer {JWT_TOKEN}" \  # 잘못됨!
  -H "Content-Type: application/json"
```

#### GET (리스트)

```markdown
## Example (cURL)

\`\`\`bash
curl -X GET "https://api.server.com/admin/v1/questions/list?page=1&limit=10&questionType=SUBJECTIVE" \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json"
\`\`\`
```

**쿼리 파라미터**: URL에 포함 (URL encoding 고려)

#### POST

```markdown
## Example (cURL)

\`\`\`bash
curl -X POST https://api.server.com/admin/v1/questions \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "테스트 문제",
    "content": "문제 내용입니다.",
    "questionType": "SUBJECTIVE",
    "difficulty": "BASIC",
    "score": 100
  }'
\`\`\`
```

**-d 옵션**: Request Body를 JSON 형식으로

#### PUT

```markdown
## Example (cURL)

\`\`\`bash
curl -X PUT https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "수정된 문제",
    "score": 150
  }'
\`\`\`
```

#### DELETE

```markdown
## Example (cURL)

\`\`\`bash
curl -X DELETE https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}"
\`\`\`
```

**DELETE는 보통 Content-Type 헤더 불필요**

---

### 8. Notes (선택 사항)

**특별한 주의사항이나 비즈니스 로직 설명이 필요한 경우에만 포함**:

```markdown
## Notes

### Twin Question (변형 문제) 생성 규칙

- Twin Question 생성 시 `originQuestionId`를 반드시 지정해야 합니다.
- Twin Question은 Origin Question과 동일한 `questionGroupId`를 사용해야 합니다.

### 유효성 검증

- `score`는 1 이상 1000 이하의 값이어야 합니다.
- `timeLimit`는 60 이상 3600 이하의 값이어야 합니다.
```

**Notes가 필요 없으면 섹션 자체를 생략**

---

### 9. Error Code (맨 마지막)

**⚠️ 중요: 항상 문서의 맨 마지막 섹션**

```markdown
## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142001 | 404 | QUESTION_NOT_EXIST | 존재하지 않는 문제입니다. |
| 142002 | 400 | QUESTION_ALREADY_EXIST | 이미 존재하는 문제입니다. |
| 142003 | 400 | INVALID_QUESTION_TYPE | 유효하지 않은 문제 유형입니다. |
```

**에러 코드 순서**:
1. 인증/권한 관련 공통 에러 (140xxx)
2. 도메인 특화 에러 (142xxx, 143xxx 등)

**잘못된 위치**:
```markdown
# ❌ Notes 이전
## Error Code
...

## Notes
...

# ❌ Example 이전
## Error Code
...

## Example (cURL)
...
```

---

## 📋 섹션 순서 체크리스트

**⚠️ 이 순서를 절대 변경하지 마세요**:

1. ✅ # {API 이름}
2. ✅ ## 기본 정보
3. ✅ ## Request Header
4. ✅ ## Path Variable
5. ✅ ## Request Body
   - ✅ ### {enumField1}
   - ✅ ### {enumField2}
   - ✅ ### Request Example
6. ✅ ## Success Response
7. ✅ ## Fail Response
8. ✅ ## Example (cURL)
9. ✅ ## Notes (선택)
10. ✅ ## Error Code (맨 마지막)

---

## ⚠️ 절대 규칙 요약

### 1. 섹션 순서
- ❌ 절대 순서를 변경하지 마세요
- ✅ 위의 체크리스트 순서를 정확히 따르세요

### 2. Enum 정의 위치
- ❌ 문서 하단 부록 금지
- ❌ Error Code 이전 금지
- ✅ Request Body 섹션 바로 아래

### 3. Error Code 위치
- ❌ Notes 이전 금지
- ❌ 문서 중간 어디든 금지
- ✅ 항상 문서 맨 마지막

### 4. Authorization 헤더
- ❌ "Bearer {token}" 문구 금지
- ✅ "JWT 인증 토큰" 또는 "JWT 인증 토큰" 만 사용
- ✅ cURL 예시에서도 Bearer 제외

### 5. page, limit 필수값
- ❌ 기본값이 있어도 선택(⛔)으로 표시 금지
- ✅ 항상 필수(✅)로 표시

### 6. Enum 값
- ❌ 추론 금지
- ✅ 실제 Enum 파일에서 읽기

---

**이 템플릿은 강제입니다. 예외는 허용되지 않습니다.**
