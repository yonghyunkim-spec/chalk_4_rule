# Rule 폴더 개요

## 📋 목적

이 폴더는 Claude Code가 슬래시 커맨드를 실행할 때 **항상 참조해야 하는 규칙**들을 정의합니다.

## 🎯 핵심 원칙

### 1. 강제성
- 이 폴더의 모든 규칙은 **필수**입니다
- Claude는 작업 전에 관련 규칙 문서를 **반드시 읽어야** 합니다
- 규칙을 위반하는 결과물은 **허용되지 않습니다**

### 2. 우선순위
규칙 문서의 우선순위:
1. **rule/** 폴더 (최우선)
2. **CLAUDE.md** (프로젝트 전체 규칙)
3. **guide/** 폴더 (참조 가이드)
4. **.claude/commands/** 폴더 (슬래시 커맨드 정의)

### 3. 적용 범위
각 슬래시 커맨드별 적용 규칙:

| 슬래시 커맨드 | 참조 규칙 문서 |
|--------------|---------------|
| /compare-ddl | compare_ddl_rules.md |
| /generate-domain | domain_generation_rules.md |
| /sync-ddl-changes | domain_generation_rules.md |
| /generate-api-doc | api_documentation/api_doc_generation_rules.md<br>api_documentation/api_doc_template.md |
| /sync-api-doc | api_documentation/api_doc_generation_rules.md<br>api_documentation/api_doc_template.md |
| /generate-postman | postman/postman_generation_rules.md<br>postman/postman_format_spec.md |
| /sync-postman | postman/postman_generation_rules.md<br>postman/postman_format_spec.md |
| /generate-repo-test | repository_test_rules.md |

### 4. Obsidian Workflow (선택적)
**중요**: 이 규칙은 사용자가 "obsidian" 또는 "옵시디언" 키워드를 명시적으로 언급한 경우에만 적용됩니다.

| 트리거 키워드 | 참조 규칙 문서 |
|-------------|---------------|
| "obsidian" 또는 "옵시디언" | obsidian_workflow_rules.md |

**적용 예시**:
- ✅ "obsidian을 사용해서 API 문서 생성 규칙을 만들어줘"
- ✅ "옵시디언으로 계획 세워서 진행해"
- ❌ "API 문서 생성 규칙을 만들어줘" (키워드 없음 → TodoWrite 사용)

## 📁 폴더 구조

```
rule/
├── README.md                              # (현재 파일) rule 폴더 개요
├── compare_ddl_rules.md                   # DDL 비교 규칙
├── domain_generation_rules.md             # 도메인 패키지 생성 규칙
├── repository_test_rules.md               # Repository 테스트 생성 규칙
├── obsidian_workflow_rules.md             # Obsidian 작업 기록 규칙 (선택적)
├── api_documentation/                     # API 문서 관련 규칙
│   ├── api_doc_generation_rules.md        # API 문서 생성 규칙
│   ├── api_doc_template.md                # API 문서 템플릿 (강한 룰)
│   └── api_doc_examples.md                # API 문서 예시
└── postman/                               # Postman 관련 규칙
    ├── postman_generation_rules.md        # Postman 컬렉션 생성 규칙
    ├── postman_format_spec.md             # Postman v2.1 포맷 규격
    └── postman_examples.md                # Postman 예시
```

## 🚨 중요 주의사항

### 절대 규칙 (Never Break)

#### 1. Enum 값 처리
- ❌ **절대 Enum 값을 추론하지 마세요**
- ✅ **항상 실제 Enum 파일을 읽어야 합니다**
- 📍 적용: domain_generation, api_doc_generation, postman_generation, repository_test

#### 2. 파일 읽기 우선
- ❌ **절대 Entity/VO/Param의 필드를 추측하지 마세요**
- ✅ **항상 실제 파일을 읽어서 확인해야 합니다**
- 📍 적용: 모든 슬래시 커맨드

#### 3. 템플릿 준수
- ❌ **절대 임의로 섹션 순서를 변경하지 마세요**
- ✅ **api_doc_template.md의 섹션 순서를 정확히 따라야 합니다**
- 📍 적용: generate-api-doc, sync-api-doc

#### 4. 네이밍 규칙
- ❌ **절대 자유로운 이름을 사용하지 마세요**
- ✅ **정의된 네이밍 규칙을 정확히 따라야 합니다**
- 📍 적용: generate-postman, sync-postman

## 📖 사용 방법

### Claude Code 작업자용

#### 슬래시 커맨드 실행 전
1. 해당 커맨드의 규칙 문서 확인
2. 관련 예시 문서 참조
3. 규칙 체크리스트 확인

#### 작업 중
1. 각 단계마다 규칙 준수 확인
2. 의심스러운 경우 규칙 문서 재확인
3. 예외 상황은 규칙 문서에 정의된 처리 방법 따름

#### 작업 후
1. 생성된 결과물이 규칙 준수하는지 검증
2. 체크리스트 모든 항목 확인
3. 위반 사항 발견 시 즉시 수정

### 사용자용

#### 규칙 추가/수정
1. 관련 규칙 문서 수정
2. 예시 문서도 함께 업데이트
3. README.md 업데이트 (필요시)

#### 규칙 검증
1. 슬래시 커맨드 실행
2. Claude가 규칙 참조했는지 확인
3. 생성된 결과물 품질 확인

## 🔍 규칙 문서 작성 원칙

### 1. 명확성
모든 규칙은 명확하고 구체적으로 작성:
- 애매한 표현 금지
- 구체적인 예시 포함
- ✅ 올바른 예 / ❌ 잘못된 예 병기

### 2. 완전성
모든 케이스 커버:
- 일반적인 경우
- Edge case
- 예외 상황 처리

### 3. 일관성
다른 문서와 일관성 유지:
- CLAUDE.md와 일치
- guide/ 폴더와 일치
- 슬래시 커맨드 정의와 일치

### 4. 참조성
쉽게 참조 가능하도록:
- 명확한 섹션 구조
- 빠른 검색 가능한 제목
- 체크리스트 형태

## 📊 규칙 준수도 체크

### 자동 체크 항목
- [ ] Enum 파일을 실제로 읽었는가?
- [ ] Entity/VO/Param 파일을 읽었는가?
- [ ] 템플릿 섹션 순서가 올바른가?
- [ ] 네이밍 규칙을 준수했는가?

### 수동 체크 항목
- [ ] 생성된 코드가 컴파일되는가?
- [ ] 생성된 문서가 읽기 쉬운가?
- [ ] 생성된 Postman 컬렉션이 실행 가능한가?
- [ ] 생성된 테스트가 통과하는가?

## ⚡ Quick Reference

### 가장 자주 확인해야 할 규칙

#### API 문서 생성 시
1. Enum 정의는 Request Body 바로 아래
2. Error Code는 문서 맨 마지막
3. Authorization에서 Bearer 제외
4. page, size는 필수값

#### Postman 생성 시
1. 네이밍: info, list, add, modify, delete
2. Enum 값: 문자열 사용
3. adminId: Request Body에서 제외
4. page: 1부터 시작

#### Domain 생성 시
1. Enum 값을 사용자에게 질문
2. nullable = false 필드 모두 설정
3. PagingParam 상속
4. QueryRepository는 구현체만

#### Repository 테스트 생성 시
1. Enum 파일 읽기
2. 타임스탬프 수동 설정
3. 모든 필수 필드 설정
4. Given-When-Then 패턴

## 🔄 업데이트 이력

| 날짜 | 버전 | 변경 내용 |
|------|------|-----------|
| 2025-11-08 | 1.0 | 초기 작성 |

---

**중요**: 이 폴더의 모든 규칙은 Claude Code가 **반드시** 따라야 합니다.
규칙을 무시하거나 위반하는 것은 허용되지 않습니다.
