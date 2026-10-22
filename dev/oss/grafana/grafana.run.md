# grafana

## grafana 실행

docker run -d -p 3000:3000 --name=grafana --restart=always -v grafana-storage:/var/lib/grafana grafana/grafana

- -d: 백그라운드에서 실행 (Detach)
- -p 3000:3000: 호스트의 3000번 포트를 컨테이너의 3000번 포트와 연결
- --name=grafana: 컨테이너 이름 지정
- --restart=always: 서버 재부팅 시 자동 시작
- -v grafana-storage:/var/lib/grafana: 대시보드 및 설정 데이터 영구 보존용 볼륨 마운트

## 초기 계정