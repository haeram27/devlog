# Permutation

- Permutation - Number of cases with consideration of order
- 순열 - 순서를 고려한 경우의 수(부분집합), {a,b} != {b,a}
- 재귀(Recursion)와 백트래킹(Backtracking)을 사용
- 재귀를 사용하는 이유는 선택지 마다 depth가 고정되어 있지 않기 때문임, 재귀에 사용되는 인자를 특정 stack 자료구조에 담을 수 있다면 재귀는 stack을 사용하는 것과 같음

```java
import java.util.Arrays;
import java.util.HashSet;
import java.util.stream.Collectors;

import org.junit.jupiter.api.Test;

import com.example.sample.EvaluatedTimeTests;
public class PermutationTests extends EvaluatedTimeTests {
    <T> void printa(T[] a) {
        for (var e : a) {
            System.out.print(e + " ");
        }
        System.out.println();
    }

    /**
    - @brief permutation
    *
    - @tparam T
    - @param data      element list (size >= r)
    - @param out       result of combination (size = r)
    - @param r         round (number of output elements)
    - @param depth     index of out[](permutation), if depth==2 then 0~(depth-1) index is filled in out[]
    -                  index i on for() statement is index of data[] to be candidate of out[depth]
    - @param visited   mark consumed index of data[] while traversal, visited is used to prevent choice of data[index] already consumed
    */
    public <T> void permutation(T[] data, T[] out, int r, int depth, boolean[] visited) {
        //System.out.println(String.format("depth=%d, start=%d", depth, start));
        if (depth == r) {
            printa(out);
            return;
        }

        /**
         - depth : index of out[], level of permutation selecting tree
         - i : index for data[] to be out[depth], i can be controlled by for() and visited[]
         - visited[i] : if true then data[i] can NOT be used in next depth 
         -              if false then data[i] can be used in previous/currrent depth
         */
        for (int i = 0; i < data.length; i++) {
            if (!visited[i]) {
                out[depth] = data[i];
                visited[i] = true; // data[i] can NOT be selected in next depth
                permutationA(data, out, r, depth + 1, visited);
                visited[i] = false; // data[i] can be selected again in previous depth
            }
        }
    }

    @Test
    public void run() {
        int r = 2;
        // permutation = {a, b}, {a, c}, {a, d}, {b, a}, {b, c}, {b, d}, {c, a}, {c, b}, {c, d}, {d, a}, {d, b}, {d, c}
        String[] data = {"a", "b", "c", "d"}; // n = 4
        String[] out = new String[r];
        boolean[] visited = new boolean[data.length];
        Arrays.fill(visited, false);

        System.out.println("\n[permutation recursive]====");

        /* !!!! combination caller !!!! */
        permutationA(data, out, r, 0, visited);
    }
}
```

Result:

```text
[permutation recursive]====
a b 
a c 
a d 
b a 
b c 
b d 
c a 
c b 
c d 
d a 
d b 
d c 
```

이 메서드는 **"사용하지 않은 모든 요소 중에서 하나를 선택하고, 다음 자리로 이동한 뒤, 돌아와서 다른 선택을 시도하는"** 방식으로 동작합니다.

## 메서드 시그니처와 파라미터의 역할

```java
public <T> void permutation(T[] data, T[] out, int r, int depth, boolean[] visited)
```

- `T[] data`: 원본 데이터 배열입니다. (예: `{"a", "b", "c", "d"}`)
- `T[] out`: 하나의 완성된 순열을 저장할 임시 배열입니다. 크기는 `r`입니다.
- `int r`: 뽑아서 나열할 요소의 개수입니다.
- `int depth`: 현재 재귀의 깊이, 즉 `out` 배열의 몇 번째 자리를 채울 차례인지를 의미합니다.
- `boolean[] visited`: **순열을 구하는 데 가장 핵심적인 파라미터입니다.** `data` 배열의 각 요소가 **현재 만들고 있는 순열에서 이미 사용되었는지**를 기록하는 배열입니다.

---

## 코드 실행 단계별 분석

`permutationA({"a", "b", "c"}, out, 2, 0, visited)` 호출을 예로 들어보겠습니다.

### 1. 종료 조건 (Base Case)

```java
if (depth == r) {
    printa(out);
    return;
}
```

- **의미:**
  - "만약 `r`개의 자리를 모두 채웠다면(`depth`가 `r`에 도달했다면), 하나의 순열이 완성된 것이다."
- **동작:**
  1. `out` 배열에 완성된 순열을 출력합니다.
  2. `return`을 통해 현재 재귀 호출을 종료하고, 이전 단계로 돌아가 다른 선택지를 탐색합니다(백트래킹).

### 2. 재귀 단계 (Recursive Step)

```java
for (int i = 0; i < data.length; i++) {
    if (!visited[i]) {
        // 1. 선택 (Choose)
        out[depth] = data[i];
        visited[i] = true;
        
        // 2. 탐색 (Explore)
        permutationA(data, out, r, depth + 1, visited);
        
        // 3. 복구 (Unchoose / Backtrack)
        visited[i] = false;
    }
}
```

이 부분이 백트래킹의 핵심 로직입니다.

- **`for (int i = 0; ...)`**: 순열은 순서가 중요하므로, 매번 **모든 요소를 처음부터 다시 후보로** 고려해야 합니다. (조합과의 가장 큰 차이점)

- **`if (!visited[i])`**: "만약 `data[i]` 요소가 현재 만들고 있는 순열에서 아직 사용되지 않았다면" 이라는 조건입니다. 이를 통해 `(a, a)`와 같은 중복 사용을 방지합니다.

- **`visited[i] = true;` / `out[depth] = data[i];` (선택)**
  1.  `data[i]`를 사용하기로 결정했으므로, `visited` 배열에 "사용 중"이라고 표시합니다.
  2.  `out` 배열의 현재 `depth` 자리에 `data[i]`를 넣습니다.

- **`permutationA(..., depth + 1, ...);` (탐색)**
  - 현재 선택을 바탕으로 다음 자리를 채우기 위해 재귀 호출을 합니다.
  - `depth`를 1 증가시켜 다음 자리로 넘어갑니다.

- **`visited[i] = false;` (복구 / 백트래킹)**
  - **가장 중요한 부분입니다.** `permutationA` 재귀 호출이 끝나고 돌아오면, 다음 `for` 루프 반복으로 넘어가기 전에 `visited[i]`를 `false`로 되돌려 놓습니다.
  - **이유:** `(a, b)` 순열을 만든 후, `(b, ?)`로 시작하는 새로운 순열을 만들기 위해서는 'a'가 "사용되지 않은" 상태여야 합니다. 이처럼 다른 재귀 경로(branch)에서 해당 요소를 다시 사용할 수 있도록 상태를 원상 복구하는 과정이 바로 백트래킹입니다.

## 시각적 흐름 (`r=2`, `data={"a","b","c"}`)

```text
perm(..., depth=0)
  |
  +-- i=0: 'a' 선택. visited[0]=true. out[0]='a'
  |   |
  |   +-- perm(..., depth=1)
  |       |
  |       +-- i=0: visited[0] is true -> skip
  |       +-- i=1: 'b' 선택. visited[1]=true. out[1]='b'
  |       |   |
  |       |   +-- perm(..., depth=2) -> depth==r -> print {"a","b"}
  |       |
  |       +-- 'b' 복구. visited[1]=false.
  |       |
  |       +-- i=2: 'c' 선택. visited[2]=true. out[1]='c'
  |       |   |
  |       |   +-- perm(..., depth=2) -> depth==r -> print {"a","c"}
  |       |
  |       +-- 'c' 복구. visited[2]=false.
  |
  +-- 'a' 복구. visited[0]=false.
  |
  +-- i=1: 'b' 선택. visited[1]=true. out[0]='b'
      |
      +-- perm(..., depth=1)
          |
          +-- i=0: 'a' 선택. visited[0]=true. out[1]='a'
          |   |
          |   +-- perm(..., depth=2) -> depth==r -> print {"b","a"}
          |
          +-- 'a' 복구. visited[0]=false.
          ... (이하 생략)
```

이처럼 `visited` 배열을 이용한 "선택-탐색-복구" 사이클을 통해 모든 가능한 순서를 빠짐없이 탐색하게 됩니다.