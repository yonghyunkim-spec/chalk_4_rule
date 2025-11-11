# MCP Tools 가이드

Claude Code에서 활용 중인 MCP(Model Context Protocol) 서버와 도구들입니다.

## 📋 개요

MCP는 Claude가 외부 도구와 통신할 수 있게 하는 프로토콜입니다. 이 프로젝트에서는 코드 분석, 편집, 사고 등의 작업을 위한 MCP 도구들을 활용합니다.

---

## 🔍 Serena MCP 서버

코드베이스 탐색 및 편집을 위한 전문 도구 모음입니다.

### 파일 탐색 도구

#### `list_dir`
디렉토리의 파일과 하위 디렉토리를 나열합니다.

**예시**:
```
디렉토리: src/main/java/com/firsthabit/chalk/board
재귀: true
```

**용도**: 패키지 구조 파악, 파일 목록 확인

---

#### `find_file`
파일명 패턴으로 파일을 검색합니다.

**예시**:
```
파일 마스크: *Controller.java
경로: src/main/java/com/firsthabit/chalk
```

**용도**: Controller, Repository 등 특정 타입 파일 찾기

---

#### `search_for_pattern`
정규식 패턴으로 코드 내용을 검색합니다.

**예시**:
```
패턴: @RestController
경로: src/main/java/com/firsthabit/chalk/board
코드 파일만: true
```

**용도**: 특정 애노테이션, 메서드, 패턴 검색

---

### 심볼 기반 도구

#### `get_symbols_overview`
파일의 최상위 심볼(클래스, 메서드 등) 개요를 가져옵니다.

**예시**:
```
파일: src/main/java/com/firsthabit/chalk/board/controller/BoardController.java
```

**반환**: 클래스, 메서드, 필드 목록

**용도**: 파일 구조 빠르게 파악

---

#### `find_symbol`
이름 경로로 심볼을 찾습니다.

**예시**:
```
이름 경로: BoardController/getBoardList
경로: src/main/java/com/firsthabit/chalk/board
본문 포함: true
깊이: 1
```

**패턴**:
- `BoardController` - 클래스만
- `BoardController/getBoardList` - 특정 메서드
- `/BoardController` - 최상위 클래스만

**용도**:
- 클래스, 메서드, 필드 정확히 찾기
- 메서드 본문 읽기
- 상속 구조 파악

---

#### `find_referencing_symbols`
특정 심볼을 참조하는 모든 위치를 찾습니다.

**예시**:
```
이름 경로: BoardService/getBoard
파일: src/main/java/com/firsthabit/chalk/board/service/BoardService.java
```

**반환**: 참조 위치, 코드 스니펫, 심볼 메타데이터

**용도**:
- 메서드 사용처 찾기
- 리팩토링 영향 범위 파악
- 의존성 분석

---

### 편집 도구

#### `replace_symbol_body`
심볼의 본문을 교체합니다.

**예시**:
```
이름 경로: BoardService/getBoard
파일: src/main/java/com/firsthabit/chalk/board/service/BoardService.java
새 본문: (메서드 전체 코드)
```

**주의**:
- 시그니처 포함 전체 교체
- 주석, import는 제외

**용도**: 메서드, 클래스 전체 교체

---

#### `insert_after_symbol`
심볼 뒤에 코드를 삽입합니다.

**예시**:
```
이름 경로: BoardService/getBoard
파일: src/main/java/com/firsthabit/chalk/board/service/BoardService.java
본문: (새 메서드 코드)
```

**용도**:
- 새 메서드 추가
- 새 필드 추가

---

#### `insert_before_symbol`
심볼 앞에 코드를 삽입합니다.

**예시**:
```
이름 경로: BoardController
파일: src/main/java/com/firsthabit/chalk/board/controller/BoardController.java
본문: import문 또는 새 클래스
```

**용도**:
- import 추가
- 클래스 앞에 새 클래스 추가

---

#### `rename_symbol`
심볼 이름을 프로젝트 전체에서 변경합니다.

**예시**:
```
이름 경로: BoardService/getBoard
파일: src/main/java/com/firsthabit/chalk/board/service/BoardService.java
새 이름: getBoardInfo
```

**용도**:
- 메서드, 클래스 이름 변경
- 모든 참조 자동 업데이트

---

### 메모리 관리

#### `write_memory`
프로젝트 정보를 메모리 파일에 저장합니다.

**예시**:
```
메모리 이름: board_package_structure
내용: board 패키지 구조 설명 (markdown)
```

**용도**: 프로젝트 지식 저장, 재사용

---

#### `read_memory`
저장된 메모리 파일을 읽습니다.

**예시**:
```
메모리 파일: board_package_structure.md
```

**용도**: 이전에 저장한 정보 불러오기

---

#### `list_memories`
저장된 모든 메모리 파일 목록을 가져옵니다.

**용도**: 어떤 정보가 저장되어 있는지 확인

---

#### `delete_memory`
메모리 파일을 삭제합니다.

**예시**:
```
메모리 파일: outdated_info.md
```

**용도**: 오래되거나 잘못된 정보 제거

---

### 사고 도구

#### `think_about_collected_information`
수집한 정보가 충분하고 관련 있는지 판단합니다.

**용도**:
- 검색 후 정보 충분성 확인
- 다음 단계 결정

---

#### `think_about_task_adherence`
현재 작업이 올바른 방향인지 확인합니다.

**용도**:
- 장시간 작업 시 방향 재확인
- 코드 작성 전 점검

---

#### `think_about_whether_you_are_done`
작업이 완료되었는지 판단합니다.

**용도**: 최종 완료 여부 확인

---

### 온보딩

#### `check_onboarding_performed`
프로젝트 온보딩이 수행되었는지 확인합니다.

**용도**: 프로젝트 활성화 후 온보딩 상태 확인

---

#### `onboarding`
프로젝트 온보딩 지시사항을 제공합니다.

**용도**: 프로젝트 정보 수집 및 저장

---

## 🧠 Sequential Thinking MCP

복잡한 문제를 단계적으로 해결하기 위한 사고 도구입니다.

### `sequentialthinking`

**용도**:
- 복잡한 다단계 작업
- 문제 해결 계획 수립
- 추론 과정 추적

**파라미터**:
- `thought`: 현재 사고 단계
- `thoughtNumber`: 현재 단계 번호
- `totalThoughts`: 총 예상 단계 수
- `nextThoughtNeeded`: 다음 단계 필요 여부
- `isRevision`: 수정 여부
- `branchFromThought`: 분기 시작점

**특징**:
- 단계별 사고 과정 기록
- 수정 및 분기 가능
- 가설 생성 및 검증

**사용 시기**:
- 복잡한 코드 생성
- 아키텍처 결정
- 다단계 리팩토링

---

## 🔧 IDE MCP

IDE 진단 정보를 제공합니다.

### `getDiagnostics`

**용도**:
- 컴파일 오류 확인
- 경고 메시지 확인
- 코드 문제 진단

**예시**:
```
URI: file:///path/to/BoardController.java
```

---

## 💡 실전 사용 예시

### 예시 1: 새 메서드 추가

```markdown
1. get_symbols_overview로 클래스 구조 파악
2. find_symbol로 기존 메서드 확인
3. insert_after_symbol로 새 메서드 추가
4. getDiagnostics로 오류 확인
```

### 예시 2: 리팩토링

```markdown
1. find_symbol로 변경할 메서드 찾기
2. find_referencing_symbols로 사용처 확인
3. replace_symbol_body로 메서드 수정
4. 또는 rename_symbol로 이름 변경
```

### 예시 3: 코드 탐색

```markdown
1. list_dir로 패키지 구조 확인
2. find_file로 Controller 찾기
3. get_symbols_overview로 API 목록 파악
4. find_symbol로 특정 메서드 상세 확인
```

### 예시 4: 복잡한 작업

```markdown
1. sequentialthinking으로 작업 계획 수립
2. 각 단계별로 심볼 도구 활용
3. think_about_task_adherence로 중간 점검
4. think_about_whether_you_are_done으로 완료 확인
```

---

## 🎯 활용 팁

1. **심볼 기반 접근**: 파일 전체보다는 심볼 단위로 작업하면 효율적입니다.

2. **참조 추적**: 리팩토링 시 `find_referencing_symbols`로 영향 범위를 먼저 파악하세요.

3. **메모리 활용**: 프로젝트 구조나 규칙을 메모리에 저장해 재사용하세요.

4. **단계적 사고**: 복잡한 작업은 `sequentialthinking`으로 계획을 먼저 세우세요.

5. **검증 습관**: 편집 후 `getDiagnostics`로 즉시 오류를 확인하세요.

---

**참고**: MCP 도구들은 Claude Code가 자동으로 활용하므로, 직접 호출할 필요는 없습니다. 이 문서는 Claude가 어떤 도구를 사용하는지 이해하기 위한 것입니다.
