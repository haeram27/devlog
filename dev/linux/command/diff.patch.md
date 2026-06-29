# diff patch로 patch file 사용하기

## diff and patch

### 파일 단위 패치 생성

```bash
# -u: 통합 포맷(unified format)으로 출력 (가장 많이 쓰임)
diff -u old.txt new.txt > my_patch.patch

# 패치 적용: my_patch.patch 를 이용하여 old.txt를 new.txt와 같게 수정함
patch old.txt < my_patch.patch

# -R 옵션을 사용하면 적용했던 패치를 다시 되돌림
patch -R old.txt < my_patch.patch
```

### 디렉토리 단위 비교하여 패치 생성

```bash
# 패치 생성
diff -Naur old_dir/ new_dir/ > project.patch

# 패치 적용 old_dir이 있는 위치에서 실행
patch -p1 --dry-run < project.patch

# 적용된 패치 되돌리기
patch -p1 -R < project.patch
```

diff 옵션:

- -N: 없는 파일도 빈 파일로 간주해서 새로 생성됨을 표시
- -a: 모든 파일을 텍스트로 처리
- -u: 통합 포맷 사용
- -r: 하위 디렉토리까지 재귀적으로 탐색

patch 옵션:

- -p0: 패치 파일에 적힌 경로 그대로(old_dir/file.txt)를 찾아가서 적용합니다.
- -p1: 경로의 첫 번째 슬래시까지(old_dir/)를 무시하고 그 안의 파일명(file.txt)을 현재 디렉토리에서 찾아 적용합니다.
- --dry-run: test로 패치 적용, 파일 수정 안함

## git diff

### 패치 생성 (git diff)

```bash
# staging(old)와 working-tree(new) 비교 패치
git diff > changes.patch

# 최근 커밋(HEAD, old) vs staging(new)
git diff --staged > changes.patch

# 최근 커밋(HEAD, old) vs working-tree(new) 비교 패치
git diff HEAD

# 이전 커밋(HEAD~1, old)과 현재 커밋(HEAD, new) 비교 패치
git diff HEAD~1 HEAD > commit_diff.patch

# branch 사이의 비교 패치 (branch1이 old로 비교)
git diff branch1 branch2
```

### 패치 적용 (git apply)

```bash
git apply changes.patch
```

## git format-patch

format-patch는 커밋의 작성자, 생성일자 등 메타데이터를 포함하여 패치를 생성합니다.

### 패치 생성 (git format-patch)

```bash
# 가장 최근 커밋 1개를 패치로 생성
git format-patch -1 HEAD

# 최근 커밋 3개를 각각 0001, 0002, 0003 번호를 붙여 생성
git format-patch -3 HEAD

# [A커밋] 이후부터 [B커밋]까지의 내역을 추출
git format-patch <커밋ID_A>..<커밋ID_B>

# 특정 커밋 하나만 추출
git format-patch -1 <커밋ID>

# main 브랜치에는 없고 현재 브랜치에만 있는 커밋들을 추출
git format-patch main
```

format-patch 옵션:

- -o <디렉토리>: 패치 파일들을 특정 폴더 안에 저장합니다.
```bash
    git format-patch -3 HEAD -o ./patches
```
- --stdout > total.patch: 여러 커밋을 번호 붙은 여러 파일이 아닌, 하나의 파일로 합쳐서 생성합니다.
- -M: 파일 이름이 변경된 경우 이를 감지하여 패치에 반영합니다.

### 패치 적용 (git am)

```bash
# 단일 패치 적용
git am 0001-setup-config.patch

# 현재 디렉토리의 모든 패치 파일을 순서대로 적용
git am *.patch
```
