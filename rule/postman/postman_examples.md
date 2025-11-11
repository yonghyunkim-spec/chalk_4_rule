# Postman 컬렉션 예시

## 📋 개요

실제 Postman Collection 예시와 올바른/잘못된 패턴을 제시합니다.

## 🎯 참조 파일

실제 생성된 컬렉션:
```
/Users/kyle/source/00.document/admin_docs/postman/question_APIs.postman_collection.json
```

## 📦 1. 완전한 컬렉션 예시

### Question APIs (축약 버전)

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
      "value": "http://localhost:8080",
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
          "raw": "{{server url}}/admin/v1/questions/list?page=1&limit=10",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions", "list"],
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
          "raw": "{\n  \"questionGroupId\": 10, // 문제 그룹 ID\n  \"questionPropertyId\": 50, // 문제 속성 ID\n  \"content\": \"다음 중 정답은?\", // 문제 내용 (HTML 가능)\n  \"answer\": \"{\\\"correctAnswer\\\": \\\"A\\\", \\\"options\\\": [\\\"A\\\", \\\"B\\\", \\\"C\\\", \\\"D\\\"]}\", // 정답 (JSON 형식)\n  \"openFlag\": \"OPEN\" // 공개 여부 (OPEN=공개,CLOSE=비공개)\n}",
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
    },
    {
      "name": "question modify",
      "request": {
        "method": "PUT",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"content\": \"다음 중 정답은? (수정)\", // 수정할 문제 내용\n  \"answer\": \"{\\\"correctAnswer\\\": \\\"B\\\", \\\"options\\\": [\\\"A\\\", \\\"B\\\", \\\"C\\\", \\\"D\\\"]}\", // 수정할 정답\n  \"openFlag\": \"CLOSE\" // 수정할 공개 여부\n}",
          "options": {
            "raw": {
              "language": "json"
            }
          }
        },
        "url": {
          "raw": "{{server url}}/admin/v1/questions/{questionId}",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions", "{questionId}"]
        },
        "description": "Question 수정 API"
      },
      "response": []
    },
    {
      "name": "question delete",
      "request": {
        "method": "DELETE",
        "header": [],
        "url": {
          "raw": "{{server url}}/admin/v1/questions/{questionId}",
          "host": ["{{server url}}"],
          "path": ["admin", "v1", "questions", "{questionId}"]
        },
        "description": "Question 삭제 API"
      },
      "response": []
    }
  ]
}
```

## 📝 2. HTTP Method별 상세 예시

### 2.1 GET (단일 조회) - Path Variable

```json
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
}
```

**포인트**:
- `{questionId}` 형태로 path variable 표현
- header에 Content-Type 포함
- url.host는 배열 형태
- description은 한글로 명확하게

### 2.2 GET (목록 조회) - Query Parameters

```json
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
      "raw": "{{server url}}/admin/v1/questions/list?page=1&limit=10",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "list"],
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
          "key": "questionGroupId",
          "value": "1",
          "description": "문제 그룹 ID 필터",
          "disabled": true
        },
        {
          "key": "openFlag",
          "value": "OPEN",
          "description": "오픈 여부 (OPEN, CLOSE)",
          "disabled": true
        },
        {
          "key": "searchValue",
          "value": "",
          "description": "검색어",
          "disabled": true
        },
        {
          "key": "searchType",
          "value": "CONTENT",
          "description": "검색 타입 (CONTENT, ANSWER)",
          "disabled": true
        }
      ]
    },
    "description": "Question 목록 조회 API (페이징)"
  },
  "response": []
}
```

**포인트**:
- ✅ **page는 "1"부터 시작** (⚠️ 중요)
- limit는 기본값 "10"
- 필수 파라미터 (page, limit): disabled 없음
- 선택 파라미터: disabled: true
- Enum 값은 문자열 ("OPEN", "CONTENT")

### 2.3 GET (카운트) - Query Parameters (page/limit 없음)

```json
{
  "name": "question list count",
  "request": {
    "method": "GET",
    "header": [
      {
        "key": "Content-Type",
        "value": "application/json"
      }
    ],
    "url": {
      "raw": "{{server url}}/admin/v1/questions/list/count",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "list", "count"],
      "query": [
        {
          "key": "questionGroupId",
          "value": "1",
          "description": "문제 그룹 ID 필터",
          "disabled": true
        },
        {
          "key": "openFlag",
          "value": "OPEN",
          "description": "오픈 여부 (OPEN, CLOSE)",
          "disabled": true
        }
      ]
    },
    "description": "Question 목록 개수 조회 API"
  },
  "response": []
}
```

**포인트**:
- page/limit 없음 (count API는 페이징 불필요)
- 필터 파라미터만 포함
- 모든 파라미터 disabled: true

### 2.4 POST (생성) - Request Body

```json
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
      "raw": "{\n  \"questionGroupId\": 10, // 문제 그룹 ID\n  \"questionPropertyId\": 50, // 문제 속성 ID\n  \"content\": \"다음 중 정답은?\", // 문제 내용 (HTML 가능)\n  \"answer\": \"{\\\"correctAnswer\\\": \\\"A\\\", \\\"options\\\": [\\\"A\\\", \\\"B\\\", \\\"C\\\", \\\"D\\\"]}\", // 정답 (JSON 형식)\n  \"openFlag\": \"OPEN\" // 공개 여부 (OPEN=공개,CLOSE=비공개)\n}",
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
```

**포인트**:
- body.mode는 "raw"
- body.raw은 JSON 문자열 (개행은 `\n`)
- 모든 필드에 주석 포함
- Enum 필드는 값과 설명 함께 (`OPEN=공개,CLOSE=비공개`)
- JSON 내부의 따옴표는 이스케이프 (`\"`)
- body.options.raw.language는 "json"
- **adminId 제외**

### 2.5 PUT (수정) - Path Variable + Request Body

```json
{
  "name": "question modify",
  "request": {
    "method": "PUT",
    "header": [
      {
        "key": "Content-Type",
        "value": "application/json"
      }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"content\": \"다음 중 정답은? (수정)\", // 수정할 문제 내용\n  \"answer\": \"{\\\"correctAnswer\\\": \\\"B\\\", \\\"options\\\": [\\\"A\\\", \\\"B\\\", \\\"C\\\", \\\"D\\\"]}\", // 수정할 정답\n  \"openFlag\": \"CLOSE\" // 수정할 공개 여부 (OPEN=공개,CLOSE=비공개)\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/questions/{questionId}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "{questionId}"]
    },
    "description": "Question 수정 API"
  },
  "response": []
}
```

**포인트**:
- POST와 유사하지만 path variable 포함
- 주석에 "수정할" 명시
- 선택적 필드만 포함 (필수 필드는 생략 가능)

### 2.6 DELETE - Path Variable Only

```json
{
  "name": "question delete",
  "request": {
    "method": "DELETE",
    "header": [],
    "url": {
      "raw": "{{server url}}/admin/v1/questions/{questionId}",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "questions", "{questionId}"]
    },
    "description": "Question 삭제 API"
  },
  "response": []
}
```

**포인트**:
- header는 빈 배열 `[]`
- body 없음
- url에 path variable만

### 2.7 POST (벌크 추가) - Array Body

```json
{
  "name": "question-chapter list add",
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
      "raw": "{\n  \"questionId\": 1, // 문제 ID\n  \"chapterIds\": [10, 20, 30] // 챕터 ID 목록 (복수 추가)\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{server url}}/admin/v1/question-chapters/list",
      "host": ["{{server url}}"],
      "path": ["admin", "v1", "question-chapters", "list"]
    },
    "description": "문제-챕터 벌크 추가 API"
  },
  "response": []
}
```

**포인트**:
- name은 `{domain1}-{domain2} list add` 형식
- 배열 필드는 `[10, 20, 30]` 형태
- 주석에 "복수 추가" 명시

## 🎯 3. 공통 패턴

### 3.1 페이징 패턴

**모든 리스트 조회 API는 동일한 페이징 사용**:

```json
{
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
    }
  ]
}
```

**변형 금지**:
- page 기본값: 항상 "1"
- limit 기본값: 항상 "10"
- description: 정확히 위와 동일

### 3.2 Enum 값 패턴

**Enum은 항상 문자열로, 주석에 모든 값 설명**:

```json
{
  "openFlag": "OPEN", // 공개 여부 (OPEN=공개,CLOSE=비공개)
  "questionType": "SUBJECTIVE", // 문제 유형 (SUBJECTIVE=주관식,OBJECTIVE=객관식,MIXED=혼합형)
  "difficulty": "BASIC" // 난이도 (BASE=기초,BASIC=기본,TOP=상급,TOP_PLUS=최상급)
}
```

**Enum 주석 형식**:
```
// 필드 설명 (VALUE1=설명1,VALUE2=설명2,VALUE3=설명3)
```

### 3.3 변수 사용 패턴

**컬렉션 변수**:
```json
{
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
}
```

**변수 참조**:
```json
{
  "url": {
    "raw": "{{server url}}/admin/v1/questions",
    "host": ["{{server url}}"]
  },
  "auth": {
    "bearer": [
      {
        "value": "{{jwt token}}"
      }
    ]
  }
}
```

### 3.4 인증 패턴

**컬렉션 레벨에서 설정**:

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

**개별 request는 상속**:
- 각 request에 auth 명시 불필요
- 컬렉션 레벨 설정이 자동 적용

## ❌ 4. 안티패턴 (잘못된 예시)

### 4.1 ❌ Page 번호 0부터 시작 (가장 흔한 실수)

**❌ 잘못된 예**:
```json
{
  "query": [
    {
      "key": "page",
      "value": "0",
      "description": "페이지 번호 (0부터 시작)"
    }
  ]
}
```

**✅ 올바른 예**:
```json
{
  "query": [
    {
      "key": "page",
      "value": "1",
      "description": "페이지 번호 (1부터 시작)"
    }
  ]
}
```

**왜 중요한가**:
- 백엔드 API는 page=1부터 시작하도록 구현됨
- page=0은 잘못된 요청으로 처리될 수 있음
- 일관성 유지 필수

### 4.2 ❌ Enum 값을 숫자로 표현

**❌ 잘못된 예**:
```json
{
  "openFlag": 1,
  "questionType": 0
}
```

**✅ 올바른 예**:
```json
{
  "openFlag": "OPEN", // 공개 여부 (OPEN=공개,CLOSE=비공개)
  "questionType": "SUBJECTIVE" // 문제 유형 (SUBJECTIVE=주관식,OBJECTIVE=객관식)
}
```

**왜 중요한가**:
- JSON 역직렬화 시 Enum은 문자열로 처리
- ordinal 값 (0, 1)은 의미가 불명확
- Enum 순서 변경 시 오류 발생

### 4.3 ❌ adminId 필드 포함

**❌ 잘못된 예**:
```json
{
  "adminId": 1,
  "title": "테스트 문제",
  "content": "내용"
}
```

**✅ 올바른 예**:
```json
{
  "title": "테스트 문제",
  "content": "내용"
}
```

**왜 중요한가**:
- adminId는 @JsonIgnore 처리됨
- JWT 토큰에서 자동으로 추출됨
- 클라이언트가 임의로 설정 불가

### 4.4 ❌ 하드코딩된 URL

**❌ 잘못된 예**:
```json
{
  "url": {
    "raw": "http://localhost:8080/admin/v1/questions",
    "host": ["localhost:8080"]
  }
}
```

**✅ 올바른 예**:
```json
{
  "url": {
    "raw": "{{server url}}/admin/v1/questions",
    "host": ["{{server url}}"]
  }
}
```

**왜 중요한가**:
- 환경별로 URL 변경 가능
- 변수 하나만 수정하면 모든 요청 적용
- 재사용성과 유지보수성 향상

### 4.5 ❌ 잘못된 Request 이름

**❌ 잘못된 예**:
```json
{
  "name": "Get Question"
}
{
  "name": "Create Question"
}
{
  "name": "Update Question"
}
{
  "name": "Question Info"
}
```

**✅ 올바른 예**:
```json
{
  "name": "question info"
}
{
  "name": "question add"
}
{
  "name": "question modify"
}
```

**네이밍 규칙**:
- 소문자만 사용
- 공백으로 단어 구분
- 동사는 고정: info, list, add, modify, delete

### 4.6 ❌ Body raw 필드 개행 처리 오류

**❌ 잘못된 예**:
```json
{
  "raw": "{
    \"field1\": \"value1\",
    \"field2\": \"value2\"
  }"
}
```

**✅ 올바른 예**:
```json
{
  "raw": "{\n  \"field1\": \"value1\",\n  \"field2\": \"value2\"\n}"
}
```

**왜 중요한가**:
- JSON 문자열 내부에서는 `\n` 사용 필수
- 실제 개행 사용 시 JSON 파싱 오류

### 4.7 ❌ Body에 주석 누락

**❌ 잘못된 예**:
```json
{
  "raw": "{\n  \"title\": \"테스트\",\n  \"openFlag\": \"OPEN\",\n  \"score\": 100\n}"
}
```

**✅ 올바른 예**:
```json
{
  "raw": "{\n  \"title\": \"테스트\", // 제목\n  \"openFlag\": \"OPEN\", // 공개 여부 (OPEN=공개,CLOSE=비공개)\n  \"score\": 100 // 점수 (최소:1,최대:1000)\n}"
}
```

**왜 중요한가**:
- 사용자가 필드 의미를 바로 이해
- Enum 값 설명으로 선택지 명확화
- 문서화 효과

### 4.8 ❌ DELETE 요청에 불필요한 header

**❌ 잘못된 예**:
```json
{
  "method": "DELETE",
  "header": [
    {
      "key": "Content-Type",
      "value": "application/json"
    }
  ]
}
```

**✅ 올바른 예**:
```json
{
  "method": "DELETE",
  "header": []
}
```

**왜 중요한가**:
- DELETE는 일반적으로 body 없음
- Content-Type 불필요
- 간결성 유지

### 4.9 ❌ host 배열 형식 오류

**❌ 잘못된 예**:
```json
{
  "host": "{{server url}}"
}
{
  "host": "localhost:8080"
}
{
  "host": ["localhost", "8080"]
}
```

**✅ 올바른 예**:
```json
{
  "host": ["{{server url}}"]
}
```

**왜 중요한가**:
- Postman v2.1 스키마는 배열 요구
- 단일 요소 배열이 표준

### 4.10 ❌ 필수/선택 파라미터 disabled 오류

**❌ 잘못된 예**:
```json
{
  "query": [
    {
      "key": "page",
      "value": "1",
      "disabled": true  // ❌ 필수인데 disabled
    },
    {
      "key": "searchValue",
      "value": ""
      // ❌ disabled 누락
    }
  ]
}
```

**✅ 올바른 예**:
```json
{
  "query": [
    {
      "key": "page",
      "value": "1"
      // disabled 없음 = 항상 활성
    },
    {
      "key": "searchValue",
      "value": "",
      "disabled": true  // 선택 파라미터
    }
  ]
}
```

**규칙**:
- page, limit: disabled 없음
- 필터, 검색: disabled: true

## 🔄 5. Before/After 수정 예시

### 예시 1: 전체 수정 (page 버그 + Enum + adminId)

**❌ Before (문제 많음)**:
```json
{
  "name": "Create Question",
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
      "raw": "{\n  \"adminId\": 1,\n  \"title\": \"테스트\",\n  \"openFlag\": 1\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "http://localhost:8080/admin/v1/questions",
      "host": ["localhost:8080"],
      "path": ["admin", "v1", "questions"]
    }
  }
}
```

**✅ After (모두 수정)**:
```json
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
      "raw": "{\n  \"title\": \"테스트\", // 문제 제목\n  \"openFlag\": \"OPEN\" // 공개 여부 (OPEN=공개,CLOSE=비공개)\n}",
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
    }
  }
}
```

**수정 사항**:
1. name: "Create Question" → "question add"
2. adminId 제거
3. openFlag: 1 → "OPEN"
4. 주석 추가
5. URL 하드코딩 → 변수 사용

### 예시 2: 리스트 조회 수정 (page=0 버그)

**❌ Before**:
```json
{
  "url": {
    "raw": "{{server url}}/admin/v1/questions/list?page=0&size=10",
    "query": [
      {
        "key": "page",
        "value": "0",
        "description": "페이지 번호 (0부터 시작)"
      },
      {
        "key": "size",
        "value": "10"
      }
    ]
  }
}
```

**✅ After**:
```json
{
  "url": {
    "raw": "{{server url}}/admin/v1/questions/list?page=1&limit=10",
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
      }
    ]
  }
}
```

**수정 사항**:
1. page: "0" → "1"
2. description: "0부터 시작" → "1부터 시작"
3. size: "10" → "20" (표준값)
4. size description 추가

## 📋 6. 검증 체크리스트

생성된 컬렉션을 다음 체크리스트로 검증하세요:

### 필수 확인
- [ ] page 값이 "1"인가? ⚠️⚠️⚠️
- [ ] page description이 "1부터 시작"인가?
- [ ] 모든 Enum 값이 문자열인가?
- [ ] adminId가 제외되었는가?
- [ ] URL에 변수 `{{server url}}`을 사용했는가?
- [ ] request 이름이 네이밍 규칙을 따르는가?

### 구조 확인
- [ ] info.name이 "{Domain} APIs" 형식인가?
- [ ] auth.type이 "bearer"인가?
- [ ] variable에 "server url"과 "jwt token"이 있는가?
- [ ] DELETE의 header가 빈 배열인가?

### 품질 확인
- [ ] body.raw에 주석이 있는가?
- [ ] Enum 필드 주석에 모든 값이 설명되었는가?
- [ ] description이 명확한가?
- [ ] 선택 파라미터에 disabled: true가 있는가?

## 📖 참조 문서

- **생성 규칙**: `rule/postman/postman_generation_rules.md`
- **포맷 규격**: `rule/postman/postman_format_spec.md`
- **실제 컬렉션**: `/Users/kyle/source/00.document/admin_docs/postman/*.json`

---

**작성일**: 2025-11-08
**버전**: 1.0
**중요 사항**: page=1 규칙 준수 필수!
