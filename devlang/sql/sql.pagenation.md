
# SQL Pageantion

## `OFFSET` pagenation

보편적으로 가장 많이 사용되는 pagenation 방식

- [Postgresql - LIMIT, OFFSET](https://www.postgresql.org/docs/current/queries-limit.html)
- {page_number} : 페이지 번호(>0), 1부터 시작
- {page_size} : 한 페이지 당 데이터 개수(>0)

- 일반 `OFFSET` pagenation query syntax

    ```sql
    SELECT * FROM {table}
    ORDER BY {column}
    LIMIT {page_size}
    OFFSET ({page_number}-1)*{page_size}
    ```

- 성능 개선형: `커버링 인덱스 OFFSET` pagenation query syntax: 전체 데이터를 읽지 않고 정렬된 index만 먼저 계산하므로 데이터 로딩 속도가 일반 방식 보다 상대적으로 빨라 성능이 개선됨

    ```sql
    SELECT t.*
    FROM {table} t
    JOIN (
        SELECT id
        FROM {table}
        ORDER BY id
        LIMIT {page_size}
        OFFSET ({page_number}-1)*{page_size}
    ) AS sub ON t.id = sub.id
    ORDER BY t.id;
    ```

- 랜덤 페이지 조회 가능: web이나 restapi 등에서 특정 페이지를 조회하는 로직에서 많이 사용된다.
- `OFFSET`을 사용할 때는 `ORDER BY`를 반드시 함께 사용: SQL 표준과 PostgreSQL에서 테이블 내 데이터는 기본적으로 순서가 없는 집합(unordered set)이다. `ORDER BY`를 지정하지 않으면 데이터베이스는 가장 효율적이라고 판단하는 순서로 데이터를 반환하는데, 이는 쿼리를 실행할 때마다 달라질 수 있으며 `ORDER BY`가 없이 `OFFSET` 방식 pagenation을 사용한다면 데이터의 중복 또는 누락이 발생한다.
- 데이터가 많을수록 성능 저하 발생: 데이터 양이 매우 많을 경우(수백만 건 이상), OFFSET 방식은 뒤로 갈수록 속도가 느려지므로 성능 최적화가 필요할 수 있다. OFFSET 방식은 특정 지점까지 데이터를 건너 뛰는게 아니라 상위 데이터를 모두 디스크로 부터 읽은 후 버리는 방식으로 하위 데이터까지 접근하기 때문이다. 인덱스만 먼저 연산하는 `커버링 인덱스` 성능 개선 방식을 사용하는 것을 추천한다. 그래도 성능이 너무 느리다면 `Key set` pagenation을 궁극적으로 고려해 봐야 한다.
- 고유한 정렬 기준 사용: 페이지네이션을 구현할 때는 가급적 중복되지 않는 값(예: Primary Key, id)을 `ORDER BY`에 포함시켜야 항상 동일한 순서를 보장받을 수 있다.
- 성능 최적화: 정렬할 컬럼에 인덱스가 생성되어 있으면 `ORDER BY`를 사용하더라도 성능 저하 없이 빠르게 데이터를 가져올 수 있다.

## `Key Set` pagenation

- 대규모 데이터(수백만개 이상) 상황에서 성능 저하를 낮추는 궁극적인 방식
- syntax :

    ```sql
    SELECT *
    FROM {table}
    WHERE id > {lastSeenId}  -- 여기서 lastSeenId는 이전 페이지의 마지막 row ID
    ORDER BY id ASC
    LIMIT {pageSize}
    ```
    주의: `{lastSeenId}`는 application에서 확인 후 전달해 해야 한다.

- 각 페이지 요청 시 파라미터 (lastSeenId)

    ```text
    1페이지 (첫 요청 시)
        lastSeenId = 0 (ID가 1부터 시작하므로 0보다 큰 것부터 가져옴)
        LIMIT = pageSize
    2페이지 (이후 요청 시)
        lastSeenId = 1페이지의 마지막 데이터 ID
        LIMIT = pageSize
    N페이지
        lastSeenId = (N-1)페이지의 마지막 데이터 ID
        LIMIT = pageSize
    ```

- 성능: Key-set은 인덱스를 타고 바로 다음 지점으로 점프하므로 `OFFSET` pagenation과 달리 대규모 데이터(수백만건 이상)에 따른 성능 저하 없음
- 정확성: 페이지 이동 사이에 데이터가 추가/삭제되어도 데이터가 중복되거나 누락되지 않음
- 이 방식은 "5페이지로 바로 가기" 같은 랜덤 액세스 페이지네이션에는 적합하지 않고, "더보기"나 "무한 스크롤"에 최적화되어 있음

## pagenation을 위한 SERIAL ID 관리

`OFFSET`을 이용한 pagenation 수행시 대상 table에는 `ORDER BY`를 위한 ID가 존재하는 것이 좋다.

데이터를 새로 쓸때 마다 ID를 RESET 할 수 있는 케이스라면 해주는 것이 좋다.

- 테이블의 모든 데이터를 삭제함과 동시에 연결된 시퀀스 값도 시작값(보통 1)으로 리셋

```sql
TRUNCATE TABLE <table> RESTART IDENTITY;
```
참고: 별도의 옵션을 주지 않으면 기본값은 `CONTINUE IDENTITY`로 작동하여, 테이블을 비워도 다음 데이터 삽입 시 이전 번호에 이어서 생성함.

- 이미 데이터를 삭제(DELETE)한 경우 sequence를 수동 리셋 하는 방법

```sql
-- 시퀀스 이름을 모를 경우 확인 방법
SELECT pg_get_serial_sequence('테이블명', '컬럼명');

-- 1부터 시작하도록 리셋
ALTER SEQUENCE <sequence name> RESTART WITH 1;
```

## Java List pagenation

```java
import java.util.List;

public class PagenationUtil {
    /**
     * Calculates the total number of pages based on the total count of items and the page size.
     * @param totalCount total count of items
     * @param pageSize number of items per page
     * @return total number of pages
     */
    public static int getTotalPages(int totalCount, int pageSize) {
        if (totalCount < 0) {
            // "Total count cannot be negative"
            return 0;
        }

        if (pageSize <= 0) {
            // Page size must be greater than 0
            return 0;
        }

        // Math.ceil is used to round up the total pages, ensuring that any remaining items are accounted for in an additional page
        // example: if totalCount is 10 and pageSize is 3, totalPages will be 4 (10/3 = 3.33, rounded up to 4)
        return (int) Math.ceil((double) totalCount / pageSize);
    }

    /**
     * Returns a sublist of the given list based on the specified page size and page number.
     * @param <T> the type of elements in the list
     * @param list the list to be paginated
     * @param pageSize number of items per page
     * @param pageNumber the page number to retrieve (1-based index)
     * @return a sublist containing the items for the specified page
     */
    public static <T> List<T> pagenationList(List<T> list, int pageSize, int pageNumber) {
        if (pageSize < 1 || pageNumber < 1) {
            // Page size and page number must be greater than 0
            return List.of(); // Return an empty list if there are no items 
        }

        if (list == null) {
            // "List cannot be null
            return List.of();
        }

        int totalCount = list.size();
        if (totalCount <= 0) {
            return List.of();
        }

        int startInclusive = Math.min((pageNumber - 1) * pageSize, totalCount);
        int endExclusive = Math.min(startInclusive + pageSize, totalCount);

        return list.subList(startInclusive, endExclusive);
    }
}
```