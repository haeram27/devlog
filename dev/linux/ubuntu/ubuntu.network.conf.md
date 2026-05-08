# ubuntu network 설정

ubuntu는 기본적으로 NIC 설정 및 사용에 netplan을 사용함 (rocky의 경우 NetworkManager)
단, 별도로 NetworkManager를 설치하면 netplan이 NetworkManger를 참조 하는 형식으로 사용이 가능함


## NetworkManger 설치 시

NetworkManager를 설치하면 다음과 같이 

```bash
$ cat /etc/netplan/01-network-manager-all.yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

```bash
$ cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
      - 192.168.57.50/24
      match:
        macaddress: 00:00:00:00:00:00
      nameservers:
        addresses:
          - 1.1.1.1
          - 2.2.2.2
          - 8.8.8.8
        search: []
      routes:
      - to: default
        via: 192.168.57.1
      set-name: ens33
    ens34:
      accept-ra: true
      dhcp4: true
  version: 2
```