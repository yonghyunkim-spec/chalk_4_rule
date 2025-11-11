# 코드 스타일 가이드

chalk-admin 프로젝트의 핵심 코딩 규칙과 패턴을 정리한 문서입니다.

## 🚫 절대 금지사항

다음 패턴들은 **절대 사용하지 마세요**:

1. **@Query 애노테이션** → JPA naming convention 사용
2. **Native Query** → QueryDSL 사용
3. **JPA Criteria API** → QueryDSL 사용
4. **수동 매핑 (toVo 메서드)** → MapStruct 사용
5. **static final 상수** → Enum 사용
6. **Entity 직접 반환** → VO 반환
7. **GET 요청에 @RequestBody** → @ModelAttribute 사용
8. **복합키 필드 접근 시 언더스코어** → camelCase 연결

## ✅ 필수 사용사항

1. **PagingParam 상속** - 모든 SearchParam 클래스
2. **QueryDSL** - 복잡한 쿼리 (QueryRepository)
3. **MapStruct** - 모든 Entity/VO 매핑
4. **Enum** - 모든 상수 값
5. **VO 반환** - 모든 API Response
6. **JPA Auditing** - @CreatedDate, @LastModifiedDate

## 📦 Repository 패턴

### JpaRepository (간단한 쿼리)

```java
@Repository
public interface BoardRepository extends JpaRepository<Board, Integer> {
    // JPA naming convention 사용
    List<Board> findAllByBoardType(BoardType boardType);
    Optional<Board> findByIdAndStatus(int id, BoardStatus status);
}
```

**복합키 패턴** (menu.repository 참조):
```java
// questionChapterId.questionId → QuestionChapterIdQuestionId
List<QuestionChapter> findAllByQuestionChapterIdQuestionId(Long questionId);
void deleteAllByQuestionChapterIdQuestionId(Long questionId);
```

### QueryRepository (복잡한 쿼리)

**중요**: 인터페이스 없이 구현체만 작성!

```java
@Repository
@RequiredArgsConstructor
public class BoardQueryRepository {
    private final JPAQueryFactory jpaQueryFactory;

    public List<BoardSimpleVo> getBoardList(BoardSearchParam searchParam, Pageable pageable) {
        return jpaQueryFactory.select(Projections.fields(BoardSimpleVo.class,
                        board.id,
                        board.title,
                        board.status,
                        board.createdAt))
                .from(board)
                .where(
                        eqStatus(searchParam.getBoardStatus()),
                        keywordMatch(searchParam.getSearchValue())
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .orderBy(board.createdAt.desc())
                .fetch();
    }

    public Long getBoardListCount(BoardSearchParam searchParam) {
        return jpaQueryFactory.select(board.count())
                .from(board)
                .where(
                        eqStatus(searchParam.getBoardStatus()),
                        keywordMatch(searchParam.getSearchValue())
                )
                .fetchOne();
    }

    // 헬퍼 메서드
    private BooleanExpression eqStatus(BoardStatus status) {
        return ObjectUtils.isEmpty(status) ? null : board.status.eq(status);
    }

    private BooleanExpression keywordMatch(String value) {
        return !hasText(value) ? null : board.title.containsIgnoreCase(value);
    }
}
```

## 🔄 MapStruct 패턴

```java
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface BoardMapper {

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateBoardFromParam(BoardModParam param, @MappingTarget Board board);

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    BoardVo updateBoardToVo(Board board);
}
```

## 📄 SearchParam 패턴

**반드시 PagingParam을 상속**:

```java
@Getter
@Setter
public class BoardSearchParam extends PagingParam {
    // Enum 필터
    private BoardType boardType;
    private BoardStatus boardStatus;

    // 키워드 검색
    private String searchValue;
}
```

## 🎯 Service 패턴

```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class BoardService {

    private final BoardRepository boardRepository;
    private final BoardQueryRepository boardQueryRepository;
    private final BoardMapper boardMapper;

    // 단일 조회 - VO 반환
    public BoardVo getBoard(int boardId) {
        Board board = retrieveBoard(boardId);
        return boardMapper.updateBoardToVo(board);
    }

    // 리스트 조회 - VO 반환
    public List<BoardSimpleVo> getBoardList(BoardSearchParam searchParam) {
        return boardQueryRepository.getBoardList(searchParam, searchParam.of());
    }

    // 수정 - VO 반환
    @Transactional(rollbackFor = {Exception.class})
    public BoardVo modBoard(BoardModParam modParam) {
        Board board = retrieveBoard(modParam.getBoardId());
        boardMapper.updateBoardFromParam(modParam, board);
        boardRepository.save(board);
        return boardMapper.updateBoardToVo(board);
    }

    // 헬퍼 메서드
    private Board retrieveBoard(int boardId) {
        return boardRepository.findById(boardId)
                .orElseThrow(() -> new RestException(ErrorCode.BOARD_NOT_EXIST));
    }
}
```

## 🎮 Controller 패턴

```java
@Slf4j
@RestController
@RequestMapping("/admin")
@RequiredArgsConstructor
public class BoardController {

    private final BoardService boardService;

    // GET - @ModelAttribute 사용
    @GetMapping("/v1/boards/list")
    public ResponseEntity<?> getBoardList(@ModelAttribute BoardSearchParam searchParam,
                                         @AuthenticationPrincipal AdminPrincipal adminPrincipal) {
        adminPrincipal.checkPrivilege(AdminPrivilege.BOARD_MANAGE);
        return ResponseEntity.ok(boardService.getBoardList(searchParam));
    }

    // POST - @RequestBody 사용
    @PostMapping("/v1/boards")
    public ResponseEntity<?> addBoard(@RequestBody BoardAddParam addParam,
                                     @AuthenticationPrincipal AdminPrincipal adminPrincipal) {
        adminPrincipal.checkPrivilege(AdminPrivilege.BOARD_MANAGE);
        addParam.setAdminId(adminPrincipal.getAdminId());
        return ResponseEntity.ok(boardService.addBoard(addParam));
    }
}
```

## 📚 참조 패키지

프로젝트 내 패턴을 따를 때는 아래 패키지들을 참조하세요:

1. **board 패키지** - 모든 패턴의 기준
   - QueryRepository 구현
   - MapStruct 사용
   - PagingParam 상속
   - Enum 사용
   - VO 직접 반환

2. **menu.repository 패키지** - 복합키 패턴
   - `AdminRolePrivilegeRepository`
   - camelCase 필드 연결

3. **question 패키지** - 복잡한 관계
   - Origin/Twin 관계
   - 참조 무결성
   - Cascade 처리

## 🎨 패키지 구조

모든 도메인 패키지는 다음 구조를 따릅니다:

```
{domain}/
├── constants/      # Enum만 (static final 금지)
├── model/          # JPA Entity
├── vo/             # Response VO (Vo + SimpleVo)
├── param/          # Request Parameters (SearchParam, AddParam, ModParam)
├── repository/     # JpaRepository + QueryRepository
├── service/        # Business Logic
├── controller/     # REST API
└── util/           # MapStruct Mapper
```

## 💡 핵심 원칙 요약

1. **QueryDSL로 VO 직접 반환** - Entity 반환 후 변환 금지
2. **MapStruct로 자동 매핑** - 수동 매핑 금지
3. **JPA naming convention** - @Query 금지
4. **Enum으로 상수 관리** - static final 금지
5. **PagingParam 상속** - 모든 SearchParam
6. **GET은 @ModelAttribute** - @RequestBody 금지
7. **복합키는 camelCase 연결** - 언더스코어 금지
8. **QueryRepository는 구현체만** - 인터페이스 불필요

---

**참고**: 더 상세한 예시는 `board` 패키지를 직접 참조하세요.
