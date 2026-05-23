# Homelab 학습 노트 - Proxmox / Ubuntu 서버 구축 (2026-05-23)

## 오늘 목표

- 안쓰는 서버용 PC에 Proxmox 설치
- 메인 PC에서 원격 접속 가능한 환경 만들기
- Ubuntu VM 생성
- 앞으로 Docker 및 개인 서비스 운영 가능한 기반 구축

---

# 1. 네트워크 문제 해결

## 상황

현재 서버 컴퓨터는 랜선을 직접 연결할 수 없는 위치에 있었음.

초기에는 USB 무선 랜카드(Wi-Fi 안테나)를 통해 인터넷 연결을 시도했음.

하지만 Proxmox는 일반적인 Linux 서버 환경 기반이라:
- 일부 USB Wi-Fi 장치 드라이버 인식 문제
- 브릿지 네트워크 구성 어려움
- 안정성 문제

등으로 인해 정상적인 네트워크 연결이 어려웠음.

---

## 해결 방법

### 중계기(브릿지 모드 가능한 공유기) 구매

목표:
- 거실 공유기의 Wi-Fi를 수신
- 서버 PC에는 유선 LAN 환경 제공

즉:

```text
집 공유기 (Wi-Fi)
   ↓
중계기 / 브릿지 공유기
   ↓ LAN
서버 PC (Proxmox)
```

구조로 변경.

---

# 2. Proxmox UI 접속 실패

## 문제 상황

Proxmox 설치 후:
- 메인 PC 브라우저에서 접속 불가능

초기 원인:
- 설치 당시 설정된 고정 IP와
- 현재 네트워크 환경이 달라짐

즉:
- 이전 네트워크 기준 IP가 유지되고 있었음.

---

## 확인에 사용한 명령어

### 현재 네트워크 상태 확인

```bash
ip a
```

확인 가능 항목:
- 네트워크 인터페이스
- 현재 IP 주소
- 연결 상태

---

# 3. Proxmox 네트워크 수정

## 해결 과정

Proxmox 콘솔에서 네트워크 설정 수정.

### 수정 파일

```bash
nano /etc/network/interfaces
```

---

## 주요 수정 내용

기존:
- 잘못된 고정 IP 설정

수정:
- 현재 공유기 대역에 맞는 IP로 변경

예시:

```text
192.168.x.x
```

형태로 수정.

---

## 적용 명령어

```bash
systemctl restart networking
```

또는

```bash
reboot
```

---

# 4. 메인 PC에서 Proxmox UI 접속 성공

## 접속 주소

```text
https://서버IP:8006
```

예시:

```text
https://192.168.x.x:8006
```

---

## 결과

메인 PC 브라우저에서:
- Proxmox Web UI 정상 접속 확인
- 서버 원격 관리 가능 상태 확인

---

# 5. Ubuntu VM 생성

## 생성 목적

- Linux 학습
- Docker
- nginx
- Next.js 서버 운영
- 네트워크 공부
- SSH 연습

등을 위한 테스트 서버 구축.

---

# VM 생성 시 설정

## CPU

서버 사양:

```text
Intel i3-2100
2 Core / 4 Thread
```

설정:

```text
Socket: 1
Core: 2
CPU Type: host
```

---

## RAM

```text
4096MB (4GB)
```

설정.

---

## 디스크

초기:
- 32GB 할당

이후 확인 결과:
- Proxmox 전체 디스크는 정상 사용중
- 단순히 VM 디스크만 32GB였음

학습용으로 우선 사용 결정.

향후:
- 운영용 Ubuntu 서버는 64GB 이상 예정.

---

# Ubuntu 설치 중 발생한 문제

## 문제

설치 중:
- `/` 루트 마운트를 실수로 해제

오류:

```text
Mount a filesystem at /
```

발생.

---

## 해결

LVM 논리 볼륨:

```text
ubuntu-lv
```

를 다시:

```text
/
```

로 마운트 설정.

최종 구조:

```text
/       28GB
/boot    2GB
```

정상 확인.

---

# 디스크 구조 확인

## 사용 명령어

```bash
lsblk
```

---

## 확인 내용

```text
sda 465GB
 └ pve-data-tpool 337GB
```

즉:
- SSD 전체는 정상적으로 Proxmox에서 사용중
- 이전 Windows 데이터는 사실상 제거된 상태

확인.

---

# 오늘 이해한 개념들

## Proxmox

가상머신(VM)을 관리하는 하이퍼바이저 플랫폼.

한 대의 물리 서버에서:
- Ubuntu
- Debian
- Windows
- Docker 서버

등 여러 서버를 동시에 운영 가능.

---

## VM (가상머신)

실제 컴퓨터 안에 존재하는 또 다른 컴퓨터 개념.

ISO 파일만 있으면:
- 여러 서버 생성 가능
- 역할 분리 가능
- 테스트 환경 구축 가능

---

## QEMU Agent

Proxmox ↔ Ubuntu VM 간 통신 기능.

활성화 시:
- IP 표시
- 정상 종료
- 상태 확인
- 백업 안정성

등 기능 사용 가능.

Ubuntu 내부 설치 명령어:

```bash
sudo apt update
sudo apt install qemu-guest-agent -y
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
```

---

# 앞으로 할 예정

## 학습용 서버

```text
Ubuntu-Test
32GB
```

목적:
- Linux 연습
- Docker 실험
- 네트워크 실습
- 자유로운 테스트

---

## 운영 연습용 서버

```text
Ubuntu-Prod
64GB+
```

목적:
- Docker
- nginx
- Next.js
- 개인 홈페이지
- API 서버
- Reverse Proxy
- Cloudflare
- SSH

---

# 느낀점

처음에는:
- 네트워크
- 고정 IP
- LVM
- VM
- 디스크 구조

등이 매우 복잡하게 느껴졌지만,

실제로:
- 직접 설치
- 문제 해결
- 콘솔 조작
- 구조 확인

을 하면서 Linux 서버와 인프라 구조가 조금씩 이해되기 시작함.

특히:
- "내 PC에서 프로그램 실행"
이 아니라

- "서비스를 운영하는 서버를 구축"

한다는 감각을 배우기 시작함.

앞으로:
- Docker
- nginx
- AWS
- DevOps
- 네트워크

공부와 자연스럽게 연결될 것으로 기대됨.