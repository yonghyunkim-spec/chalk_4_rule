# API 문서 생성 규칙

## 📋 개요

Controller 클래스를 분석하여 완전하고 정확한 API 문서를 자동으로 생성하는 규칙을 정의합니다.

## 🎯 적용 대상

- `/generate-api-doc` 슬래시 커맨드
- `/sync-api-doc` 슬래시 커맨드

## 📁 출력 경로 규칙

### 파일 경로 패턴
```
C:\Users\user\Documents\chalk4_api_docs\admin_docs\{package}\{domain}\{domain}_{action}.md
```

### Action 네이밍 규칙

| HTTP Method | Controller 메서드 패턴 | Action 이름 | 파일명 예시 |
|-------------|----------------------|------------|------------|
| GET (단일) | `get{Domain}({id})` | `info` | `question_info.md` |
| GET (목록) | `get{Domain}List()` | `list` | `question_list.md` |
| GET (개수) | `get{Domain}ListCount()` | `list_count` | `question_list_count.md` |
| POST | `add{Domain}()` | `add` | `question_add.md` |
| PUT | `mod{Domain}()` | `mod` | `question_mod.md` |
| DELETE | `del{Domain}()` | `del` | `question_del.md` |

### 폴더 구조 예시
```
C:\Users\user\Documents\chalk4_api_docs\admin_docs\
├── ADMIN_API.md                          # 전체 목차
└── question\                             # package
    └── question\                         # domain
        ├── question_info.md
        ├── question_list.md
        ├── question_list_count.md
        ├── question_add.md
        ├── question_mod.md
        └── question_del.md
```

## 🔍 Controller 분석 규칙

### 1. Controller 파일 찾기

**필수 단계**:
```
mcp__serena__find_symbol 사용:
- name_path: "{Domain}Controller"
- relative_path: "src/main/java/com/firsthabit/chalk/{package}"
- include_body: true
- depth: 1  (메서드 포함)
```

### 2. 각 메서드별 추출 정보

| 항목 | 추출 방법 | 비고 |
|------|----------|------|
| HTTP Method | @GetMapping, @PostMapping, @PutMapping, @DeleteMapping | 필수 |
| URL 패턴 | @RequestMapping + @*Mapping | 경로 조합 |
| @PathVariable | 메서드 파라미터 분석 | URL 경로 변수 |
| @RequestBody | 메서드 파라미터 분석 | POST/PUT Body |
| @ModelAttribute | 메서드 파라미터 분석 | GET Query 파라미터 |
| @AuthenticationPrincipal | 메서드 파라미터 분석 | 문서에 표시 안 함 |
| Response 타입 | Service 메서드 추적 | ResponseEntity<?> 내부 추적 |
| 권한 | `adminPrincipal.checkPrivilege()` | AdminPrivilege enum 확인 |

## 📝 Param/VO 분석 규칙

### 1. Request 파라미터 분석

#### SearchParam (GET 목록 조회)
```
mcp__serena__find_symbol:
- name_path: "{Domain}SearchParam"
- depth: 1  (모든 필드 포함)
- include_body: false

주의: PagingParam 상속 필드도 포함 (page, limit)
```

#### AddParam (POST 추가)
```
mcp__serena__find_symbol:
- name_path: "{Domain}AddParam"
- depth: 1

제외 필드: @JsonIgnore 필드 (adminId 등)
```

#### ModParam (PUT 수정)
```
mcp__serena__find_symbol:
- name_path: "{Domain}ModParam"
- depth: 1

제외 필드: @JsonIgnore 필드
```

### 2. Response VO 분석

#### {Domain}Vo (단일 조회)
```
모든 필드 목록 포함
```

#### {Domain}SimpleVo (목록 조회)
```
주요 필드만 포함
```

#### CountVo (카운트 조회)
```
count 필드만 포함
```

### 3. Java 타입 → JSON 타입 변환 규칙

**⚠️ 중요: API 문서는 JSON 인터페이스 관점에서 작성해야 합니다**

Entity/VO/Param 클래스의 Java 타입을 분석한 후, 반드시 JSON 타입으로 변환하여 문서에 표기합니다.

#### 타입 변환 테이블

**⚠️ 중요: 순수 JSON 타입 체계 사용**

JSON 표준에는 6가지 타입만 존재합니다: `string`, `number`, `boolean`, `null`, `object`, `array`

**표기 규칙**:
- 타입은 **소문자**로 표기합니다 (`string`, `number`, `boolean`, `array`, `object`)
- ❌ 잘못된 표기: `String`, `Number`, `Boolean`, `Array`, `Object`
- ✅ 올바른 표기: `string`, `number`, `boolean`, `array`, `object`

| Java 타입 | JSON 타입 (문서 표기) | Description 작성 예시 |
|-----------|---------------------|---------------------|
| `String` | `string` | 문자열 |
| `Integer`, `int` | `number` | 정수 (Integer) |
| `Long`, `long` | `number` | 정수 (Long) |
| `Double`, `double` | `number` | 실수 (Double) |
| `Float`, `float` | `number` | 실수 (Float) |
| `Boolean`, `boolean` | `boolean` | true/false |
| `BigDecimal` | `string` | 금액 (정밀도 보장, 문자열 전송) |
| `LocalDateTime` | `string` | 날짜+시간 (ISO 8601: YYYY-MM-DDTHH:mm:ss) |
| `LocalDate` | `string` | 날짜 (YYYY-MM-DD) |
| `LocalTime` | `string` | 시간 (HH:mm:ss) |
| `Enum` | `string` | Enum 값 (예: "SUBJECTIVE") |
| `List<T>` | `array` | 배열 (요소 타입 명시) |
| `Object`, 커스텀 클래스 (VO) | `object` | 객체 (반드시 세부 필드 문서화 필요) |

**Description 작성 규칙**:
- `number` 타입: Java 타입을 괄호 안에 명시 (예: "정수 (Long)", "점수 (Integer)")
- `string` 타입: 형식이나 용도 명시 (예: "날짜 (YYYY-MM-DD)", "금액 (소수점 2자리)")
- `array` 타입: 요소 타입 명시 (예: "태그 목록 (string 배열)", "ID 목록 (number 배열)")

#### VO(커스텀 클래스) 타입 필드 처리 규칙

**⚠️ 중요: VO 타입 필드는 `object`로만 표기하지 말고, 반드시 세부 필드를 문서화해야 합니다**

**처리 절차**:
1. Response에서 VO 타입 필드 발견 시 해당 VO 파일을 **반드시 읽기**
2. `### {VoName}` subsection 생성
3. VO의 모든 필드를 Java → JSON 타입 변환하여 테이블로 문서화
4. VO 내부의 Enum 필드도 별도 `### {enumFieldName}` subsection으로 문서화

**잘못된 예 ❌ (object로만 표기하고 세부 필드 문서화 없음)**:
```markdown
## Success Response

| name | type | description |
|------|------|-------------|
| data | object | 관리자 정보 |

// ❌ 문제: 세부 필드가 없어서 클라이언트가 어떤 데이터를 받는지 알 수 없음
```

**올바른 예 ✅ (object 타입 표기 + VO 파일 읽어서 세부 필드 문서화)**:
```markdown
## Success Response

```json
{
  "adminId": 123,
  "roleName": "시스템 관리자",
  "status": "NORMAL",
  "email": "admin@example.com"
}
```

### AdminDetailVo

| name | type | description |
|------|------|-------------|
| adminId | number | 관리자 ID (Long) |
| roleName | string | 역할 이름 |
| status | string | 관리자 상태 |
| email | string | 이메일 |

### status

관리자 상태

| Value | Description |
|-------|-------------|
| NORMAL | 정상 |
| LOGOUT | 로그아웃 |
| STOP | 정지 |
```

**또 다른 예시 (Response에 VO 필드가 있는 경우)**:
```markdown
## Success Response

```json
{
  "totalCount": 100,
  "items": [
    {
      "adminId": 123,
      "email": "admin@example.com"
    }
  ]
}
```

| name | type | description |
|------|------|-------------|
| totalCount | number | 전체 개수 (Integer) |
| items | array | 관리자 목록 (AdminSimpleVo 배열) |

### AdminSimpleVo

| name | type | description |
|------|------|-------------|
| adminId | number | 관리자 ID (Long) |
| email | string | 이메일 |
```

**목적**: 클라이언트 개발자가 실제로 받는 JSON 구조를 정확히 알 수 있도록 모든 필드를 명시합니다.

#### 변환 예시

**잘못된 예 ❌ (Java 타입 그대로 사용)**:
```markdown
| name | type | description |
|------|------|-------------|
| id | Long | 퀴즈 고유번호 |
| score | Integer | 점수 |
| createdAt | LocalDateTime | 생성 일시 |
| publishDate | LocalDate | 공개일 |
| questionType | QuestionType | 문제 유형 |
```

**올바른 예 ✅ (JSON 타입으로 변환)**:
```markdown
| name | type | description |
|------|------|-------------|
| id | number | 퀴즈 고유번호 (Long) |
| score | number | 점수 (Integer) |
| createdAt | string | 생성 일시 (ISO 8601) |
| publishDate | string | 공개일 (YYYY-MM-DD) |
| questionType | string | 문제 유형 |
```

#### JSON 예시와 타입 테이블의 일관성

**필수**: JSON 예시와 필드 테이블의 타입은 반드시 일치해야 합니다.

```markdown
## Success Response

| name | type | description |
|------|------|-------------|
| id | number | 퀴즈 고유번호 (Long) |        ✅ number
| createdAt | string | 생성 일시 (ISO 8601) |  ✅ string
| quizType | string | 퀴즈 유형 |             ✅ string

\`\`\`json
{
  "id": 123,                            ✅ number (숫자)
  "createdAt": "2025-01-15T10:30:00",  ✅ string (문자열)
  "quizType": "OBJECTIVE"              ✅ string (문자열)
}
\`\`\`
```

#### 특별 규칙

1. **숫자 타입**:
   - 모두 `number`로 표기
   - Description에 Java 타입 명시: `정수 (Long)`, `점수 (Integer)`, `실수 (Double)` 등
   - 값의 범위나 제약이 있으면 추가: `점수 (Integer, 0-100)`

2. **날짜/시간 타입**:
   - 모두 `string`으로 표기
   - Description에 형식 명시: `(ISO 8601)`, `(YYYY-MM-DD)` 등

3. **Enum 타입**:
   - `string`으로 표기
   - 별도 Enum 정의 섹션에서 가능한 값 나열

4. **BigDecimal**:
   - `string`으로 표기 (정밀도 보장)
   - Description: `금액 (소수점 2자리)` 등

5. **컬렉션 타입**:
   - `array`로 표기
   - Description에 요소 타입 명시: `태그 목록 (string 배열)`, `ID 목록 (number 배열)`

## 🚨 Enum 처리 규칙 (매우 중요)

### 절대 규칙

❌ **절대 Enum 값을 추론하지 마세요**
✅ **항상 실제 Enum 파일을 읽어야 합니다**

### Enum 파일 읽기

Param/VO에서 Enum 필드 발견 시:
```
mcp__serena__find_symbol:
- name_path: "{EnumType}"
- relative_path: "src/main/java/com/firsthabit/chalk/{package}/constants"
- include_body: true
```

### Enum 값 파악

```java
// Enum 파일 예시
@Getter
public enum QuestionType {
    SUBJECTIVE(0, "주관식"),
    OBJECTIVE(1, "객관식"),
    MIXED(2, "혼합형");

    private final int type;
    private final String description;

    QuestionType(int type, String description) {
        this.type = type;
        this.description = description;
    }
}
```

문서 표현:
```markdown
### questionType

| Value | Description |
|-------|-------------|
| SUBJECTIVE | 주관식 |
| OBJECTIVE | 객관식 |
| MIXED | 혼합형 |
```

### Enum Description 작성 규칙

**⚠️ 중요: 모든 Enum 값을 포함하고 구체적인 설명을 작성해야 합니다**

#### 1. 모든 값 포함 (UNKNOWN 포함)

Enum 파일에 정의된 **모든 값**을 문서에 포함해야 합니다:

```java
// ❌ 잘못된 예: UNKNOWN 값을 문서에서 누락
public enum CategoryType {
    PROFILE("profile"),
    RESOURCE("resource"),
    LECTURE("lecture"),
    UNKNOWN("unknown");  // 이 값도 문서에 포함해야 함!
}
```

```markdown
# ✅ 올바른 예: 모든 값 포함
### category

| Value | Description |
|-------|-------------|
| profile | 프로필 이미지 |
| resource | 리소스 파일 |
| lecture | 강의 자료 |
| unknown | 알 수 없는 카테고리 |
```

#### 2. 구체적이고 명확한 Description

단순한 번역이 아닌 **용도와 의미**를 명확히 표현:

```markdown
# ❌ 잘못된 예: 너무 단순
| profile | 프로필 |
| resource | 리소스 |
| lecture | 강의 |

# ✅ 올바른 예: 구체적이고 명확
| profile | 프로필 이미지 |
| resource | 리소스 파일 |
| lecture | 강의 자료 |
```

#### 3. 필요시 예시 추가

파일 타입이나 형식이 명확하지 않은 경우 **예시를 괄호로 추가**:

```markdown
# ✅ 예시 추가
### type

| Value | Description |
|-------|-------------|
| image | 이미지 파일 (jpg, png 등) |
| video | 비디오 파일 (mp4, avi 등) |
| none | 일반 파일 |
| unknown | 알 수 없는 타입 |
```

#### 4. 실제 코드 기반 작성

Enum의 `description` 필드가 있으면 사용하고, 없으면 **코드 로직과 용도를 분석**하여 작성:

```java
// description 필드가 있는 경우
public enum QuestionType {
    SUBJECTIVE(0, "주관식"),  // description 필드 사용
    OBJECTIVE(1, "객관식"),
    MIXED(2, "혼합형");

    private final String description;
}

// description 필드가 없는 경우
public enum CategoryType {
    PROFILE("profile"),  // 값과 용도를 분석하여 설명 작성
    RESOURCE("resource"),
    LECTURE("lecture");

    private String type;  // description 필드 없음
}
```

#### 5. 일관성 유지

같은 도메인/패키지 내에서는 **동일한 스타일과 수준**으로 작성:

```markdown
# ✅ 일관된 스타일
### category
| profile | 프로필 이미지 |
| resource | 리소스 파일 |
| lecture | 강의 자료 |

### type
| image | 이미지 파일 (jpg, png 등) |
| video | 비디오 파일 (mp4, avi 등) |
| none | 일반 파일 |
```

#### 검증 체크리스트

- [ ] Enum 파일의 **모든 값**이 포함되었는가? (UNKNOWN, NONE 등 포함)
- [ ] Description이 단순 번역이 아닌 **구체적인 설명**인가?
- [ ] 필요시 **예시**가 추가되었는가?
- [ ] Enum의 `description` 필드 또는 **실제 용도**를 반영했는가?
- [ ] 같은 도메인 내 다른 Enum과 **일관성**이 있는가?

## 🔎 ErrorCode 분석 규칙

### 1. ErrorCode Enum 읽기

```
mcp__serena__find_symbol:
- name_path: "ErrorCode"
- relative_path: "src/main/java/com/firsthabit/chalk/config/error"
- include_body: true
```

### 2. API별 에러 추적

**추적 대상**:
1. Service 메서드의 `throw new RestException(ErrorCode.XXX)`
2. Service 내부 호출하는 다른 Service의 에러

**공통 에러** (모든 API에 포함):
- JWT_TOKEN_AUTH_ERROR (140001)
- INVALID_AUTH_TOKEN (140003)
- INVALID_PRIVILEGE (140004)

### 3. Error Table 순서

1. 인증/권한 관련 공통 에러 (140xxx)
2. 도메인 특화 에러 (142xxx, 143xxx 등)

## 📄 문서 섹션 구조 (필수 순서)

**⚠️ 중요: 이 순서를 절대 변경하지 마세요**

1. 기본 정보
2. Request Header
3. Path Variable
4. Request Body
   - **Enum 정의는 여기에 바로 포함**
5. Request Example
6. Success Response
7. Fail Response
8. Example (cURL)
9. Notes (필요 시)
10. **Error Code (맨 마지막)**

## 🎨 섹션별 작성 규칙

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

**제외 항목**:
- Permission (별도 표시 안 함)
- Tag (별도 표시 안 함)

### 2. Request Header

```markdown
## Request Header

| name | type | description | required |
|------|------|-------------|-----------|
| Authorization | String | JWT 인증 토큰 | ✅ |
| Content-Type | String | `application/json` | ✅ |
```

**⚠️ 중요 규칙**:
- Authorization description에서 **"Bearer {token}" 문구 제외**
- 올바른 예: "JWT 인증 토큰"
- 잘못된 예: "Bearer {token} 형식의 JWT 인증 토큰"

### 3. Path Variable

```markdown
## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| questionId | Long | 문제 고유번호 | ✅ |
```

**⚠️ Enum 타입 Path Variable 규칙**:

Path Variable이 Enum 타입인 경우, **description에 가능한 값을 괄호로 포함**해야 합니다:

```markdown
## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| adminKey | String | 관리자 인증 키 | ✅ |
| category | String | 파일 카테고리 (profile/resource/lecture/unknown) | ✅ |
| type | String | 파일 타입 (image/video/none/unknown) | ✅ |
```

**목적**: 사용자가 Path Variable 테이블만 봐도 어떤 값을 사용할 수 있는지 즉시 파악할 수 있습니다.

**형식**: `(value1/value2/value3/...)`
- 슬래시(`/`)로 구분
- 소문자로 표기 (Enum의 실제 값 기준)
- 모든 값 포함 (UNKNOWN 포함)

Path Variable이 없으면:
```markdown
## Path Variable

> 없음
```

### 4. Request Body

**⚠️ 중요: Request Parameters vs Request Body 구분**

Spring Boot Controller에서:
- **GET 메서드 + @RequestParam** → 섹션명: `## Query Parameters` 사용
- **POST/PUT 메서드 + @RequestBody** → 섹션명: `## Request Body` 사용
- **❌ 절대 금지**: `## Request Parameters` 섹션명 사용 금지 (혼란 유발)

**올바른 섹션명**:
```markdown
# ✅ GET 메서드
## Query Parameters

# ✅ POST/PUT 메서드
## Request Body
```

```markdown
## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| title | String | 문제 제목 | ✅ |
| questionType | String | 문제 유형 | ✅ |
| difficulty | String | 난이도 | ✅ |
| score | Integer | 점수 | ✅ |
```

**⚠️ Enum 정의 위치 규칙**:

Enum 필드가 있으면 **Request Body (또는 Query Parameters) 섹션 바로 아래**에 정의:

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

**잘못된 위치**:
- ❌ 문서 하단 부록
- ❌ Error Code 이전
- ❌ Notes 섹션

**⚠️ page, limit 필수값 규칙**:

리스트 조회 API의 page, limit는 **기본값이 있어도 필수(✅)로 표시**:

```markdown
| page | Integer | 페이지 번호 (기본값: 1) | ✅ |
| limit | Integer | 페이지당 개수 (기본값: 10) | ✅ |
```

잘못된 예:
```markdown
| page | Integer | 페이지 번호 (기본값: 1) | ⛔ |  # 잘못됨!
```

Request Body가 없으면:
```markdown
## Request Body

> 없음
```

### 5. Request Example

Request Body가 있으면:
```markdown
### Request Example

\`\`\`json
{
  "title": "테스트 문제",
  "questionType": "SUBJECTIVE",
  "difficulty": "BASIC",
  "score": 100
}
\`\`\`
```

Request Body가 없으면 생략

### 6. Success Response

#### 단일 객체 응답
```markdown
## Success Response

| name | type | description |
|------|------|-------------|
| questionId | Long | 문제 고유번호 |
| title | String | 문제 제목 |
| questionType | String | 문제 유형 |
| createdAt | String | 생성일시 (ISO 8601) |

\`\`\`json
{
  "questionId": 123,
  "title": "테스트 문제",
  "questionType": "SUBJECTIVE",
  "createdAt": "2025-10-29T10:30:00"
}
\`\`\`
```

#### 배열 응답 (리스트)
```markdown
## Success Response

> Response는 QuestionSimpleVo 배열 형태입니다.

| name | type | description |
|------|------|-------------|
| questionId | Long | 문제 고유번호 |
| title | String | 문제 제목 |

\`\`\`json
[
  {
    "questionId": 123,
    "title": "문제 1"
  },
  {
    "questionId": 124,
    "title": "문제 2"
  }
]
\`\`\`
```

#### 204 No Content (DELETE)
```markdown
## Success Response

> HTTP 204 No Content

> 본문 없음
```

### 7. Fail Response

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

### 8. Example (cURL)

**⚠️ Authorization 헤더 규칙**:

```bash
# ✅ 올바른 예 (Bearer 제외)
curl -X GET https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json"
```

```bash
# ❌ 잘못된 예 (Bearer 포함)
curl -X GET https://api.server.com/admin/v1/questions/123 \
  -H "Authorization: Bearer {JWT_TOKEN}" \  # 잘못됨!
  -H "Content-Type: application/json"
```

Request Body가 있는 경우:
```bash
curl -X POST https://api.server.com/admin/v1/questions \
  -H "Authorization: {JWT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "테스트 문제",
    "questionType": "SUBJECTIVE"
  }'
```

### 9. Notes

특별한 주의사항이나 비즈니스 로직 설명이 필요한 경우에만 작성:

```markdown
## Notes

### Twin Question (변형 문제) 생성 규칙

- Twin Question 생성 시 `originQuestionId`를 반드시 지정해야 합니다.
- Twin Question은 Origin Question과 동일한 `questionGroupId`를 사용해야 합니다.
```

Notes가 필요 없으면 생략

### 10. Error Code

**⚠️ 중요: 항상 문서의 맨 마지막에 위치**

```markdown
## Error Code

| Code | Status | Message | Explain |
|------|---------|----------|----------|
| 140001 | 401 | JWT_TOKEN_AUTH_ERROR | JWT 토큰 인증 오류 |
| 140003 | 401 | INVALID_AUTH_TOKEN | 유효하지 않은 인증 토큰 |
| 140004 | 403 | INVALID_PRIVILEGE | 권한 없음 (QUESTION_MANAGE 권한 필요) |
| 142001 | 404 | QUESTION_NOT_EXIST | 존재하지 않는 문제입니다. |
| 142002 | 400 | QUESTION_ALREADY_EXIST | 이미 존재하는 문제입니다. |
```

**에러 코드 순서**:
1. 인증/권한 관련 공통 에러 (140xxx) 먼저
2. 도메인 특화 에러 (142xxx, 143xxx 등) 나중

**잘못된 위치**:
- ❌ Notes 이전
- ❌ Example 이전
- ❌ 문서 중간 어디든

## 📋 검증 체크리스트

각 API 문서마다 확인:

### 필수 확인 사항
- [ ] 기본 정보가 정확한가?
- [ ] Request Header가 누락되지 않았는가?
- [ ] Authorization에서 Bearer 문구가 제거되었는가? ⚠️
- [ ] Path Variable이 모두 포함되었는가?
- [ ] **Enum 타입 Path Variable의 description에 가능한 값이 포함되었는가? (예: "파일 카테고리 (profile/resource/lecture/unknown)")** ⚠️
- [ ] **섹션명이 올바른가? (GET=Query Parameters, POST/PUT=Request Body)** ⚠️
- [ ] **"Request Parameters" 섹션명을 사용하지 않았는가?** ⚠️
- [ ] Request Body 또는 Query Parameters 필드가 모두 포함되었는가?
- [ ] **Enum 정의가 Request Body (또는 Query Parameters) 섹션 바로 아래에 있는가?** ⚠️
- [ ] **page, limit가 필수값(✅)으로 표시되었는가?** ⚠️
- [ ] Request Example이 실제 실행 가능한가?
- [ ] Success Response 필드가 모두 포함되었는가?
- [ ] Success Response JSON 예시가 정확한가?
- [ ] Fail Response 예시가 있는가?
- [ ] cURL 예시가 실제 실행 가능한가?
- [ ] cURL에서 Bearer가 제거되었는가? ⚠️
- [ ] Notes가 필요한 경우 작성했는가?
- [ ] **Error Code 섹션이 문서 맨 마지막에 있는가?** ⚠️
- [ ] Error Table에 모든 가능한 에러가 포함되었는가?

### Java 타입 → JSON 타입 변환 확인
- [ ] **모든 필드 타입이 순수 JSON 타입(string, number, boolean, array, object)으로 변환되었는가?** ⚠️
- [ ] Integer, Long, Double 등이 number로 표기되었는가? ⚠️
- [ ] LocalDateTime, LocalDate, LocalTime이 string으로 표기되었는가? ⚠️
- [ ] Enum 타입이 string으로 표기되었는가? ⚠️
- [ ] BigDecimal이 string으로 표기되었는가? ⚠️
- [ ] List<T>가 array로 표기되었는가? ⚠️
- [ ] JSON 예시와 필드 테이블의 타입이 일치하는가? ⚠️
- [ ] Description에 Java 타입이 명시되었는가? (예: "정수 (Long)") ⚠️

### VO(커스텀 클래스) 타입 필드 처리 확인
- [ ] **Response에 VO 타입이 있을 경우 해당 VO 파일을 읽었는가?** ⚠️
- [ ] **VO를 `object`로만 표기하지 않았는가?** ⚠️
- [ ] **`### {VoName}` subsection을 생성했는가?** ⚠️
- [ ] VO의 모든 필드를 JSON 타입으로 변환하여 테이블로 문서화했는가? ⚠️
- [ ] VO 내부의 Enum 필드도 별도 subsection으로 문서화했는가? ⚠️

### Enum 관련 확인
- [ ] 모든 Enum 필드에 대해 실제 Enum 파일을 읽었는가? ⚠️
- [ ] Enum 값을 추론하지 않았는가? ⚠️
- [ ] **Enum 파일의 모든 값이 포함되었는가? (UNKNOWN, NONE 등 포함)** ⚠️
- [ ] **Description이 단순 번역이 아닌 구체적인 설명인가?** ⚠️
- [ ] **필요시 예시가 추가되었는가? (예: "이미지 파일 (jpg, png 등)")** ⚠️
- [ ] Enum 정의가 Request Body 또는 Path Variable 섹션 바로 아래에 있는가? ⚠️

### 섹션 순서 확인
- [ ] 섹션 순서가 정확한가? ⚠️
  1. 기본 정보
  2. Request Header
  3. Path Variable
  4. Request Body (POST/PUT) 또는 Query Parameters (GET) (+ Enum 정의)
  5. Request Example
  6. Success Response
  7. Fail Response
  8. Example (cURL)
  9. Notes (선택)
  10. Error Code (맨 마지막)

## ⚠️ 자주 발생하는 실수

### 1. Path Variable에서 Enum 값 누락
```markdown
# ❌ 잘못된 예 (Enum 값 미포함)
## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| category | String | 파일 카테고리 | ✅ |
| type | String | 파일 타입 | ✅ |

// 문제: 어떤 값을 사용할 수 있는지 알 수 없음!

# ✅ 올바른 예 (Enum 값 포함)
## Path Variable

| name | type | description | required |
|------|------|-------------|-----------|
| category | String | 파일 카테고리 (profile/resource/lecture/unknown) | ✅ |
| type | String | 파일 타입 (image/video/none/unknown) | ✅ |

// 가독성 향상: Path Variable 테이블만 봐도 사용 가능한 값을 즉시 파악 가능
```

### 2. Request Parameters 섹션명 사용
```markdown
# ❌ 잘못된 예 (POST 메서드인데 Request Parameters 사용)
| **Method** | POST |
| **Endpoint** | `/admin/v1/quizzes` |

## Request Parameters

| name | type | description | required |
|------|------|-------------|-----------|
| title | String | 퀴즈 제목 | ✅ |

// 문제: POST 메서드는 Request Body를 사용해야 함!

# ✅ 올바른 예
| **Method** | POST |
| **Endpoint** | `/admin/v1/quizzes` |

## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| title | String | 퀴즈 제목 | ✅ |
```

```markdown
# ❌ 잘못된 예 (GET 메서드인데 Request Body 사용)
| **Method** | GET |
| **Endpoint** | `/admin/v1/quizzes/list` |

## Request Body

| name | type | description | required |
|------|------|-------------|-----------|
| page | Integer | 페이지 번호 | ✅ |

// 문제: GET 메서드는 Query Parameters를 사용해야 함!

# ✅ 올바른 예
| **Method** | GET |
| **Endpoint** | `/admin/v1/quizzes/list` |

## Query Parameters

| name | type | description | required |
|------|------|-------------|-----------|
| page | Integer | 페이지 번호 | ✅ |
```

### 2. Enum 값 추론 및 불완전한 Description
```markdown
# ❌ 잘못된 예 (값 추론)
### questionType

| Value | Description |
|-------|-------------|
| 0 | 주관식 |
| 1 | 객관식 |

# ❌ 잘못된 예 (UNKNOWN 값 누락)
### category

| Value | Description |
|-------|-------------|
| profile | 프로필 |
| resource | 리소스 |
| lecture | 강의 |
// UNKNOWN 값이 누락됨!

# ❌ 잘못된 예 (단순 번역)
### type

| Value | Description |
|-------|-------------|
| image | 이미지 |
| video | 비디오 |
| none | 없음 |
// 너무 단순함!

# ✅ 올바른 예 (파일에서 읽고 모든 값 포함, 구체적인 설명)
### questionType

| Value | Description |
|-------|-------------|
| SUBJECTIVE | 주관식 |
| OBJECTIVE | 객관식 |
| MIXED | 혼합형 |

### category

| Value | Description |
|-------|-------------|
| profile | 프로필 이미지 |
| resource | 리소스 파일 |
| lecture | 강의 자료 |
| unknown | 알 수 없는 카테고리 |

### type

| Value | Description |
|-------|-------------|
| image | 이미지 파일 (jpg, png 등) |
| video | 비디오 파일 (mp4, avi 등) |
| none | 일반 파일 |
| unknown | 알 수 없는 타입 |
```

### 3. Enum 위치
```markdown
# ❌ 잘못된 예 (문서 하단)
## Error Code
...

## 부록: Enum 정의
### questionType
...

# ✅ 올바른 예 (Request Body 바로 아래)
## Request Body
| questionType | String | 문제 유형 | ✅ |

### questionType
| Value | Description |
|-------|-------------|
| SUBJECTIVE | 주관식 |
```

### 3. Error Code 위치
```markdown
# ❌ 잘못된 예 (중간에 위치)
## Error Code
...

## Notes
...

# ✅ 올바른 예 (맨 마지막)
## Notes
...

## Error Code
...
```

### 4. Authorization 헤더
```markdown
# ❌ 잘못된 예
| Authorization | String | Bearer {token} 형식의 JWT 인증 토큰 | ✅ |

curl -H "Authorization: Bearer {JWT_TOKEN}"

# ✅ 올바른 예
| Authorization | String | JWT 인증 토큰 | ✅ |

curl -H "Authorization: {JWT_TOKEN}"
```

### 5. Java 타입을 그대로 사용
```markdown
# ❌ 잘못된 예 (Java 타입 사용)
| name | type | description |
|------|------|-------------|
| id | Long | 퀴즈 고유번호 |
| score | Integer | 점수 |
| createdAt | LocalDateTime | 생성 일시 |
| publishDate | LocalDate | 공개일 |
| questionType | QuestionType | 문제 유형 |
| tags | List<String> | 태그 목록 |

\`\`\`json
{
  "id": 123,                            // 실제 JSON은 number
  "score": 85,                          // number
  "createdAt": "2025-01-15T10:30:00",  // string
  "publishDate": "2025-01-15",         // string
  "questionType": "SUBJECTIVE",        // string
  "tags": ["수학", "정수론"]            // array
}
\`\`\`

# ✅ 올바른 예 (순수 JSON 타입으로 변환)
| name | type | description |
|------|------|-------------|
| id | number | 퀴즈 고유번호 (Long) |
| score | number | 점수 (Integer) |
| createdAt | string | 생성 일시 (ISO 8601) |
| publishDate | string | 공개일 (YYYY-MM-DD) |
| questionType | string | 문제 유형 |
| tags | array | 태그 목록 (string 배열) |

\`\`\`json
{
  "id": 123,                            // number - 일치!
  "score": 85,                          // number - 일치!
  "createdAt": "2025-01-15T10:30:00",  // string - 일치!
  "publishDate": "2025-01-15",         // string - 일치!
  "questionType": "SUBJECTIVE",        // string - 일치!
  "tags": ["수학", "정수론"]            // array - 일치!
}
\`\`\`
```

### 6. page, limit 필수값
```markdown
# ❌ 잘못된 예
| page | Integer | 페이지 번호 (기본값: 1) | ⛔ |
| size | Integer | 페이지당 개수 (기본값: 10) | ⛔ |

# ✅ 올바른 예
| page | Integer | 페이지 번호 (기본값: 1) | ✅ |
| limit | Integer | 페이지당 개수 (기본값: 10) | ✅ |
```

### 7. VO 타입을 object로만 표기
```markdown
# ❌ 잘못된 예 (세부 필드 문서화 없음)
## Success Response

| name | type | description |
|------|------|-------------|
| data | object | 관리자 상세 정보 |

\`\`\`json
{
  "adminId": 123,
  "email": "admin@example.com"
}
\`\`\`

// 문제: AdminDetailVo의 세부 필드가 문서화되지 않음!

# ✅ 올바른 예 (VO 파일 읽어서 세부 필드 문서화)
## Success Response

\`\`\`json
{
  "adminId": 123,
  "email": "admin@example.com",
  "status": "NORMAL"
}
\`\`\`

### AdminDetailVo

| name | type | description |
|------|------|-------------|
| adminId | number | 관리자 ID (Long) |
| email | string | 이메일 |
| status | string | 관리자 상태 |

### status

| Value | Description |
|-------|-------------|
| NORMAL | 정상 |
| LOGOUT | 로그아웃 |
```

## 🔗 참조 문서

- **템플릿**: `rule/api_documentation/api_doc_template.md`
- **예시**: `rule/api_documentation/api_doc_examples.md`
- **슬래시 커맨드**: `.claude/commands/generate-api-doc.md`
- **가이드**: `guide/api_doc_auto_template.md`

---

**중요**: 이 규칙을 **100% 준수**해야 합니다. 예외는 허용되지 않습니다.
