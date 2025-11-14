# 로컬 호스트 내에서 포트 포워딩

호스트 외부에서 접속이 허용되지 않는 A 포트를 외부에서 접속 하려면 두 가지 방법이 있다.

* SSH : ssh로 호스트에 접속 후 호스트 내부에서 A 포트에 연결
* port-fowrding : 포트 포워딩으로 외부에서 접속가능한 포트 B를 생성하여 외부의 접속 요청을 포트 A로 전달

다음은 동일 호스트 내에서 포트 포워딩을 하는 방법을 설명 한다.

포트 포워딩 구성

```text
req -> 8080(forward-port) -> 3000(bind-port) -> application
```

## 1. iptables를 사용한 방법

```bash
# 예: 8080 포트로 들어오는 트래픽을 3000 포트로 포워딩
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 3000

# 로컬 루프백에서도 작동하도록 설정
sudo iptables -t nat -A OUTPUT -o lo -p tcp --dport 8080 -j REDIRECT --to-port 3000
```

설정을 영구적으로 저장:

```bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

## 2. firewalld를 사용한 방법 (Rocky Linux 권장)

```bash
# 포트 포워딩 규칙 추가 (예: 8080 → 3000)
sudo firewall-cmd --zone=public --add-forward-port=port=8080:proto=tcp:toport=3000

# 영구 설정
sudo firewall-cmd --permanent --zone=public --add-forward-port=port=8080:proto=tcp:toport=3000

# 규칙 적용
sudo firewall-cmd --reload
```

## 3. socat을 사용한 방법

socat 설치:

```bash
sudo dnf install socat -y
```

syntax:

```bash
socat TCP-LISTEN:<forwarded-port>,fork TCP:localhost:<bind-port>
```

* forwarded-port:
* bind-port:

임시 포워딩:

```bash
socat TCP-LISTEN:8080,fork TCP:localhost:3000
```

systemd 서비스로 등록하여 영구 실행:

```bash
sudo tee /etc/systemd/system/port-forward.service << 'EOF'
[Unit]
Description=Port Forward 8080 to 3000
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/socat TCP-LISTEN:8080,fork,reuseaddr TCP:localhost:3000
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now port-forward.service
```

## 4. 설정 확인

```bash
# firewalld 규칙 확인
sudo firewall-cmd --list-all

# iptables 규칙 확인
sudo iptables -t nat -L -n -v

# 포트 리스닝 확인
sudo ss -tlnp | grep -E '8080|3000'
```

어떤 포트에서 어떤 포트로 포워딩하려고 하시나요? 구체적인 상황을 알려주시면 더 정확한 설정 방법을 안내해드리겠습니다.