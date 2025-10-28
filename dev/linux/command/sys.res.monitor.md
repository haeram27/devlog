# linux system resource 모니터링 도구

btop과 glances 혼합 사용 권장

- htop : 시스템 리소스 모니터링 기본형, 각 process에 signal 전송 또는 renice 기능 제공
- iotop : DISK I/O 전용 모니터링
- glances : htop 대비 DISK I/O 정보 추가 모니터링 가능
- btop : 사용자 친화적 UI 제공, htop의 상위 호환(htop의 기능 대부분 지원 가능), 마우스 지원