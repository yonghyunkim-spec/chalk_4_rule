# Repository 테스트 생성 규칙

## 📋 개요

Repository와 Entity를 분석하여 완전한 테스트 클래스를 자동으로 생성하는 규칙을 정의합니다.

## 🎯 적용 대상

- `/generate-repo-test` 슬래시 커맨드

## 📁 출력 경로

```
src/test/java/com/firsthabit/chalk/{domain}/repository/{Domain}RepositoryTest.java
src/test/java/com/firsthabit/chalk/{domain}/repository/{Domain}QueryRepositoryTest.java
```

## 🔍 Repository 분석

### 1. JpaRepository 찾기

```
mcp__serena__find_symbol:
- name_path: "{Domain}Repository"
- include_body: true
- depth: 1
```

### 2. QueryRepository 찾기

```
mcp__serena__find_symbol:
- name_path: "{Domain}QueryRepository"
- include_body: true
```

## 🚨 Entity 분석 (매우 중요)

### Entity 파일 읽기

```
mcp__serena__find_symbol:
- name_path: "{Domain}"
- relative_path: "src/main/java/com/firsthabit/chalk/{domain}/model"
- include_body: true
```

### 필드 정보 파악

- PK 타입 및 필드명
- 모든 필드 타입
- **nullable = false 필드 (필수 설정 필요)**
- Enum 필드

## 🚨 Enum 처리 규칙

### 절대 규칙

❌ **절대 Enum 값을 추론하지 마세요**
✅ **항상 실제 Enum 파일을 읽어야 합니다**

### Enum 파일 읽기

```
mcp__serena__find_symbol:
- name_path: "{EnumType}"
- relative_path: "src/main/java/com/firsthabit/chalk/{domain}/constants"
- include_body: true
```

### 테스트에서 사용

```java
// ✅ 올바른 예 (파일에서 읽은 값)
.visibility(ChapterVisibility.HIDDEN)

// ❌ 잘못된 예 (추론 또는 숫자)
.visibility(0)  // 컴파일 에러!
```

## 📝 테스트 템플릿

### 기본 구조

```java
package com.firsthabit.chalk.{domain}.repository;

import com.firsthabit.chalk.{domain}.constants.*;
import com.firsthabit.chalk.{domain}.model.{Domain};
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

@Transactional
@ExtendWith(SpringExtension.class)
@ActiveProfiles("test")
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class {Domain}RepositoryTest {

    @Autowired
    private {Domain}Repository {domain}Repository;

    @Test
    void save_{domain}_then_saved() {
        // Given
        {Domain} {domain} = {Domain}.builder()
                .{field1}({value1})
                .{field2}({value2})
                // ... 모든 nullable = false 필드
                .createdAt(LocalDateTime.now())  // 필수!
                .updatedAt(LocalDateTime.now())  // 필수!
                .build();

        // When
        {Domain} saved = {domain}Repository.save({domain});

        // Then
        assertNotNull(saved.get{PkField}());
        assertEquals({value1}, saved.get{Field1}());
    }
}
```

## ⚠️ 필수 규칙

### 1. 모든 nullable = false 필드 설정

```java
// ❌ 잘못된 예 (필수 필드 누락)
{Domain} {domain} = {Domain}.builder()
        .title("Test")  // name은 nullable = false인데 누락!
        .build();

// ✅ 올바른 예 (모든 필수 필드 설정)
{Domain} {domain} = {Domain}.builder()
        .title("Test")
        .name("Required Name")  // 필수!
        .createdAt(LocalDateTime.now())  // 필수!
        .updatedAt(LocalDateTime.now())  // 필수!
        .build();
```

### 2. 타임스탬프 수동 설정

**⚠️ 중요: JPA Auditing은 테스트에서 작동하지 않습니다**

```java
// ❌ JPA Auditing에 의존 (작동 안 함)
{Domain} {domain} = {Domain}.builder()
        .title("Test")
        .build();  // createdAt, updatedAt 없음 → 저장 실패!

// ✅ 수동 설정 필수
{Domain} {domain} = {Domain}.builder()
        .title("Test")
        .createdAt(LocalDateTime.now())
        .updatedAt(LocalDateTime.now())
        .build();
```

### 3. Enum 값 사용

```java
// ❌ 추론 또는 숫자 사용
.visibility(0)  // 컴파일 에러!

// ✅ 실제 Enum 사용
.visibility(ChapterVisibility.HIDDEN)
```

## 📊 QueryRepository 테스트

### 추가 설정 필요

```java
@Transactional
@ExtendWith(SpringExtension.class)
@ActiveProfiles("test")
@DataJpaTest
@Import(QueryDsLTestConfig.class)  // ⚠️ 필수!
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class {Domain}QueryRepositoryTest {

    @Autowired
    private {Domain}Repository {domain}Repository;

    @Autowired
    private {Domain}QueryRepository {domain}QueryRepository;

    @Test
    void get{Domain}List_withSearchParam_then_returns_list() {
        // Given
        {Domain} {domain}1 = {Domain}.builder()
                .{field1}({value1})
                .createdAt(LocalDateTime.now())
                .updatedAt(LocalDateTime.now())
                .build();
        {domain}Repository.save({domain}1);

        {Domain}SearchParam searchParam = new {Domain}SearchParam();
        searchParam.setPage(1);
        searchParam.setSize(10);

        // When
        List<{Domain}SimpleVo> result = {domain}QueryRepository.get{Domain}List(searchParam, searchParam.of());

        // Then
        assertNotNull(result);
        assertTrue(result.size() > 0);
    }
}
```

### QueryDsLTestConfig 업데이트

```java
// src/test/java/com/firsthabit/chalk/config/QueryDsLTestConfig.java

@Bean
public {Domain}QueryRepository {domain}QueryRepository() {
    return new {Domain}QueryRepository(jpaQueryFactory());
}
```

## 📋 검증 체크리스트

### 필수 확인 사항
- [ ] 모든 nullable = false 필드가 설정되었는가? ⚠️
- [ ] Enum 값이 정확한가? (파일에서 읽은 실제 값) ⚠️
- [ ] createdAt, updatedAt이 수동 설정되었는가? ⚠️
- [ ] Given-When-Then 패턴을 따랐는가?
- [ ] 각 테스트가 독립적인가?
- [ ] QueryRepository 테스트에 @Import(QueryDsLTestConfig.class)가 있는가?

### Enum 관련 확인
- [ ] 모든 Enum 필드에 대해 실제 Enum 파일을 읽었는가? ⚠️
- [ ] Enum 값을 추론하지 않았는가? ⚠️

## 🔗 참조 문서

- **테스트 가이드**: `guide/REPOSITORY_TEST_GUIDE.md`
- **슬래시 커맨드**: `.claude/commands/generate-repo-test.md`
- **기존 테스트**: `src/test/java/com/firsthabit/chalk/question/repository/QuestionRepositoryTest.java`

---

**중요**: Enum 파일을 읽고, 모든 필수 필드를 설정하고, 타임스탬프를 수동으로 설정해야 합니다.
