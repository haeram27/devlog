# ubuntu에 vscode 설치하기

## install script
```bash
# 1. 필수 패키지 및 Microsoft GPG 키 추가
sudo apt install -y wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo rm -f /etc/apt/keyrings/packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
rm -f packages.microsoft.gpg

# 2. apt 저장소 등록
sudo vi /etc/apt/sources.list.d/vscode.sources
#Types: deb
#URIs: https://packages.microsoft.com/repos/code
#Suites: stable
#Components: main
#Architectures: amd64
#Signed-By: /etc/apt/keyrings/packages.microsoft.gpg

# 3. 설치
sudo apt update
sudo apt install code
```