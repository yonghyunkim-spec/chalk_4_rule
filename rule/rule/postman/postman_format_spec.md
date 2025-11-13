# Postman Collection v2.1 포맷 규격

## 📋 개요

이 문서는 Postman Collection v2.1 포맷의 완전한 JSON 스키마와 각 필드의 명세를 정의합니다.

## 🎯 공식 스키마

```
https://schema.postman.com/json/collection/v2.1.0/collection.json
```

## 📐 전체 구조

```json
{
  "info": { /* 컬렉션 메타데이터 */ },
  "auth": { /* 컬렉션 레벨 인증 */ },
  "variable": [ /* 컬렉션 변수 */ ],
  "item": [ /* 요청 항목 배열 */ ]
}
```

## 1. info 객체 (필수)

컬렉션의 기본 정보를 정의합니다.

### 스키마

```json
{
  "name": "string (required)",
  "_postman_id": "string (optional)",
  "description": "string (optional)",
  "schema": "string (required)"
}
```

### 필드 명세

| 필드 | 타입 | 필수 | 설명 | 값 규칙 |
|------|------|------|------|---------|
| name | string | ✅ | 컬렉션 이름 | `"{Domain} APIs"` 형식 |
| _postman_id | string | ❌ | Postman 고유 ID | UUID 형식 권장 |
| description | string | ❌ | 컬렉션 설명 | 한글 설명 가능 |
| schema | string | ✅ | 스키마 버전 URL | `"https://schema.postman.com/json/collection/v2.1.0/collection.json"` |

### 예시

```json
{
  "info": {
    "name": "Question APIs",
    "_postman_id": "question-apis-collection-001",
    "description": "Question 도메인의 CRUD API 컬렉션",
    "schema": "https://schema.postman.com/json/collection/v2.1.0/collection.json"
  }
}
```

### 네이밍 규칙

```
✅ "Question APIs"
✅ "Board APIs"
✅ "Lecture APIs"

❌ "question_apis"
❌ "Question API"
❌ "Questions"
```

## 2. auth 객체 (선택) - ⚠️ 사용 안 함

**중요**: 이 프로젝트에서는 Collection level auth를 **사용하지 않습니다**.
대신 각 request의 header에 직접 Authorization을 추가합니다.

### 이전 방식 (사용 안 함)

~~Collection level에서 Bearer Token 설정~~

```json
// ❌ 사용하지 않음
{
  "type": "bearer",
  "bearer": [
    {
      "key": "token",
      "value": "{{jwt token}}",
      "type": "string"
    }
  ]
}
```

### 새로운 방식 (Header-based Auth)

각 request의 header 배열에 Authorization 추가:

```json
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
]
```

**⚠️ 중요**:
- `Bearer` 키워드는 **포함하지 않습니다**
- Postman에서 토큰 입력 시 Bearer를 포함한 전체 토큰 문자열을 입력해야 합니다
- 예: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

**장점**:
- Headers 탭에서 바로 확인 가능
- 각 request별로 쉽게 활성화/비활성화 가능
- 변수 `{{jwt token}}`를 통해 여전히 중앙 관리

### 지원 인증 타입

| 타입 | 사용 여부 | 비고 |
|------|-----------|------|
| bearer | ✅ 사용 | JWT 토큰 인증 (기본) |
| basic | ❌ 미사용 | - |
| apikey | ❌ 미사용 | - |
| oauth2 | ❌ 미사용 | - |
| noauth | ❌ 미사용 | 인증 없음 |

### 예시

```json
{
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ]
  }
}
```

## 3. variable 배열 (필수)

컬렉션 전역에서 사용 가능한 변수들을 정의합니다.

### 스키마

```json
{
  "variable": [
    {
      "key": "string (required)",
      "value": "string (required)",
      "type": "string (required)"
    }
  ]
}
```

### 필드 명세

| 필드 | 타입 | 필수 | 설명 | 값 규칙 |
|------|------|------|------|---------|
| key | string | ✅ | 변수 이름 | 공백 포함 가능 |
| value | string | ✅ | 변수 기본값 | 빈 문자열 가능 |
| type | string | ✅ | 값 타입 | `"string"` 고정 |

### 필수 변수

**⚠️ 중요: 다음 2개 변수는 필수입니다**

```json
{
  "variable": [
    {
      "key": "server url",
      "value": "http://localhost:38084",
      "type": "string"
    },
    {
      "key": "jwt token",
      "value": "",
      "type": "string"
    }
  ]
}
```

### 변수 사용 규칙

**변수 참조 문법**: `{{variable_key}}`

```json
// URL에서 사용
"url": "{{server url}}/admin/v1/questions"

// Authorization에서 사용
"value": "{{jwt token}}"

// 경로에서 사용
"host": ["{{server url}}"]
```

**주의사항**:
- 중괄호 2개 사용: `{{key}}`
- 공백이 있는 키도 가능: `{{server url}}`
- 대소문자 구분: `{{JWT Token}}` ≠ `{{jwt token}}`

## 4. item 배열 (필수)

실제 API 요청들을 정의하는 배열입니다.

### 스키마

```json
{
  "item": [
    {
      "name": "string (required)",
      "request": { /* request 객체 */ },
      "response": [ /* response 배열 */ ]
    }
  ]
}
```

### Item 객체 필드 명세

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | string | ✅ | 요청 이름 (네이밍 규칙 준수) |
| request | object | ✅ | 요청 정의 객체 |
| response | array | ✅ | 응답 예시 배열 (빈 배열 가능) |

### Request 이름 네이밍 규칙

**⚠️ 중요: 정확히 준수해야 합니다**

| HTTP Method | 용도 | 형식 | 예시 |
|-------------|------|------|------|
| GET (단일) | 상세 조회 | `{domain} info` | `question info` |
| GET (목록) | 리스트 조회 | `{domain} list` | `question list` |
| GET (카운트) | 개수 조회 | `{domain} list count` | `question list count` |
| POST | 생성 | `{domain} add` | `question add` |
| PUT | 수정 | `{domain} modify` | `question modify` |
| DELETE | 삭제 | `{domain} delete` | `question delete` |
| POST (벌크) | 목록 추가 | `{domain1}-{domain2} list add` | `question-chapter list add` |

**형식 규칙**:
- 소문자만 사용
- 공백으로 구분
- 동사는 고정 (info, list, add, modify, delete)

## 5. request 객체 (필수)

개별 API 요청의 상세 정보를 정의합니다.

### 스키마

```json
{
  "request": {
    "method": "string (required)",
    "header": [ /* header 배열 */ ],
    "body": { /* body 객체 (POST/PUT만) */ },
    "url": { /* url 객체 */ },
    "description": "string (optional)"
  }
}
```

### 필드 명세

| 필드 | 타입 | 필수 | 설명 | 적용 Method |
|------|------|------|------|-------------|
| method | string | ✅ | HTTP Method | 모두 |
| header | array | ✅ | 요청 헤더 배열 | GET, POST, PUT (DELETE는 빈 배열) |
| body | object | ❌ | 요청 본문 | POST, PUT만 |
| url | object | ✅ | URL 정보 객체 | 모두 |
| description | string | ❌ | 요청 설명 | 모두 (권장) |

### 5.1 method 필드

**허용 값**:
- `"GET"`
- `"POST"`
- `"PUT"`
- `"DELETE"`
- `"PATCH"` (사용 안 함)

**규칙**:
- 대문자만 사용
- 표준 HTTP Method만 사용

### 5.2 header 배열

#### 스키마

```json
{
  "header": [
    {
      "key": "string",
      "value": "string"
    }
  ]
}
```

#### Method별 Header 규칙

| Method | Content-Type 필요 여부 |
|--------|----------------------|
| GET | ✅ 필요 |
| POST | ✅ 필요 |
| PUT | ✅ 필요 |
| DELETE | ❌ 빈 배열 `[]` |

#### 예시

```json
// GET, POST, PUT
"header": [
  {
    "key": "Content-Type",
    "value": "application/json"
  }
]

// DELETE
"header": []
```

### 5.3 body 객체 (POST/PUT만)

#### 스키마

```json
{
  "body": {
    "mode": "raw",
    "raw": "string (JSON 문자열)",
    "options": {
      "raw": {
        "language": "json"
      }
    }
  }
}
```

#### 필드 명세

| 필드 | 타입 | 필수 | 값 |
|------|------|------|-----|
| mode | string | ✅ | `"raw"` 고정 |
| raw | string | ✅ | JSON 문자열 (개행 `\n` 포함) |
| options.raw.language | string | ✅ | `"json"` 고정 |

#### raw 필드 작성 규칙

**⚠️ 중요: JSON 주석 포함 필수**

```json
{
  "body": {
    "mode": "raw",
    "raw": "{\n  \"field1\": \"value1\", // 필드1 설명\n  \"enumField\": \"ENUM_VALUE\", // Enum 설명 (VALUE1=설명1,VALUE2=설명2)\n  \"page\": 1, // 페이지 번호 (1부터 시작)\n  \"size\": 10 // 페이지당 개수\n}",
    "options": {
      "raw": {
        "language": "json"
      }
    }
  }
}
```

**주석 작성 규칙**:
1. 일반 필드: `// 필드 설명`
2. Enum 필드: `// 필드 설명 (VALUE1=설명1,VALUE2=설명2)`
3. Boolean 필드: `// 필드 설명 (true=참,false=거짓)`
4. 숫자 범위: `// 필드 설명 (최소:N,최대:M)`
5. 날짜: `// 필드 설명 (ISO 8601)`

**개행 표현**:
- JSON 문자열 내부에서 `\n` 사용
- 들여쓰기: 공백 2개

#### 제외 필드 규칙

**❌ 절대 포함하지 않을 필드**:
- `adminId` (@JsonIgnore 필드)
- `createdAt` (자동 생성)
- `updatedAt` (자동 생성)
- `createdById` (자동 생성)
- `updatedById` (자동 생성)

### 5.4 url 객체

#### 스키마

```json
{
  "url": {
    "raw": "string (required)",
    "host": ["string"],
    "path": ["string", "string", ...],
    "query": [ /* query 배열 (GET list만) */ ]
  }
}
```

#### 필드 명세

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| raw | string | ✅ | 완전한 URL 문자열 |
| host | array | ✅ | 호스트 부분 (변수 사용) |
| path | array | ✅ | 경로 세그먼트 배열 |
| query | array | ❌ | 쿼리 파라미터 배열 (GET list만) |

#### raw 필드 규칙

**변수 사용 필수**: `{{server url}}`

```
✅ "{{server url}}/admin/v1/questions"
✅ "{{server url}}/admin/v1/questions/{questionId}"
✅ "{{server url}}/admin/v1/questions/list?page=1&size=20"

❌ "http://localhost:8080/admin/v1/questions"
❌ "localhost:8080/admin/v1/questions"
```

#### host 배열 규칙

**항상 단일 요소 배열**:

```json
"host": ["{{server url}}"]
```

#### path 배열 규칙

**경로를 `/`로 분리하여 배열화**:

```json
// URL: /admin/v1/questions
"path": ["admin", "v1", "questions"]

// URL: /admin/v1/questions/{questionId}
"path": ["admin", "v1", "questions", "{questionId}"]

// URL: /admin/v1/questions/list
"path": ["admin", "v1", "questions", "list"]
```

**Path Variable 표현**:
- 중괄호 사용: `"{questionId}"`
- 변수명은 camelCase

#### query 배열 (GET list만)

##### 스키마

```json
{
  "query": [
    {
      "key": "string",
      "value": "string",
      "description": "string",
      "disabled": boolean
    }
  ]
}
```

##### 필드 명세

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| key | string | ✅ | 쿼리 파라미터 이름 |
| value | string | ✅ | 기본값 |
| description | string | ✅ | 파라미터 설명 |
| disabled | boolean | ❌ | 비활성화 여부 (선택 파라미터만) |

##### 필수 쿼리 파라미터 (page, size)

**⚠️ 중요: page는 1부터 시작**

```json
{
  "query": [
    {
      "key": "page",
      "value": "1",
      "description": "페이지 번호 (1부터 시작)"
    },
    {
      "key": "size",
      "value": "10",
      "description": "페이지당 개수"
    }
  ]
}
```

**잘못된 예**:
```json
{
  "key": "page",
  "value": "0",  // ❌ 0부터 시작 금지!
  "description": "페이지 번호 (0부터 시작)"  // ❌ 잘못된 설명
}
```

##### 선택 쿼리 파라미터

**disabled: true 설정 필수**

```json
{
  "query": [
    {
      "key": "searchValue",
      "value": "",
      "description": "검색어",
      "disabled": true
    },
    {
      "key": "openFlag",
      "value": "OPEN",
      "description": "오픈 여부 (OPEN, CLOSE)",
      "disabled": true
    }
  ]
}
```

**규칙**:
- page, size: `disabled` 없음 (항상 활성)
- 필터, 검색 파라미터: `disabled: true`

### 5.5 description 필드

**권장 형식**:

```
"{Domain} {동작} API"
```

**예시**:
```json
"description": "Question 단일 조회 API"
"description": "Question 목록 조회 API (페이징)"
"description": "Question 추가 API"
"description": "Question 수정 API"
"description": "Question 삭제 API"
```

## 6. response 배열

현재는 빈 배열로 사용합니다.

```json
"response": []
```

**향후 확장 가능**:
- 응답 예시 추가
- 성공/실패 케이스 샘플

## 📋 전체 예시 (완전한 컬렉션)

```json
{
  "info": {
    "name": "Question APIs",
    "_postman_id": "question-apis-collection-001",
    "description": "Question 도메인의 CRUD API 컬렉션",
    "schema": "https://schema.postman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{jwt token}}",
        "type": "string"
      }
    ]
  },
  "variable": [
    {
      "key": "server url",
      "value": "http://localhost:38084",
      "type": "string"
    },
    {
      "key": "jwt token",
      "value": "",
      "type": "string"
    }
  ],
  "item": [
    {
      "name": "question info",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "url": {
          "raw": "{{server url}}/admin/v1/questions/{questionId}",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions", "{questionId}"]
        },
        "description": "Question 단일 조회 API"
      },
      "response": []
    },
    {
      "name": "question list",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "url": {
          "raw": "{{server url}}/admin/v1/questions/list?page=1&size=10",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions", "list"],
          "query": [
            {
              "key": "page",
              "value": "1",
              "description": "페이지 번호 (1부터 시작)"
            },
            {
              "key": "size",
              "value": "10",
              "description": "페이지당 개수"
            },
            {
              "key": "searchValue",
              "value": "",
              "description": "검색어",
              "disabled": true
            }
          ]
        },
        "description": "Question 목록 조회 API (페이징)"
      },
      "response": []
    },
    {
      "name": "question add",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"questionGroupId\": 10, // 문제 그룹 ID\n  \"content\": \"문제 내용\", // 문제 내용 (HTML 가능)\n  \"answer\": \"{}\", // 정답 (JSON 형식)\n  \"openFlag\": \"OPEN\" // 공개 여부 (OPEN=공개,CLOSE=비공개)\n}",
          "options": {
            "raw": {
              "language": "json"
            }
          }
        },
        "url": {
          "raw": "{{server url}}/admin/v1/questions",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions"]
        },
        "description": "Question 추가 API"
      },
      "response": []
    }
  ]
}
```

## ✅ 검증 체크리스트

### 컬렉션 레벨
- [ ] info.name이 "{Domain} APIs" 형식인가?
- [ ] info.schema가 v2.1.0 URL인가?
- [ ] auth.type이 "bearer"인가?
- [ ] auth.bearer[0].value가 "{{jwt token}}"인가?
- [ ] variable에 "server url"과 "jwt token"이 있는가?

### 요청 레벨
- [ ] 모든 item[].name이 네이밍 규칙을 따르는가?
- [ ] 모든 request.url.host가 `["{{server url}}"]`인가?
- [ ] 모든 request.url.raw에 `{{server url}}`이 사용되었는가?
- [ ] GET/POST/PUT은 header에 Content-Type이 있는가?
- [ ] DELETE는 header가 빈 배열인가?

### Body 관련 (POST/PUT)
- [ ] body.mode가 "raw"인가?
- [ ] body.raw에 주석이 포함되었는가?
- [ ] body.options.raw.language가 "json"인가?
- [ ] adminId가 제외되었는가?
- [ ] Enum 값이 문자열로 표현되었는가?

### Query Parameter 관련 (GET list)
- [ ] page 값이 "1"인가? (⚠️ "0" 아님)
- [ ] page description이 "1부터 시작"인가?
- [ ] size가 포함되었는가?
- [ ] 선택 파라미터에 disabled: true가 있는가?

## 🚨 자주 발생하는 실수

### 1. page 시작 번호
```json
// ❌ 잘못된 예
{
  "key": "page",
  "value": "0",
  "description": "페이지 번호 (0부터 시작)"
}

// ✅ 올바른 예
{
  "key": "page",
  "value": "1",
  "description": "페이지 번호 (1부터 시작)"
}
```

### 2. host 배열 형식
```json
// ❌ 잘못된 예
"host": "{{server url}}"
"host": "localhost:8080"

// ✅ 올바른 예
"host": ["{{server url}}"]
```

### 3. Enum 값 표현
```json
// ❌ 잘못된 예
{
  "openFlag": 1
}

// ✅ 올바른 예
{
  "openFlag": "OPEN"
}
```

### 4. adminId 포함
```json
// ❌ 잘못된 예
{
  "adminId": 1,
  "title": "Test"
}

// ✅ 올바른 예
{
  "title": "Test"
}
```

### 5. body.raw 개행 처리
```json
// ❌ 잘못된 예 (실제 개행)
"raw": "{
  \"field\": \"value\"
}"

// ✅ 올바른 예 (\n 사용)
"raw": "{\n  \"field\": \"value\"\n}"
```

## 📖 참조 문서

- **생성 규칙**: `rule/postman/postman_generation_rules.md`
- **예시**: `rule/postman/postman_examples.md`
- **공식 스키마**: https://schema.postman.com/json/collection/v2.1.0/collection.json

---

**작성일**: 2025-11-08
**버전**: 1.0
