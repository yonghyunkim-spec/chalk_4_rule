# Postman 컬렉션 생성 규칙

## 📋 개요

Controller 클래스를 분석하여 Postman v2.1 포맷의 완전한 컬렉션을 자동으로 생성하는 규칙을 정의합니다.

## 🎯 적용 대상

- `/generate-postman` 슬래시 커맨드
- `/sync-postman` 슬래시 커맨드

## 📁 출력 경로 규칙

### 파일 경로 패턴
```
C:\Users\user\Documents\chalk4_api_docs\postman\{package}_APIs.postman_collection.json
```

### 파일명 예시
```
question_APIs.postman_collection.json
board_APIs.postman_collection.json
lecture_APIs.postman_collection.json
```

## 🏷️ 네이밍 규칙

### Request Name 형식

**⚠️ 중요: 이 네이밍 규칙을 정확히 따라야 합니다**

| HTTP Method | 용도 | Postman Request Name 형식 |
|-------------|------|---------------------------|
| GET (단일) | 상세 조회 | `{resource} info` |
| GET (목록) | 리스트 조회 | `{resource} list` |
| GET (카운트) | 리스트 개수 | `{resource} list count` |
| POST | 생성 | `{resource} add` |
| PUT | 수정 | `{resource} modify` |
| DELETE | 삭제 | `{resource} delete` |

### 예시

```
question info          (GET /admin/v1/questions/{id})
question list          (GET /admin/v1/questions/list)
question list count    (GET /admin/v1/questions/list/count)
question add           (POST /admin/v1/questions)
question modify        (PUT /admin/v1/questions/{id})
question delete        (DELETE /admin/v1/questions/{id})
```

### 잘못된 네이밍

```
❌ Get Question
❌ Question Info
❌ getQuestion
❌ Create Question
❌ Update Question
❌ Remove Question
```

## 🔍 Controller 분석 규칙

### 1. Controller 파일 찾기

```
mcp__serena__find_symbol:
- name_path: "{Domain}Controller"
- include_body: true
- depth: 1
```

### 2. Endpoint 정보 추출

각 메서드에서 추출:
- HTTP Method (@GetMapping, @PostMapping, @PutMapping, @DeleteMapping)
- URL 패턴 (@RequestMapping + @*Mapping)
- Request 구조 (@PathVariable, @RequestBody, @ModelAttribute)
- Response 타입

## 📝 Param/VO 분석 규칙

### Request Parameters

SearchParam, AddParam, ModParam 구조 파악:

```
mcp__serena__find_symbol:
- name_path: "{Domain}{ParamType}"
- depth: 1
```

**주의사항**:
- @JsonIgnore 필드 제외 (adminId 등)
- 자동 생성 필드 제외 (createdAt, updatedAt 등)

### Response VOs

{Domain}Vo, {Domain}SimpleVo 구조 파악

## 🚨 Enum 처리 규칙

### 절대 규칙

❌ **절대 Enum 값을 추론하지 마세요**
✅ **항상 실제 Enum 파일을 읽어야 합니다**
✅ **Enum 값은 문자열로 표현합니다**

### Enum 파일 읽기

```
mcp__serena__find_symbol:
- name_path: "{EnumType}"
- relative_path: "src/main/java/com/firsthabit/chalk/{package}/constants"
- include_body: true
```

### Enum 값 사용 규칙

```java
// Enum 파일
@Getter
public enum OpenFlag {
    CLOSE(0, "비공개"),
    OPEN(1, "공개");

    private final int type;
    private final String description;
}
```

Postman Request Body:
```json
{
  "openFlag": "OPEN"  // ✅ 문자열 사용
}
```

**잘못된 사용**:
```json
{
  "openFlag": 1  // ❌ ordinal 숫자 사용 금지!
}
```

## 🌐 환경 변수 규칙

### 필수 환경 변수

```json
"variable": [
  {
    "key": "server url",
    "value": "http://localhost:8080",
    "type": "string"
  },
  {
    "key": "jwt token",
    "value": "",
    "type": "string"
  }
]
```

**변수 사용**:
- URL: `{{server url}}/admin/v1/questions`
- Authorization: `{{jwt token}}`

## 🔐 인증 설정 규칙

### Header-based Auth

**⚠️ 중요**:
- Collection level에는 auth 설정 **없음** (제거)
- 각 request의 header에 Authorization 추가
- 변수 `{{jwt token}}` 사용으로 중앙 관리

```json
// Collection level - auth 없음, variable만 설정
"variable": [
  {
    "key": "server url",
    "value": "http://localhost:8080",
    "type": "string"
  },
  {
    "key": "jwt token",
    "value": "",
    "type": "string"
  }
]
```

```json
// 각 Request의 header에 추가
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

**⚠️ 중요**: `Bearer` 키워드는 **포함하지 않습니다**. Postman에서 토큰 값에 자동으로 추가됩니다.

## 📄 Request Body 생성 규칙

### 1. 제외 필드

**항상 제외**:
- adminId (@JsonIgnore 필드)
- createdAt (자동 생성)
- updatedAt (자동 생성)
- createdById (자동 생성)
- updatedById (자동 생성)

### 2. 페이징 필드

**리스트 조회 API**:
```json
{
  "page": 1,    // 1부터 시작
  "limit": 10   // 기본값 10
}
```

**⚠️ 중요**: page는 1부터 시작 (0 아님)

### 3. 예시 값 생성 규칙

| 타입 | 예시 값 | 설명 |
|------|--------|------|
| String | "Sample Text", "테스트" | 의미 있는 값 |
| Integer/Long | 1, 100, 1000 | 명확한 값 |
| Boolean | true, false | - |
| Enum | "ENUM_VALUE" | 문자열 (대문자) |
| LocalDateTime | "2024-01-01T00:00:00" | ISO 8601 |
| LocalDate | "2024-01-01" | YYYY-MM-DD |
| BigDecimal | "10.5", "100.00" | 문자열 형태 |

### 4. JSON 주석 형식

**⚠️ 중요: 각 필드에 주석 추가**

```json
{
  "title": "테스트 문제", // 문제 제목
  "content": "문제 내용", // 문제 내용 (HTML 가능)
  "questionType": "SUBJECTIVE", // 문제 유형 (SUBJECTIVE=주관식,OBJECTIVE=객관식,MIXED=혼합형)
  "difficulty": "BASIC", // 난이도 (BASE=기초,BASIC=기본,TOP=상급,TOP_PLUS=최상급,SEMI_CONTEST=준경시)
  "score": 100, // 점수 (최소:1,최대:1000)
  "timeLimit": 300, // 제한시간 (초 단위)
  "isVisible": true, // 공개 여부 (true=공개,false=비공개)
  "tags": ["수학", "정수론"], // 태그 목록
  "publishDate": "2024-01-01T00:00:00" // 공개일시 (ISO 8601)
}
```

**주석 작성 규칙**:
- 일반 필드: `// 필드 설명`
- Enum 필드: `// 필드 설명 (VALUE1=설명1,VALUE2=설명2,VALUE3=설명3)`
- Boolean 필드: `// 필드 설명 (true=설명,false=설명)`
- 숫자 필드: `// 필드 설명 (최소:N,최대:M)`
- 날짜 필드: `// 필드 설명 (ISO 8601)`

## 📋 Request 템플릿

### GET (단일 조회)

```json
{
  "name": "{domain} info",
  "request": {
    "method": "GET",
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
    "url": {
      "raw": "{{server url}}/admin/v1/{domains}/{{{pkField}}}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "{domains}", "{{{pkField}}}"]
    },
    "description": "{Domain} 단일 조회 API"
  },
  "response": []
}
```

### GET (리스트)

```json
{
  "name": "{domain} list",
  "request": {
    "method": "GET",
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
    "url": {
      "raw": "{{server url}}/admin/v1/{domains}/list?page=1&limit=10",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "{domains}", "list"],
      "query": [
        {
          "key": "page",
          "value": "1",
          "description": "페이지 번호 (1부터 시작)"
        },
        {
          "key": "limit",
          "value": "10",
          "description": "페이지당 개수"
        },
        {
          "key": "{enumField}",
          "value": "{ENUM_VALUE}",
          "description": "{enumField} 필터",
          "disabled": true
        },
        {
          "key": "searchValue",
          "value": "",
          "description": "검색어",
          "disabled": true
        }
      ]
    },
    "description": "{Domain} 목록 조회 API (페이징)"
  },
  "response": []
}
```

**Query Parameter 규칙**:
- page, limit: enabled (항상 활성)
- 필터 파라미터: disabled (선택적으로 활성화)

### POST (추가)

```json
{
  "name": "{domain} add",
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
      "raw": "{\n  \"field1\": \"value1\", // 필드1 설명\n  \"field2\": 100, // 필드2 설명 (숫자형)\n  \"enumField\": \"ENUM_VALUE\" // enum 설명 (ENUM1=설명1,ENUM2=설명2)\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/{domains}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "{domains}"]
    },
    "description": "{Domain} 추가 API"
  },
  "response": []
}
```

**Body 생성 규칙**:
- mode: "raw"
- raw: JSON 문자열 (개행 포함, `\n` 사용)
- 주석 포함
- language: "json"

### PUT (수정)

```json
{
  "name": "{domain} modify",
  "request": {
    "method": "PUT",
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
      "raw": "{\n  \"field1\": \"updated value1\", // 수정할 필드1 설명\n  \"field2\": 200 // 수정할 필드2 설명\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/{domains}/{{{pkField}}}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "{domains}", "{{{pkField}}}"]
    },
    "description": "{Domain} 수정 API"
  },
  "response": []
}
```

### DELETE (삭제)

```json
{
  "name": "{domain} delete",
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
      "raw": "{{server url}}/admin/v1/{domains}/{{{pkField}}}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "{domains}", "{{{pkField}}}"]
    },
    "description": "{Domain} 삭제 API"
  },
  "response": []
}
```

**DELETE도 Authorization 헤더 필요**

## 📋 검증 체크리스트

### 필수 확인 사항
- [ ] info.name이 "{Domain} APIs" 형식인가?
- [ ] Collection level에 auth 설정이 **없는가**? (제거되어야 함)
- [ ] 모든 request의 header에 `Authorization: {{jwt token}}`이 있는가? (Bearer 없이)
- [ ] 모든 request의 name이 네이밍 규칙을 따르는가?
- [ ] 모든 request의 url.host가 `["{{server url}}"]`인가?
- [ ] POST/PUT request의 body가 실제로 사용 가능한가?
- [ ] GET list request에 page=1, limit=10이 포함되었는가?
- [ ] disabled: true인 query parameter가 적절히 설정되었는가?

### Enum 관련 확인
- [ ] 모든 Enum 필드에 대해 실제 Enum 파일을 읽었는가? ⚠️
- [ ] Enum 값을 추론하지 않았는가? ⚠️
- [ ] Enum 값을 문자열로 표현했는가? ⚠️

### adminId 확인
- [ ] Request Body에서 adminId를 제외했는가? ⚠️

### page 확인
- [ ] page가 1부터 시작하는가? ⚠️

### JSON 주석 확인
- [ ] 모든 필드에 주석이 있는가?
- [ ] Enum 필드 주석에 모든 값이 설명되어 있는가?

## ⚠️ 자주 발생하는 실수

### 1. Enum 값 표현
```json
// ❌ 잘못된 예 (ordinal 숫자)
{
  "questionType": 0
}

// ✅ 올바른 예 (Enum 이름)
{
  "questionType": "SUBJECTIVE"
}
```

### 2. 페이징 파라미터
```json
// ❌ page를 0부터 시작
{
  "page": 0,
  "limit": 10
}

// ✅ page는 1부터 시작
{
  "page": 1,
  "limit": 10
}
```

### 3. adminId 필드
```json
// ❌ adminId 포함
{
  "adminId": 1,
  "title": "Test"
}

// ✅ adminId 제외
{
  "title": "Test"
}
```

### 4. 환경 변수 사용
```json
// ❌ 하드코딩
"url": "http://localhost:8080/admin/v1/questions"

// ✅ 변수 사용
"url": "{{server url}}/admin/v1/questions"
```

### 5. Request 이름
```json
// ❌ 자유로운 이름
"name": "Get Question"
"name": "Create Question"

// ✅ 네이밍 규칙 준수
"name": "question info"
"name": "question add"
```

## 🔗 참조 문서

- **포맷 규격**: `rule/postman/postman_format_spec.md`
- **예시**: `rule/postman/postman_examples.md`
- **슬래시 커맨드**: `.claude/commands/generate-postman.md`

---

**중요**: 이 규칙을 **100% 준수**해야 합니다. 예외는 허용되지 않습니다.
