# Linux 시스템 관제 자동화

## 1. 프로젝트 개요

Ubuntu Linux 환경에서 서버 운영에 필요한 기본 보안 설정, 사용자/그룹 권한 관리, 애플리케이션 실행 환경 구성, 시스템 관제 자동화 스크립트를 구현하였다.

주요 수행 내용은 다음과 같다.

* SSH 포트 20022 변경
* Root 원격 접속 차단
* UFW 방화벽 설정
* 계정 및 그룹 생성
* 디렉토리 권한 및 ACL 확인
* 제공 애플리케이션 실행
* Bash 기반 monitor.sh 작성
* cron을 통한 매분 자동 실행
* monitor.log 누적 기록 확인

---

## 2. 개발 환경

* OS: Ubuntu 22.04 기반 WSL
* Shell: Bash
* Firewall: UFW
* App Port: 15034
* SSH Port: 20022
* Scheduler: cron

---

## 3. 사용자 및 그룹 구성

### 사용자

* agent-admin
* agent-dev
* agent-test

### 그룹

* agent-common
* agent-core

### 그룹 정책

* agent-common: agent-admin, agent-dev, agent-test
* agent-core: agent-admin, agent-dev

공용 파일 영역과 핵심 보안 영역을 분리하기 위해 그룹 기반 권한 정책을 적용하였다.

---

## 4. 디렉토리 및 권한 구성

### 디렉토리 구조

```bash
/home/agent-admin/agent-app
/home/agent-admin/agent-app/upload_files
/home/agent-admin/agent-app/api_keys
/var/log/agent-app
```

### 권한 정책

* upload_files: agent-common 그룹 접근 가능
* api_keys: agent-core 그룹만 접근 가능
* /var/log/agent-app: agent-core 그룹만 접근 가능

최소 권한 원칙을 적용하여 공용 영역과 보안 영역을 분리하였다.

---

## 5. SSH 보안 설정

SSH 기본 포트인 22번 대신 20022번 포트를 사용하도록 설정하였다.

```bash
Port 20022
PermitRootLogin no
```

Root 원격 접속을 차단하여 관리자 계정 직접 접근 위험을 줄였다.

---

## 6. 방화벽 설정

UFW를 활성화하고 필요한 포트만 허용하였다.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 20022/tcp
sudo ufw allow 15034/tcp
sudo ufw enable
```

허용 포트:

* 20022/tcp: SSH
* 15034/tcp: Application

---

## 7. 애플리케이션 실행 환경

환경 변수는 다음과 같이 설정하였다.

```bash
export AGENT_HOME=/home/agent-admin/agent-app
export AGENT_PORT=15034
export AGENT_UPLOAD_DIR=/home/agent-admin/agent-app/upload_files
export AGENT_KEY_PATH=/home/agent-admin/agent-app/api_keys/t_secret.key
export AGENT_LOG_DIR=/var/log/agent-app
```

키 파일:

```bash
/home/agent-admin/agent-app/api_keys/t_secret.key
```

내용:

```text
agent_api_key_test
```

앱 실행 후 Boot Sequence 5단계 통과 및 Agent READY 상태를 확인하였다.

---

## 8. monitor.sh 구현

monitor.sh는 Bash로 작성하였다.

### 주요 기능

* agent-app 프로세스 확인
* TCP 15034 포트 LISTEN 확인
* CPU 사용률 수집
* 메모리 사용률 수집
* Root 디스크 사용률 수집
* monitor.log에 상태 기록

### 로그 파일 경로

```bash
/var/log/agent-app/monitor.log
```

### 로그 형식

```text
[YYYY-MM-DD HH:MM:SS] PID:... CPU:..% MEM:..% DISK_USED:..%
```

---

## 9. cron 자동 실행

monitor.sh가 매분 자동 실행되도록 crontab에 등록하였다.

```bash
* * * * * /home/agent-admin/agent-app/bin/monitor.sh
```

등록 후 monitor.log에 로그가 자동 누적되는 것을 확인하였다.

---

## 10. 증빙 자료

### SSH 포트 변경 및 Root 접속 차단

![SSH](evidence/01_ssh_20022.png)

### UFW 방화벽 설정

![UFW](evidence/02_ufw_firewall.png)

### 계정 및 그룹 생성

![Accounts](evidence/03_accounts_groups.png)

### 디렉토리 및 ACL 권한

![ACL](evidence/04_acl_permissions.png)

### 앱 Boot Sequence 및 Agent READY

![Agent Ready](evidence/05_agent_ready.png)

### 앱 포트 15034 LISTEN

![App Port](evidence/06_app_port.png)

### monitor.sh 실행 결과

![Monitor](evidence/07_monitor_result.png)

### monitor.log 누적 기록

![Monitor Log](evidence/08_monitor_log.png)

### cron 등록 확인

![Cron](evidence/09_cron_registration.png)

---

## 11. 제출 파일

* README.md
* monitor.sh
* evidence 디렉토리 내 증빙 이미지
