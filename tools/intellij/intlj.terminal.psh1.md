# IntelliJ Terminal: Powershell 설정

메뉴: Settings > Tools > Terminal

## 실행 환경 변수 지정

터미널에서 어플리케이션이 사용할 환경 변수 지정 e.g. 프록시 또는 어플리케이션이 환경 변수

여러개 일 경우 ';'으로 구분

- Project Settings > Environment variables: `HTTP_PROXY=http://1.2.3.4:3128;NODE_EXTRA_CA_CERTS=C:\path\to\cert.crt`

## 쉘 실행 파일 지정

인텔리제이 쉘의 경우, Windows Terminal과 달리 초기화 profile

- Application Settings >  Shell Path: `"C:\Program Files\PowerShell\7\pwsh.exe" -NoExit -Command ". $HOME\Documents\PowerShell\Microsoft.PowerShell_profile.ps1"`