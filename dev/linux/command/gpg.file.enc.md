# gpg

- gpg는 OpenPGP의 암호화 및 사이닝 도구 이다.
- 비대칭키 암호화(-e, --encrypt)와 대칭키 암호화(-c, --symmetric)를 지원한다.
- 암호화된 파일은 .gpg 확장자를 가진다.
- 암호화 된 내용(바이너리)를 base64 인코딩된 텍스트로 출력할 수 있다. 이렇게 base64 인코딩 텍스트로 생성된 결과는 ascii armored out 이라고 한다.
- armored 출력 파일은 .asc (ascii?) 확장자를 가진다.

## gpg 로 대칭키 암호화 (-c, --symmetric)

대칭키 암호화를 하는 경우 편리하게 비밀번호(대칭키)를 사용하는 암호화 방식이 된다.

```bash
gpg [-o plain.txt.gpg] -c plain.txt
```

## gpg armored 대칭키 암호화 (-a, --armor)

```bash
gpg -ac plain.txt
```

## 복호화 하기 (-d, --decrypt)

```bash
gpg -d -o plain.txt plain.txt.gpg
gpg -d -o plain.txt plain.txt.asc

# loopback 모드(비밀번호 입력 프롬트프 사용)
gpg -d  --pinentry-mode loopback -o plain.txt ./plain.txt.gpg
```

### 대칭키 자동 캐싱 방지하기

- 대칭키 사용시 gpg는 자동으로 사용된 대칭키를 재사용하기 위해 caching 한다.
- 사용자가 암호화나 복호화시 비밀번호(대칭키)를 입력하지 않는다면 cache에 저장된 비밀번호가 사용된다.
- 캐싱된 비밀번호는 `gpg-agent` 프로세스가 메모리에 보관한다. 캐시를 삭제하려면 `gpg-agent`를 재시작 한다.
- 대칭키가 시스템에 caching되는 것을 방지 하려면 `no-symkey-cache` 옵션을 사용한다.

```bash
echo no-symkey-cache > ~/.gnupg/gpg.conf
gpg [-o comp.tgz.gpg] [--no-symkey-cache] -c comp.tgz
```

### 캐시 삭제

캐시를 삭제하려면 gpg-agent를 재시작하거나 다음 명령으로 캐시만 삭제 시도 한다.

```bash
gpg-connect-agent reloadagent /bye
```

### 비밀번호 강제 입력하기

캐시된 비밀번호를 사용하지 않고 암호화/복호화시 비밀번호를 강제 입력할 수 있다

```bash
# 비밀번호 입력 프롬프트 발생
gpg -c --pinentry-mode loopback plain.txt

# command에 비밀번호 입력
gpg -c --pinentry-mode --passphrase "1234" loopback plain.txt
```


## ssh 환경에서 사용하기

ssh 환경과 같이 GPG가 암호를 입력받기 위한 pinentry(화면 출력 도구)를 찾지 못했거나, 터미널 세션에서 암호 입력창을 띄울 권한이 없을 때 `gpg: problem with the agent: No pinentry` 오류가 발생한다.

`--pineeenty-mode loopback` 옵션을 사용하면 일시적으로 암호 입력창이 아닌 터미널 프롬프트에서 직접 암호를 입력받도록 강제한다

```bash
gpg -c --pinentry-mode loopback plain.txt
```

### loopback pineentry mode 영구 사용

gpg-agent.conf에 다음과 같이 설정

```txt
# configuration
$ vi ~/.gnupg/gpg-agent.conf
allow-loopback-pinentry

# restart agent
$ gpgconf --kill gpg-agent
```
