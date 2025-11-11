# Guide 폴더

chalk-admin 프로젝트 개발을 위한 가이드 문서 모음입니다.

## 📁 문서 구조

```
guide/
├── README.md                # 이 파일 (가이드 폴더 소개)
├── CODE_STYLE_GUIDE.md      # 코드 스타일 가이드라인
├── SLASH_COMMANDS.md        # 슬래시 커맨드 사용법
└── MCP_TOOLS.md             # MCP 도구 설명
```

## 📚 문서 설명

### [CODE_STYLE_GUIDE.md](CODE_STYLE_GUIDE.md)

프로젝트의 코딩 규칙과 패턴을 정리한 문서입니다.

**포함 내용**:
- 🚫 절대 금지사항 (8가지)
- ✅ 필수 사용사항 (6가지)
- Repository 패턴 (JPA + QueryDSL)
- MapStruct 패턴
- SearchParam 패턴
- Service 패턴
- Controller 패턴
- 참조 패키지 (board, menu.repository, question)
- 패키지 구조

**핵심 원칙**:
1. QueryDSL로 VO 직접 반환
2. MapStruct로 자동 매핑
3. JPA naming convention
4. Enum으로 상수 관리
5. PagingParam 상속
6. GET은 @ModelAttribute
7. 복합키는 camelCase 연결
8. QueryRepository는 구현체만

---

### [SLASH_COMMANDS.md](SLASH_COMMANDS.md)

Claude Code의 슬래시 커맨드 사용법을 정리한 문서입니다.

**Phase 1: 생성 커맨드**:
- `/generate-domain` - DDL → 도메인 패키지 생성
- `/generate-api-doc` - Controller → API 문서 생성
- `/generate-postman` - Controller → Postman 컬렉션 생성
- `/generate-repo-test` - Repository → 테스트 코드 생성

**Phase 2: 동기화 커맨드**:
- `/sync-ddl-changes` - DDL 변경사항 반영
- `/sync-api-doc` - API 문서 동기화
- `/sync-postman` - Postman 컬렉션 동기화

**일반적인 워크플로우**:
```bash
# 새 도메인 추가
/generate-domain {domain}
/generate-api-doc {package}
/generate-postman {package}
/generate-repo-test {package}

# 기존 도메인 수정
/sync-ddl-changes {domain}
/sync-api-doc {package}
/sync-postman {package}
```

---

### [MCP_TOOLS.md](MCP_TOOLS.md)

Claude Code가 활용하는 MCP(Model Context Protocol) 도구들을 설명한 문서입니다.

**Serena MCP 서버**:
- 파일 탐색: `list_dir`, `find_file`, `search_for_pattern`
- 심볼 기반: `get_symbols_overview`, `find_symbol`, `find_referencing_symbols`
- 편집: `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`, `rename_symbol`
- 메모리: `write_memory`, `read_memory`, `list_memories`, `delete_memory`
- 사고: `think_about_*`

**Sequential Thinking MCP**:
- 복잡한 문제 단계적 해결
- 가설 생성 및 검증

**IDE MCP**:
- 진단 정보: `getDiagnostics`

---

## 🚀 빠른 시작

### 1. 새 도메인 추가하기

```bash
# 1단계: DDL 파일 준비
# src/main/resources/db/migration/V*__*.sql

# 2단계: 도메인 패키지 생성
/generate-domain chapter

# 3단계: 문서 생성
/generate-api-doc chapter
/generate-postman chapter

# 4단계: 테스트 생성
/generate-repo-test chapter

# 5단계: 테스트 실행
./gradlew test --tests "ChapterRepositoryTest"
```

### 2. 코드 작성 시

1. **참조 패키지 확인**: `board` 패키지를 항상 참조하세요.
2. **코드 스타일 준수**: `CODE_STYLE_GUIDE.md`의 금지사항을 지키세요.
3. **슬래시 커맨드 활용**: 반복 작업은 자동화하세요.

### 3. 문제 해결 시

1. **MCP 도구 이해**: Claude가 어떤 도구를 사용하는지 알면 도움이 됩니다.
2. **심볼 기반 접근**: 파일 전체보다는 클래스/메서드 단위로 작업하세요.
3. **참조 추적**: 리팩토링 시 `find_referencing_symbols`로 영향 범위를 파악하세요.

---

## 💡 추가 자료

### 프로젝트 루트의 CLAUDE.md

프로젝트 전체 개요와 추가 정보는 프로젝트 루트의 `CLAUDE.md` 파일을 참조하세요.

**주요 내용**:
- 프로젝트 개요
- 기술 스택
- 개발 명령어 (빌드, 실행, 테스트)
- 아키텍처 & 코드 패턴
- 테스트 가이드
- 패키지 상태

### 참조 패키지 직접 확인

이론보다는 실제 코드를 보는 것이 가장 좋습니다:

```
src/main/java/com/firsthabit/chalk/
├── board/           # ⭐ 모든 패턴의 기준
├── menu/            # 복합키 패턴 참조
└── question/        # 복잡한 관계 참조
```

---

## 📞 도움이 필요하신가요?

1. **코드 스타일**: `CODE_STYLE_GUIDE.md` 참조
2. **자동화**: `SLASH_COMMANDS.md` 참조
3. **도구 이해**: `MCP_TOOLS.md` 참조
4. **프로젝트 전반**: 루트의 `CLAUDE.md` 참조
5. **실제 예시**: `board` 패키지 코드 참조

---

**마지막 업데이트**: 2025-01-01

**관리**: 가이드 문서는 주요 패턴 변경 시 함께 업데이트됩니다.
