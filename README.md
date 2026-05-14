# empty-AI-SW-basic
# 📋 시스템 관제 자동화 프로젝트 수행 내역서

본 문서는 리눅스 서버 보안 설정, 권한 관리, 그리고 시스템 관제 자동화 스크립트 구현에 대한 수행 내역을 기록한 문서입니다.

---

## 1. 설정 및 명령어 기록 (Implementation Log)

### 🔐 1.1 SSH 및 방화벽 설정
*   **SSH 포트 변경 및 Root 접속 차단**
    ```bash
    # /etc/ssh/sshd_config 수정 내역
    Port 20022
    PermitRootLogin no
    
    # 서비스 재시작
    sudo systemctl restart ssh
    ```
*   **방화벽(UFW) 설정**
    ```bash
    sudo ufw allow 20022/tcp
    sudo ufw allow 15034/tcp
    sudo ufw enable
    ```

### 👥 1.2 계정 및 권한 체계 구축
*   **계정 및 그룹 생성**
    ```bash
    # 그룹 생성
    sudo groupadd agent-common
    sudo groupadd agent-core

    # 계정 생성 및 그룹 할당
    sudo useradd -m -G agent-core,agent-common agent-admin
    sudo useradd -m -G agent-core,agent-common agent-dev
    sudo useradd -m -G agent-common agent-test
    ```
*   **디렉토리 권한 및 ACL 설정**
    ```bash
    # AGENT_HOME 설정 및 권한 부여
    sudo chown :agent-common $AGENT_HOME/upload_files
    sudo chmod 770 $AGENT_HOME/upload_files
    
    # 보안 디렉토리 및 로그 디렉토리 (agent-core 전용)
    sudo chown :agent-core $AGENT_HOME/api_keys /var/log/agent-app
    sudo chmod 770 $AGENT_HOME/api_keys /var/log/agent-app
    ```

### ⚙️ 1.3 환경 변수 및 자동화 등록
*   **환경 변수 설정 (`.bashrc` 또는 `/etc/environment`)**
    ```bash
    export AGENT_HOME="/home/agent-admin/agent-app"
    export AGENT_PORT=15034
    ```
*   **Crontab 등록 (agent-admin 계정)**
    ```bash
    # crontab -e
    * * * * * /home/agent-admin/agent-app/bin/monitor.sh
    ```

---

## 2. 필수 증거 자료 체크리스트 (Evidence Checklist)


### SSH 포트(20022) 및 Root 차단   
`ss -tulnp \| grep 20022`
```bash
# 포트 20022로 변경 및 Root 접속 차단
sudo sed -i 's/^#\?Port .*/Port 20022/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config

# 설정 적용 (ssh 서비스 재시작)
sudo systemctl restart ssh

# 설정 적용 확인
ss -tuln | grep 20022
```

<img width="566" height="108" alt="스크린샷 2026-05-14 오후 10 22 22" src="https://github.com/user-attachments/assets/1de527ba-714d-4331-8177-602ef910c073" />
  
### 방화벽 허용 (20022, 15034)  
`sudo ufw status verbose`   
```bash
# 방화벽 도구 UFW 설치
sudo apt update
sudo apt install ufw -y

# 포트 허용
sudo ufw allow 20022/tcp
sudo ufw allow 15034/tcp

# 방화벽 활성화
sudo ufw --force enable

# 상태 확인
sudo ufw status

```

<img width="478" height="307" alt="스크린샷 2026-05-14 오후 10 28 35" src="https://github.com/user-attachments/assets/90e781b6-f1d6-4782-a8b6-27ede1256deb" />
  
### 계정/그룹 생성 확인   
`id agent-admin`, `id agent-dev`  
```bash
# 1. 그룹 생성
sudo groupadd agent-common
sudo groupadd agent-core

# 2. 사용자 생성 (홈 디렉토리 생성 및 보조 그룹 할당)
sudo useradd -m -G agent-core,agent-common agent-admin
sudo useradd -m -G agent-core,agent-common agent-dev
sudo useradd -m -G agent-common agent-test

# 3. 생성 확인 (정상이라면 각 계정 정보가 한 줄씩 출력됨)
id agent-admin && id agent-dev && id agent-test

```
<img width="642" height="254" alt="스크린샷 2026-05-14 오후 10 32 20" src="https://github.com/user-attachments/assets/0b053376-c2e3-44d4-b3bd-5bab5d074bf4" />

  
### 디렉토리 구조 및 ACL 권한  
`ls -ld`, `getfacl [디렉토리명]`  
```bash
# 1. 디렉토리 생성
sudo mkdir -p /home/agent-admin/agent-app/upload_files
sudo mkdir -p /home/agent-admin/agent-app/api_keys
sudo mkdir -p /var/log/agent-app

# 2. 소유권 설정 (admin 소유, common 그룹 관리)
sudo chown -R agent-admin:agent-common /home/agent-admin/agent-app

# 3. 상세 권한 설정 (핵심!)
# api_keys와 log는 core 그룹(admin, dev)만 볼 수 있게 설정
sudo chown :agent-core /home/agent-admin/agent-app/api_keys /var/log/agent-app
sudo chmod 770 /home/agent-admin/agent-app/upload_files
sudo chmod 770 /home/agent-admin/agent-app/api_keys
sudo chmod 770 /var/log/agent-app

# 4. 검증
ls -ld /home/agent-admin/agent-app/upload_files
ls -ld /home/agent-admin/agent-app/api_keys
ls -ld /var/log/agent-app
```
<img width="696" height="268" alt="스크린샷 2026-05-14 오후 10 35 27" src="https://github.com/user-attachments/assets/6e58b44e-1c4a-4ca9-9c38-4181000ef327" />

### 앱 Boot Sequence 5단계 [OK]  
`Agent READY` 출력 확인  
    
### `monitor.sh` 실행 결과 프로세스/리소스 정상 수집 확인   
  
### `monitor.log` 누적 기록   
`tail -f /var/log/agent-app/monitor.log`  
  
  
### crontab 자동 실행 확인  
1분 주기 로그 증가 확인  
  

---

## 3. 자동화 스크립트 소스코드 (Source Code)

### 📄 monitor.sh
```bash
#!/bin/bash

# [프로젝트 경로 설정]
AGENT_HOME="/home/agent-admin/agent-app"
LOG_FILE="/var/log/agent-app/monitor.log"

# 1. Health Check (Process & Port)
# ... (작성하신 스크립트 내용을 여기에 붙여넣으세요) ...

# 2. Resource Collection (CPU, MEM, DISK)
# ...

# 3. Logging
# [$(date '+%Y-%m-%d %H:%M:%S')] PID:... CPU:..% MEM:..% DISK_USED:..% >> $LOG_FILE
```

## 5. 학습 고찰 및 핵심 개념 정리 (Q&A)

본 프로젝트를 수행하며 학습한 핵심 보안 및 운영 개념을 정리합니다.

### Q1. SSH 포트 변경과 Root 원격 접속 차단이 왜 기본 보안의 핵심인가요?
*   **답변**: 전 세계의 해커들은 봇(Bot)을 이용해 전형적인 SSH 포트인 **22번**을 대상으로 무차별 대입 공격(Brute-force)을 시도합니다. 포트를 **20022**와 같은 비표준 포트로 변경하는 것만으로도 자동화된 공격의 90% 이상을 회피할 수 있습니다(Security by Obscurity). 
*   또한, **Root 계정**은 시스템의 전권을 가진 계정이므로, 원격 접속을 허용할 경우 단 한 번의 비밀번호 유출로 시스템 전체가 장악될 위험이 있습니다. 따라서 일반 계정으로 접속 후 필요한 경우에만 `sudo`를 사용하는 것이 표준 보안 절차입니다.

### Q2. "필요 포트만 허용"하는 방화벽 정책의 중요성은 무엇인가요?
*   **답변**: 서버에 열려 있는 모든 포트는 잠재적인 공격 통로(Attack Surface)입니다. UFW를 통해 서비스에 꼭 필요한 **20022(SSH)**와 **15034(APP)**만 허용하고 나머지를 차단(Default Deny)함으로써, 알 수 없는 서비스의 취약점을 통한 침입을 원천 봉쇄할 수 있습니다.

### Q3. 계정/그룹과 ACL을 통해 디렉토리를 분리하는 이유는 무엇인가요?
*   **답변**: **최소 권한 원칙(Principle of Least Privilege)**을 실현하기 위함입니다. `agent-test` 계정은 테스트 파일에만 접근하고, 보안이 중요한 `api_keys`나 로그 파일에는 접근할 수 없도록 격리함으로써 내부 사용자에 의한 실수나 악의적인 데이터 유출 사고를 방지할 수 있습니다.

### Q4. 환경 변수(AGENT_HOME 등)로 실행 환경을 고정하는 이유는 무엇인가요?
*   **답변**: 서비스가 설치된 경로가 사용자마다 다르거나 실행 위치에 따라 상대 경로가 바뀌면 프로그램이 오작동할 수 있습니다. 환경 변수를 사용하면 시스템 내 어디서든 **동일한 환경(Context)**에서 앱이 실행되도록 보장할 수 있으며, 관리 및 이식성이 높아집니다. `echo $AGENT_HOME` 명령어로 즉시 설정값을 검증할 수 있습니다.

### Q5. 쉘 스크립트로 시스템 상태를 수집하고 로그를 남기는 흐름의 장점은?
*   **답변**: 장애가 발생했을 때 과거의 상태를 알 수 없다면 원인 분석은 불가능해집니다. `monitor.sh`를 통해 주기적으로 CPU, 메모리, 포트 상태를 기록해 두면, 장애 발생 시점의 로그를 분석하여 **"언제, 어떤 자원이 부족해서 문제가 생겼는지"** 객관적으로 추적하고 재발 방지 대책을 세울 수 있습니다.

### Q6. Crontab 주기 실행과 로그 보존 정책(압축/삭제)이 왜 필요한가요?
*   **답변**: 모니터링은 끊김 없이 지속되어야 하므로 `crontab`을 통한 자동화가 필수입니다. 하지만 무한정 생성되는 로그는 결국 **디스크 풀(Disk Full)** 장애를 유발하여 서버를 멈추게 만듭니다. 따라서 일정 기간이 지난 로그는 압축하여 용량을 줄이고, 오래된 데이터는 삭제하는 보존 정책이 서버의 가용성을 유지하는 데 핵심적인 역할을 합니다.

