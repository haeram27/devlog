# shell array 사용

## 배열 선언

- 괄호와 공백 사용: 가장 일반적인 방법입니다.

```bash
my_array=("apple" "banana" "cherry")
```

- declare -a 사용: 배열임을 명시적으로 선언합니다.

```bash
declare -a my_array
my_array=("apple" "banana" "cherry")
```

- 요소별로 할당: 요소를 하나씩 할당하면 배열이 자동으로 생성됩니다.

```bash
my_array[0]="apple"
my_array[1]="banana"
my_array[2]="cherry"
```

- 요소 추가: 기존 배열에 요소를 추가(+= 연산자)할 수 있습니다.

```bash
my_array=("apple" "banana")
my_array+=("cherry")
my_array+=("date")
```

## 배열 참조

- "${array[@]}": 모든 요소를 개별 단어로 확장 (quoting 사용하여 공백 포함 이름 처리 할 것)
- "${array[index_number]}": 배열의 지정 인덱스 요소
- "${!array[@]}": 배열 모든 요소의 개별 인덱스 번호로 확장
- ${#array[@]}: 배열의 크기
- 따옴표 사용 필수: "${array[@]}" (공백이 포함된 요소 처리)

```bash
# "${array[@]}": 모든 요소를 개별 단어로 확장 (공백 포함 이름 처리)
# "${!array[@]}": 배열의 모든 인덱스
# "${array[index_number]}": 배열의 지정 인덱스
# ${#array[@]}: 배열의 크기
# 따옴표 사용 필수: "${array[@]}" (공백이 포함된 요소 처리)

test_array=(a b c d e)


# lookup forward
echo -e "\n## lookup forward"
for i in "${test_array[@]}"; do
    echo ${i}
done

## lookup forward
# a
# b
# c
# d
# e

# array index lookup
echo -e "\n## lookup index"
for i in "${!test_array[@]}"; do
    echo "Index ${i}: ${test_array[$i]}"
done

## lookup index
# Index 0: a
# Index 1: b
# Index 2: c
# Index 3: d
# Index 4: e

# c-style lookup forward
echo -e "\n## c-style lookup forward"
for ((i=0; i<${#test_array[@]}; i++)); do
    echo "Element ${i}: ${test_array[$i]}"
done

## c-style lookup forward
# Element 0: a
# Element 1: b
# Element 2: c
# Element 3: d
# Element 4: e

# revser order 
echo -e "\n## c-style lookup backward"
for ((i=${#test_array[@]}-1; i>=0; i--)); do
    echo "Reverse ${i}: ${test_array[$i]}"
done

## c-style lookup backward
# Reverse 4: e
# Reverse 3: d
# Reverse 2: c
# Reverse 1: b
# Reverse 0: a
```