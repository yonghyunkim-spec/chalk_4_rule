# 도메인 패키지 생성 규칙

## 📋 개요

DDL (CREATE TABLE)을 기반으로 완전한 CRUD 기능을 갖춘 도메인 패키지를 생성하는 규칙을 정의합니다.

## 🎯 적용 대상

- `/generate-domain` 슬래시 커맨드
- `/sync-ddl-changes` 슬래시 커맨드

## 📁 생성되는 패키지 구조

```
src/main/java/com/firsthabit/chalk/{domain}/
├── constants/          # Enum 상수들
├── model/             # Entity
├── vo/                # Response VO
├── param/             # Request Parameters
├── repository/        # JPA + QueryDSL
├── service/           # Business Logic
├── controller/        # REST API
└── util/              # MapStruct Mapper
```

## 🚨 필수 기술 스택

### 필수 사용
- ✅ **QueryDSL** (복잡한 쿼리)
- ✅ **MapStruct** (모든 매핑)
- ✅ **Enum** (모든 상수)
- ✅ **VO 반환** (Entity 직접 반환 금지)
- ✅ **PagingParam 상속** (모든 SearchParam)
- ✅ **JPA Auditing** (@CreatedDate, @LastModifiedDate)
- ✅ **QueryRepository는 구현체만** (인터페이스 없이)

### 절대 금지
- ❌ **@Query 애노테이션** → JPA naming convention 사용
- ❌ **Native Query** → QueryDSL 사용
- ❌ **JPA Criteria API** → QueryDSL 사용
- ❌ **수동 매핑** → MapStruct 사용
- ❌ **static final 상수** → enum 사용
- ❌ **Entity 직접 반환** → VO 반환
- ❌ **GET 요청에 @RequestBody** → @ModelAttribute 사용
- ❌ **복합키 필드 접근 시 언더스코어** → camelCase 연결

## 🗃️ DDL → Java 타입 매핑

### 기본 타입 매핑 테이블

| DB 타입 | Java 타입 | 조건 |
|---------|-----------|------|
| INT | int | PK 또는 NOT NULL |
| INT | Integer | NULL 허용 |
| BIGINT | long | PK 또는 NOT NULL |
| BIGINT | Long | NULL 허용 |
| VARCHAR | String | - |
| TEXT | String | - |
| CHAR | String | - |
| DATETIME | LocalDateTime | - |
| TIMESTAMP | LocalDateTime | - |
| DATE | LocalDate | - |
| DECIMAL | BigDecimal | - |
| TINYINT | Enum | status, type, category, flag 등 |

### Enum 식별 규칙

**다음 조건을 만족하면 Enum**:
- 컬럼명에 `type`, `status`, `category`, `flag` 포함
- TINYINT 타입
- 예: `board_type` → `BoardType` enum

## 🚨 Enum 처리 (매우 중요)

### 절대 규칙

❌ **절대 Enum 값을 추론하지 마세요**
✅ **반드시 사용자에게 질문하세요**

### 질문 템플릿

```markdown
{테이블명} 테이블에서 다음 Enum 컬럼들을 발견했습니다:

1. `{column1}` (TINYINT) → {Domain}{ColumnName} enum
2. `{column2}` (TINYINT) → {Domain}{ColumnName2} enum

각 Enum의 값을 알려주세요:

**예시 형식**:
- {Domain}{ColumnName}: VALUE1=0, VALUE2=1, VALUE3=2
- {Domain}{ColumnName2}: ACTIVE=0, INACTIVE=1

**참고**:
- 값=숫자 형식으로 알려주세요
- 값이 없으면 소스 생성을 중단합니다
```

### Enum 생성 템플릿

```java
package com.firsthabit.chalk.{domain}.constants;

import lombok.Getter;

@Getter
public enum {Domain}{ColumnName} {
    {VALUE1}({0}, "{설명1}"),
    {VALUE2}({1}, "{설명2}"),
    ;

    private final int type;
    private final String description;

    {Domain}{ColumnName}(int type, String description) {
        this.type = type;
        this.description = description;
    }
}
```

## 📝 Entity 생성 규칙

### 기본 구조

```java
package com.firsthabit.chalk.{domain}.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import com.firsthabit.chalk.{domain}.constants.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;
import java.math.BigDecimal;

@Data
@Builder
@Entity(name = "{table_name}")
@NoArgsConstructor
@AllArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class {Domain} {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private {pkType} {pkField};

    // 일반 컬럼
    @Column(name = "{column_name}")
    private {JavaType} {fieldName};

    // Enum 컬럼
    @Enumerated(EnumType.ORDINAL)
    @Column(name = "{column_name}", nullable = false)
    private {EnumType} {fieldName};

    // 타임스탬프
    @CreatedDate
    @Column(nullable = false, updatable = false, name = "created_at")
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    // created_by_id, updated_by_id가 있으면 추가
    @Column(name = "created_by_id")
    private Integer createdById;

    @Column(name = "updated_by_id")
    private Integer updatedById;
}
```

### 중요 사항
- `@EntityListeners(AuditingEntityListener.class)` 필수
- Enum은 `@Enumerated(EnumType.ORDINAL)` 사용
- `@Column(name = ...)` snake_case 유지
- nullable = false인 컬럼은 필수 표시

## 📊 Repository 패턴

### JpaRepository (Simple queries)

```java
package com.firsthabit.chalk.{domain}.repository;

import com.firsthabit.chalk.{domain}.model.{Domain};
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface {Domain}Repository extends JpaRepository<{Domain}, {PkType}> {
    // JPA naming convention만 사용 (@Query 금지!)

    // 예시: 특정 enum으로 조회
    List<{Domain}> findAllBy{EnumField}({EnumType} {enumField});
}
```

**JPA Naming Convention 규칙**:
- 복합키 필드 접근: camelCase 연결
- 예: `findAllByQuestionChapterIdQuestionId(Long questionId)`
- ❌ `findAllByQuestionChapterId_QuestionId` (언더스코어 금지)

### QueryRepository (Complex queries)

**⚠️ 중요: 구현체만 작성 (인터페이스 없이)**

```java
package com.firsthabit.chalk.{domain}.repository;

import com.querydsl.core.types.Projections;
import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import lombok.RequiredArgsConstructor;
import com.firsthabit.chalk.{domain}.constants.*;
import com.firsthabit.chalk.{domain}.param.{Domain}SearchParam;
import com.firsthabit.chalk.{domain}.vo.{Domain}SimpleVo;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Repository;
import org.springframework.util.ObjectUtils;

import java.util.List;

import static com.firsthabit.chalk.{domain}.model.Q{Domain}.{domain};
import static org.flywaydb.core.internal.util.StringUtils.hasText;

@Repository
@RequiredArgsConstructor
public class {Domain}QueryRepository {
    private final JPAQueryFactory jpaQueryFactory;

    public List<{Domain}SimpleVo> get{Domain}List({Domain}SearchParam searchParam, Pageable pageable) {
        return jpaQueryFactory.select(Projections.fields({Domain}SimpleVo.class,
                        {domain}.{pkField},
                        {domain}.{mainField},
                        {domain}.{statusField},
                        {domain}.createdAt,
                        {domain}.updatedAt))
                .from({domain})
                .where(
                        eq{EnumField}(searchParam.get{EnumField}()),
                        keywordMatch(searchParam.getSearchValue())
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .orderBy({domain}.createdAt.desc())
                .fetch();
    }

    public Long get{Domain}ListCount({Domain}SearchParam searchParam) {
        return jpaQueryFactory.select({domain}.count())
                .from({domain})
                .where(
                        eq{EnumField}(searchParam.get{EnumField}()),
                        keywordMatch(searchParam.getSearchValue())
                )
                .fetchOne();
    }

    // 헬퍼 메서드들
    private BooleanExpression eq{EnumField}({EnumType} {enumField}) {
        return ObjectUtils.isEmpty({enumField}) ? null : {domain}.{enumField}.eq({enumField});
    }

    private BooleanExpression keywordMatch(String value) {
        if (!hasText(value)) {
            return null;
        }
        return {domain}.{mainField}.containsIgnoreCase(value);
    }
}
```

**QueryRepository 규칙**:
- ❌ 인터페이스 작성 금지
- ✅ `@Repository` 어노테이션 필수
- ✅ `JPAQueryFactory` 주입
- ✅ VO 반환 (Entity 반환 금지)
- ✅ `Projections.fields()` 사용
- ✅ BooleanExpression 헬퍼 메서드 작성

## 🗺️ MapStruct 패턴

```java
package com.firsthabit.chalk.{domain}.util;

import com.firsthabit.chalk.{domain}.model.{Domain};
import com.firsthabit.chalk.{domain}.param.{Domain}ModParam;
import com.firsthabit.chalk.{domain}.vo.{Domain}Vo;
import org.mapstruct.*;

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface {Domain}Mapper {

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void update{Domain}FromParam({Domain}ModParam param, @MappingTarget {Domain} {domain});

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    {Domain}Vo update{Domain}ToVo({Domain} {domain});
}
```

**MapStruct 규칙**:
- ✅ Interface (NOT class)
- ✅ `componentModel = "spring"`
- ✅ `unmappedTargetPolicy = ReportingPolicy.IGNORE`
- ✅ `nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE`
- ❌ @Mapping 애노테이션 최소화 (필드명 일치 시 자동 매핑)

## 🔧 ErrorCode, AdminAction, AdminPrivilege 추가

### ErrorCode 추가

```java
// src/main/java/com/firsthabit/chalk/config/error/ErrorCode.java
{DOMAIN}_NOT_EXIST(404, "존재하지 않는 {한글명}입니다."),
```

### AdminAction 추가

```java
// src/main/java/com/firsthabit/chalk/admin/constants/AdminAction.java
ADD_{DOMAIN}({nextCode}, "{한글명} 추가"),
MOD_{DOMAIN}({nextCode+1}, "{한글명} 수정"),
DEL_{DOMAIN}({nextCode+2}, "{한글명} 삭제"),
```

### AdminPrivilege 추가

```java
// src/main/java/com/firsthabit/chalk/admin/constants/AdminPrivilege.java
{DOMAIN}_MANAGE("{domain}_manage", "{한글명} 관리"),
```

## 📋 검증 체크리스트

### 파일 생성 확인
- [ ] constants/ 디렉토리에 모든 Enum 생성됨
- [ ] model/{Domain}.java 생성됨
- [ ] vo/{Domain}Vo.java, {Domain}SimpleVo.java 생성됨
- [ ] param/ 디렉토리에 SearchParam, AddParam, ModParam 생성됨
- [ ] repository/ 디렉토리에 Repository, QueryRepository 생성됨
- [ ] util/{Domain}Mapper.java 생성됨
- [ ] service/{Domain}Service.java 생성됨
- [ ] controller/{Domain}Controller.java 생성됨

### 코드 품질 확인
- [ ] PagingParam을 상속받았는가?
- [ ] QueryDSL을 사용했는가?
- [ ] MapStruct를 사용했는가?
- [ ] Enum을 사용했는가? (static final X)
- [ ] VO를 반환하는가? (Entity 반환 X)
- [ ] @Query 애노테이션을 사용하지 않았는가?
- [ ] GET 요청에 @ModelAttribute를 사용했는가?
- [ ] ErrorCode, AdminAction, AdminPrivilege가 추가되었는가?

### Enum 관련 확인
- [ ] 모든 Enum 값을 사용자에게 질문했는가? ⚠️
- [ ] Enum 값을 추론하지 않았는가? ⚠️
- [ ] Enum 파일이 정확히 생성되었는가?

## 🔗 참조 문서

- **슬래시 커맨드**: `.claude/commands/generate-domain.md`
- **참조 패키지**: `src/main/java/com/firsthabit/chalk/board/`
- **코드 스타일**: `guide/CODE_STYLE_GUIDELINES.md`
- **개발 프로세스**: `guide/DEVELOPMENT_WORK_INSTRUCTIONS.md`

---

**중요**: 이 규칙을 **100% 준수**해야 합니다. 특히 Enum 값 추론 금지 규칙은 절대적입니다.
