# Combination

- Combination - Number of cases without consideration of order
- 조합 - 순서를 고려하지 않은 경우의 수(부분집합), {a,b} == {b,a}
- 재귀(Recursion)와 백트래킹(Backtracking)을 사용
- 재귀를 사용하는 이유는 선택지 마다 depth가 고정되어 있지 않기 때문임, 재귀에 사용되는 인자를 특정 stack 자료구조에 담을 수 있다면 재귀는 stack을 사용하는 것과 같음

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashSet;
import java.util.List;
import java.util.stream.Collectors;

import org.junit.jupiter.api.Test;

import com.example.sample.EvaluatedTimeTests;

public class CombinationTests extends EvaluatedTimeTests {

    <T> void printa(T[] a) {
        for (var e : a) {
            System.out.print(e + " ");
        }
        System.out.println();
    }

    /**
    * @brief combination
    *
    * @tparam T
    * @param data      element list (size >= r)
    * @param out       result of combination (size = r)
    * @param r         round (number of output elements)
    * @param depth     current level of combination, if depth==2 then 0~(depth-1) index is filled in out array 
    * @param start     start is index of data to be candidate of out[depth]
    */
    <T> void combinationA(T[] data, T[] out, int r, int depth, int start) {
        //System.out.println(String.format("depth=%d, start=%d", depth, start));
        if (depth == r) {
            printa(out);
            return;
        }

        /**
         * depth : index of out[], level of combination selecting tree
         * i : index for data[] to be out[depth], i can be controlled by for() and start
         */
        for (int i = start; i < data.length; i++) {
            // data[i] is selected for current depth
            out[depth] = data[i];
            // data[i] can NOT be select in next depth because pass i+1 as start
            combinationA(data, out, r, depth + 1, i + 1);
        }
    }

    @Test
    public void run() {
        int r = 3;
        // combination, r==2 : {{a, b}, {a,c}, {a,d}, {b,c}, {b,d}, {c,d}} 
        // combination, r==3 : {{a,b,c}, {a,b,d}, {a,c,d}, {b,c,d}}
        String[] data = {"a", "b", "c", "d"}; // n = 4
        String[] out = new String[r];

        System.out.println("\n[combination recursive]====");

        /* !!!! combination caller !!!! */
        combination(data, out, r, 0, 0);
    }
}
```

Result:

```text
[combination recursive]====
a b 
a c 
a d 
b c 
b d 
c d 
```

이 메서드는 **백트래킹(Backtracking)** 이라는 알고리즘 기법을 사용합니다. "하나를 선택하고, 그 상태에서 다음 단계를 진행한 뒤, 다시 원래 상태로 돌아와 다른 선택을 하는" 방식입니다.

## 메서드 시그니처와 파라미터의 역할

```java
<T> void combination(T[] data, T[] out, int r, int depth, int start)
```

- `T[] data`: 원본 데이터 배열입니다. (예: `{"a", "b", "c", "d"}`)
- `T[] out`: 하나의 완성된 조합을 저장할 임시 배열입니다. 크기는 `r`입니다. (예: `{"a", "b", "c"}`)
- `int r`: 뽑고자 하는 요소의 개수입니다. (예: 3개를 뽑음)
- `int depth`: 현재 재귀의 깊이를 나타냅니다. `out` 배열의 몇 번째 인덱스를 채울 차례인지를 의미하며, 0부터 시작합니다.
- `int start`: `data` 배열에서 탐색을 시작할 인덱스입니다. **조합에서 중복을 피하게 해주는 가장 핵심적인 파라미터입니다.**

---

## 코드 실행 단계별 분석

`combination({"a", "b", "c", "d"}, out, 3, 0, 0)` 호출을 예로 들어보겠습니다.

### 1. 종료 조건 (Base Case)

```java
if (r == depth) {
    printa(out);
    return;
}
```

- **의미:**
  - "만약 `r`개의 요소를 모두 뽑았다면 (`depth`가 `r`에 도달했다면), 하나의 조합이 완성된 것이다."
- **동작:**
  1. 현재까지 `out` 배열에 채워진 조합을 출력합니다.
  2. `return`을 통해 현재 재귀 호출을 종료하고, 이전 호출 지점으로 돌아갑니다(백트래킹).

### 2. 재귀 단계 (Recursive Step)

```java
for (int i = start; i < data.length; i++) {
    // 1. 선택 (Choose)
    out[depth] = data[i];
    
    // 2. 탐색 (Explore)
    combination(data, out, r, depth + 1, i + 1);
    
    // 3. 복구 (Unchoose) - 이 코드에서는 별도 복구가 필요 없음
}
```

이 `for` 루프가 백트래킹의 핵심입니다.

- **`for (int i = start; ...)`**: `data` 배열의 `start` 인덱스부터 끝까지 순회하며 현재 `depth`에 들어갈 요소를 선택합니다.
  - **`start`의 역할:** 이전에 선택한 요소의 인덱스보다 큰 인덱스에서만 다음 요소를 찾도록 범위를 제한합니다. 예를 들어, 'a'를 뽑았다면 그 다음엔 'a' 이후의 요소들만 고려하게 되어 `{b, a}`와 같은 중복 조합이 생기지 않습니다.

- **`out[depth] = data[i];` (선택)**
  - 현재 `depth` 위치에 `data[i]` 요소를 저장합니다. 첫 호출에서는 `out[0]`에 'a'가 저장됩니다.

- **`combination(..., depth + 1, i + 1);` (탐색)**
  - 다음 요소를 뽑기 위해 재귀적으로 자기 자신을 호출합니다.
  - `선택` 과정에서 현재 `depth` 위치에 `data[i]` 요소가 선택 되었으므로 다음 `depth+1`에서는 이번에 선택된 `i` 요소를 제외하고 `i+1` 부터 선택이 가능합니다.
  - `depth + 1`: 이제 `out` 배열의 다음 칸을 채울 차례라는 의미입니다.
  - `i + 1`: **가장 중요한 부분입니다.** 다음 요소는 현재 선택한 `data[i]`의 **다음 인덱스(`i+1`)부터** 찾아야 한다고 알려줍니다. 이것이 순서를 고려하지 않는 조합의 핵심입니다.

- **복구 (Unchoose)**
  - `for` 루프가 다음 반복으로 넘어가면 `out[depth] = data[i]` 라인에서 새로운 값으로 덮어쓰기 때문에, 별도의 "제거" 코드가 필요 없습니다. 예를 들어 `out[0]`에 'a'를 넣고 재귀를 다녀온 뒤, 다음 루프에서 `out[0]`에 'b'를 넣는 식으로 자연스럽게 상태가 바뀝니다.

## 시각적 흐름 (`r=3`, `data={"a","b","c","d"}`)

```text
combination(..., r=3, depth=0, start=0)
  |
  +-- i=0: out[0] = 'a'
  |   |
  |   +-- combination(..., depth=1, start=1)
  |       |
  |       +-- i=1: out[1] = 'b'
  |       |   |
  |       |   +-- combination(..., depth=2, start=2)
  |       |       |
  |       |       +-- i=2: out[2] = 'c' -> depth==r -> print {"a","b","c"}
  |       |       +-- i=3: out[2] = 'd' -> depth==r -> print {"a","b","d"}
  |       |
  |       +-- i=2: out[1] = 'c'
  |           |
  |           +-- combination(..., depth=2, start=3)
  |               |
  |               +-- i=3: out[2] = 'd' -> depth==r -> print {"a","c","d"}
  |
  +-- i=1: out[0] = 'b'
      |
      +-- combination(..., depth=1, start=2)
          ... (이하 생략)
```

이처럼 `start` 인덱스를 계속 증가시키며 재귀를 호출하는 간단한 구조만으로 순서가 없는 모든 조합을 효율적으로 찾아낼 수 있습니다.