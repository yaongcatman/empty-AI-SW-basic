# empty-AI-SW-basic
# 📋 시스템 관제 자동화 프로젝트 수행 내역서

본 문서는 리눅스 서버 보안 설정, 권한 관리, 그리고 시스템 관제 자동화 스크립트 구현에 대한 수행 내역을 기록한 문서입니다.
서버 관제 프로젝트는 '안전한 집을 짓고 CCTV를 설치하는 과정. 핵심 개념인 하드닝(Hardening)은 서버의 방어력을 높여 공격받기 어렵게 단단히 만드는 모든 작업.
---

## 1. 설정 및 명령어 기록 (Implementation Log)

### 🔐 1.1 SSH 및 방화벽 설정
*   **SSH 포트 변경 및 Root 접속 차단**
*   도둑이 들어오지 못하게 집을 보강하고 담장을 쌓는 단계
    ```bash
    # /etc/ssh/sshd_config 수정 내역
    Port 20022
    PermitRootLogin no
    
    # 서비스 재시작
    sudo systemctl restart ssh
    ```
    * 뒷문 만들기 (SSH 포트 변경): 도둑들이 주로 노리는 정문(22번) 대신, 나만 아는 뒷문(20022번)을 새로 만듭니다.

    * 집주인 보호 (Root 접속 차단): 모든 권한을 가진 집주인 계정으로 직접 들어오는 것을 막아, 비밀번호 하나로 집 전체가 털리는         것을 방지합니다.
    
*   **방화벽(UFW) 설정**
    ```bash
    sudo ufw allow 20022/tcp
    sudo ufw allow 15034/tcp
    sudo ufw enable
    ```
    * 마당 검문소 (방화벽 UFW): 집 주변에 담장을 치고, 허락된 포트(20022, 15034)만 통과시키는 검문소를 세웁니다.
  

### 👥 1.2 계정 및 권한 체계 구축
*   **계정 및 그룹 생성**
*   가족마다 들어갈 수 있는 방을 나누어 피해를 최소화하는 단계 
    ```bash
    # 그룹 생성
    sudo groupadd agent-common
    sudo groupadd agent-core

    # 계정 생성 및 그룹 할당
    sudo useradd -m -G agent-core,agent-common agent-admin
    sudo useradd -m -G agent-core,agent-common agent-dev
    sudo useradd -m -G agent-common agent-test
     ```
    * 계정 나누기: 관리자, 개발자, 테스터에게 각자 다른 아이디를 줍니다.  
    
    *   **디렉토리 권한 및 ACL 설정**
    ```bash
    # AGENT_HOME 설정 및 권한 부여
    sudo chown :agent-common $AGENT_HOME/upload_files
    sudo chmod 770 $AGENT_HOME/upload_files
    
    # 보안 디렉토리 및 로그 디렉토리 (agent-core 전용)
    sudo chown :agent-core $AGENT_HOME/api_keys /var/log/agent-app
    sudo chmod 770 $AGENT_HOME/api_keys /var/log/agent-app
    ```
    *방마다 자물쇠 채우기: 테스터는 테스트 방만, 관리자는 금고 방까지 들어갈 수 있게 설정합니다. 이렇게 하면 누군가 실수해도 다른      방은 안전합니다.
    
### ⚙️ 1.3 환경 변수 및 자동화 등록
*   **환경 변수 설정 (`.bashrc` 또는 `/etc/environment`)**
*   앱을 실행하기 전, 주변이 완벽한지 검사하는 부트 시퀀스(Boot Sequence) 과정
       ```bash
    export AGENT_HOME="/home/agent-admin/agent-app"
    export AGENT_PORT=15034
    ```
    * 이정표 세우기 (환경 변수): 앱이 길을 잃지 않게 "우리 집 주소는 여기야"라고 미리 장소를 알려줍니다.
    * 5단계 합격: 계정 권한이나 필수 파일이 있는지 하나씩 체크해서 모두 [OK]가 떠야 "준비 완료(Ready)" 상태가 됩니
*   **Crontab 등록 (agent-admin 계정)**
    ```bash
    # crontab -e
    * * * * * /home/agent-admin/agent-app/bin/monitor.sh
    ```
* 로봇이 멈추지 않고 일하게 하고, 관찰한 기록을 관리하는 마지막 단계.
* 알람 시계 (Crontab): 리눅스의 알람 시계를 맞춰 1분마다 로봇(monitor.sh)을 깨워 감시하게 시킵니다.
* 시스템 일기장 (로그): 로봇이 본 내용을 일기(monitor.log)로 남깁니다. 나중에 문제가 생기면 이 일기를 보고 원인을 찾습니다.
* 일기장 정리: 일기가 너무 많아져 창고를 차지하지 않게, 오래된 일기는 압축하거나 버리는 규칙을 세웁니다.
---

## 2. 필수 증거 자료 체크리스트 (Evidence Checklist)


### ⁂SSH 포트(20022) 및 Root 차단   
코디
```bash
# 포트 20022로 변경 및 Root 접속 차단
sudo sed -i 's/^#\?Port .*/Port 20022/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config

# 설정 적용 (ssh 서비스 재시작)
sudo systemctl restart ssh

# 설정 적용 확인
ss -tuln | grep 20022
```
### 🔍 SSH 보안 설정 명령어 상세 가이드

서버의 대문을 튼튼하게 잠그고, 나만 아는 비밀 통로를 만드는 핵심 명령어들에 대한 분석입니다.

---

#### 1️⃣ 포트 번호 변경 (Port 20022)
> **명령어:** `sudo sed -i 's/^#\?Port .*/Port 20022/' /etc/ssh/sshd_config`

* **명령어 분석**
    * **`sudo`** (**S**ubstitute **U**ser **DO**): 관리자 권한으로 실행합니다.
    * **`sed`** (**S**tream **Ed**itor): 텍스트를 자동으로 편집하는 도구입니다.
    * **`-i`** (**i**n-place): 파일을 열지 않고 그 자리에서 즉시 수정합니다.
* **사용 이유 (Why)**: 모든 해커가 노리는 기본 통로(22번) 대신, 비표준 포트(20022번)를 사용하여 자동화된 공격을 90% 이상 피하기 위해 사용합니다.
> **Q: 왜 하필 20022번인가요? 원래 있는 문인가요?**
> **A:** 아닙니다. 20022번은 사용자가 임의로 지정한 **비표준 포트**입니다. 
> 리눅스의 SSH 설정 파일에서 포트 번호를 변경하면, 시스템은 기본 22번 통로를 닫고 
> 우리가 지정한 20022번 번호를 '새로운 통로'로 인식하여 손님(데이터)을 맞이하게 됩니다.

### 🧩 명령어 속 '외계어' 해부하기 (정규 표현식 분석)

`sed` 명령어에서 사용된 `'s/^#\?PermitRootLogin .*/PermitRootLogin no/'` 부분의 핵심 규칙을 정리했습니다.

#### 1. `s/` : 찾아서 바꾸기 (Substitute)
* **의미**: "지금부터 **S**ubstitute(교체) 작업을 시작할게!"라는 신호입니다.
* **구조**: `s/찾을내용/바꿀내용/` 형식으로 사용됩니다.

#### 2. `^#\?` : 주석 처리된 줄까지 모두 찾기
* **`^` (Caret)**: "줄의 맨 처음부터 시작하는 것만 찾아라"라는 뜻입니다.
* **`#`**: 리눅스 설정 파일에서 `#`은 주석(설명 처리되어 작동 안 함)을 의미합니다.
* **`\?`**: 바로 앞의 글자(`#`)가 **"있을 수도 있고, 없을 수도 있다"**는 뜻입니다.
    * **이유**: 설정이 `#PermitRootLogin`으로 되어 있든, 그냥 `PermitRootLogin`으로 되어 있든 상관없이 다 찾아내기 위해서입니다.

#### 3. `.*` : 그 뒤에 뭐가 있든 끝까지 선택
* **`.` (Dot)**: "아무 글자나 한 글자"를 의미합니다.
* **`*` (Asterisk)**: "그게 몇 개가 있든 상관없다(0개 이상)"는 뜻입니다.
* **이유**: `PermitRootLogin` 뒤에 기존에 `yes`가 써 있든, `prohibit-password`가 써 있든 상관없이 **그 줄 전체를 싹 다 잡아서 지우기 위해** 사용합니다.

#### 4. `/PermitRootLogin no/` : 새로운 값으로 덮어쓰기
* **의미**: 위에서 `^#\?PermitRootLogin .*` 패턴으로 찾은 **지저분한 줄 전체를 지우고**, 그 자리에 아주 깔끔하게 `PermitRootLogin no`라는 글자를 딱 써 넣으라는 명령입니다.

---

### 💡 한 줄 요약
> **"주석(`#`)이 달려 있든 아니든, `PermitRootLogin`으로 시작하는 줄을 통째로 찾아서, 깨끗하게 `PermitRootLogin no`로 바꿔버려!"** 라는 뜻입니다.

---

#### 2️⃣ Root 계정 접속 차단 (Security Hardening)
> **명령어:** `sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config`

* **주요 용어**
    * **`Permit`**: 허용하다 / **`Root`**: 리눅스 최고 관리자(집주인) / **`no`**: 아니요
* **사용 이유 (Why)**: Root 계정은 집의 모든 열쇠를 가진 것과 같습니다. 이 계정이 털리면 서버 전체가 장악되므로, 외부에서 Root로 바로 접속하는 길을 아예 없애 보안을 강화합니다.

---

#### 3️⃣ 설정 사항 즉시 적용 (Service Restart)
> **명령어:** `sudo systemctl restart ssh`

* **명령어 분석**
    * **`systemctl`** (**system** **c**on**t**rol): 시스템 서비스를 제어하는 본부입니다.
    * **`restart`**: 서비스를 껐다가 다시 켭니다.
* **사용 이유 (Why)**: 설정 파일을 고쳤다고 시스템이 바로 알 수는 없습니다. 서비스(SSH)에게 "바뀐 장부를 읽고 다시 근무를 시작해!"라고 명령하는 과정입니다.

---

#### 4️⃣ 설정 결과 최종 확인 (Verification)
> **명령어:** `ss -tuln | grep 20022`

* **명령어 분석**
    * **`ss`** (**s**ocket **s**tatistics): 네트워크 연결 상태를 보여주는 도구입니다.
    * **`-tuln`**: TCP/UDP 연결을 Listening(대기) 상태로 숫자로 표시합니다.
    * **`grep`**: 수많은 정보 중에서 내가 원하는 글자만 찾아내는 돋보기입니다.
* **사용 이유 (Why)**: 내가 만든 비밀 통로(20022번)가 정상적으로 열려 있는지 직접 확인(검증)하기 위해 사용합니다.

### 📊 SSH 보안 설정 명령어 요약표

| 명령어 | 핵심 약자 | 쉬운 비유 | 사용하는 이유 (Why) |
| :--- | :--- | :--- | :--- |
| **`sed`** | **S**tream **Ed**itor | 자동 글자 교체 마법 | 파일을 직접 열지 않고 빠르게 수정하려고 |
| **`systemctl`** | **system** **c**on**trol** | 경비원 깨우기 | 바뀐 규칙을 시스템에 즉시 반영하려고 |
| **`ss`** | **s**ocket **s**tatistics | 문 단속 확인하기 | 통로(포트)가 잘 열렸는지 감시하려고 |
| **`grep`** | **G.R.E.P** (찾기) | 데이터 돋보기 | 필요한 정보만 쏙 골라내서 보려고 |

---

### 📝 명령어 상세 분석 및 실행 목적

| 실행 코드 (Command) | 기능 및 역할 (Function) | 상세 설명 (Description) |
| :--- | :--- | :--- |
| `sudo sed -i 's/.../Port 20022/'` | **포트 번호 변경** | 22번(정문) 대신 20022번(비밀문)으로 접속 통로를 변경함 |
| `sudo sed -i 's/.../PermitRootLogin no/'` | **관리자 접속 차단** | 최고 관리자인 Root가 외부에서 바로 들어오지 못하게 잠금 |
| `sudo systemctl restart ssh` | **서비스 재시작** | 수정된 보안 설정 장부를 경비원(SSH 서비스)에게 새로 적용함 |
| `ss -tuln \| grep 20022` | **설정 결과 검증** | 20022번 포트가 서버에서 정상적으로 작동 중인지 확인함 |


<img width="566" height="108" alt="스크린샷 2026-05-14 오후 10 22 22" src="https://github.com/user-attachments/assets/1de527ba-714d-4331-8177-602ef910c073" />

##

### ⁂방화벽 허용 (20022, 15034)  
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

##

### ⁂계정/그룹 생성 확인   
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

##

### ⁂디렉토리 구조 및 ACL 권한  
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
sudo ls -ld /home/agent-admin/agent-app/upload_files
sudo ls -ld /home/agent-admin/agent-app/api_keys
sudo ls -ld /var/log/agent-app
```

<img width="629" height="158" alt="스크린샷 2026-05-14 오후 10 39 08" src="https://github.com/user-attachments/assets/96d15249-dcad-4ee3-a407-0314df4437fc" />

<img width="665" height="97" alt="스크린샷 2026-05-14 오후 10 39 21" src="https://github.com/user-attachments/assets/edf7f5e3-1770-4787-92b5-8928a13e1cc9" />

##

### ⁂앱 Boot Sequence 5단계 [OK]  
`Agent READY` 출력 확인  

```bash
# 1단계
# 소유권 설정
sudo chown agent-admin:agent-common /home/agent-admin/agent-app/agent-app
  
# 실행 권한 부여 (확장자가 없으므로 실행 가능하게 만들어야 함)
sudo chmod +x /home/agent-admin/agent-app/agent-app

#검증
# 1 단계 -u 옵션을 사용하여 agent-admin 계정으로 앱을 실행합니다.  
sudo -u agent-admin /home/agent-admin/agent-app/agent-app | sudo tee /var/log/agent-app/boot.log
```
<img width="772" height="29" alt="스크린샷 2026-05-14 오후 11 21 01" src="https://github.com/user-attachments/assets/176efd82-9d12-4ccb-a6b0-7455053c8e54" />
<img width="811" height="217" alt="스크린샷 2026-05-14 오후 11 21 54" src="https://github.com/user-attachments/assets/ec873522-06ec-4071-9d89-9316ceeca602" />


```bash
# 2 단계 / 3단계
[3단계]
#앱이 원하는 업로드 폴더 생성
sudo mkdir -p /home/agent-admin/agent-app/upload_files

#앱이 원하는 키 폴더 및 파일 생성
sudo mkdir -p /home/agent-admin/agent-app/api_keys
sudo touch /home/agent-admin/agent-app/api_keys/t_secret.key

#소유권 전체 재설정
sudo chown -R agent-admin:agent-common /home/agent-admin/agent-app/

#검증
[2단계]
sudo -u agent-admin \
AGENT_HOME=/home/agent-admin/agent-app \
AGENT_PORT=15034 \
AGENT_UPLOAD_DIR=/home/agent-admin/agent-app/upload_files \
AGENT_KEY_PATH=/home/agent-admin/agent-app/api_keys/t_secret.key \
/home/agent-admin/agent-app/agent-app | sudo tee /var/log/agent-app/boot.log
```
<img width="745" height="383" alt="스크린샷 2026-05-14 오후 11 30 32" src="https://github.com/user-attachments/assets/28b2ff30-48ba-4aed-bc67-51b1f1f5f18b" />

```bash
# 4단계 / 5단계
1. 15034 포트를 쓰고 있는 범인 찾기
sudo ss -tulpn | grep 15034

1. 범인 소탕 (프로세스 종료)
sudo kill -9 2396

#검증
sudo -u agent-admin \
AGENT_HOME=/home/agent-admin/agent-app \
AGENT_PORT=15034 \
AGENT_UPLOAD_DIR=/home/agent-admin/agent-app/upload_files \
AGENT_KEY_PATH=/home/agent-admin/agent-app/api_keys/t_secret.key \
/home/agent-admin/agent-app/agent-app | sudo tee /var/log/agent-app/boot.log

```
<img width="754" height="453" alt="스크린샷 2026-05-14 오후 11 35 24" src="https://github.com/user-attachments/assets/4fb5dbe1-3d17-4d70-b438-7ea74b47a35e" />

```bash
마지막검증
sudo cat /var/log/agent-app/boot.log
```
<img width="623" height="439" alt="스크린샷 2026-05-14 오후 11 41 56" src="https://github.com/user-attachments/assets/ac286607-6cde-415e-bfe2-26272ce46a10" />

##

### ⁂`monitor.sh` 실행 결과 프로세스/리소스 정상 수집 확인   

```bash
# 실행 권한 부여
sudo chmod 755 /home/agent-admin/agent-app/monitor.sh

# 소유권 설정 (계정과 그룹 확인)
sudo chown agent-admin:agent-common /home/agent-admin/agent-app/monitor.sh

# 3단계: crontab으로 1분 주기 자동화 등록
sudo crontab -u agent-admin -e
- # agent-admin 계정의 예약 작업 편집
* * * * * /home/agent-admin/agent-app/monitor.sh

#터미널에서 (* * * * * /home/agent-admin/agent-app/monitor.sh) 작동검사
sudo crontab -u agent-admin -ㅣ
```

<img width="675" height="69" alt="스크린샷 2026-05-15 오전 12 29 37" src="https://github.com/user-attachments/assets/4e73f9b2-7143-47ff-a67a-b326905bee7d" />
<img width="1061" height="493" alt="스크린샷 2026-05-14 오후 11 58 01" src="https://github.com/user-attachments/assets/3fe0287f-0632-4609-98e1-c5161adee18b" />
<img width="614" height="396" alt="스크린샷 2026-05-15 오전 12 01 05" src="https://github.com/user-attachments/assets/bdaa6043-8ecb-4e4d-b32c-9f9450cdf7ce" />

##

### ⁂`monitor.log` 누적 기록 / crontab 자동 실행 확인  (1분 주기 로그 증가 확인)  
[오류버전 ]
```bash
스크립트 실행  
`sudo -u agent-admin /home/agent-admin/agent-app/monitor.sh`  
로그확인  
`sudo tail -f /var/log/agent-app/monitor.log`
```
<img width="837" height="195" alt="스크린샷 2026-05-15 오전 12 58 43" src="https://github.com/user-attachments/assets/5aa2b7cf-144d-4588-8cf2-a6a40f5a27cb" />

##

[노멀버전] 
```bash
1. 프로세스 완전 종료 (찌꺼기 제거)
sudo pkill -f agent-app

2. 앱 다시 실행 (환경 변수 포함)
sudo -u agent-admin \
AGENT_HOME=/home/agent-admin/agent-app \
AGENT_PORT=15034 \
AGENT_UPLOAD_DIR=/home/agent-admin/agent-app/upload_files \
AGENT_KEY_PATH=/home/agent-admin/agent-app/api_keys/t_secret.key \
nohup /home/agent-admin/agent-app/agent-app > /dev/null 2>&1 &

3. 로그 확인
sudo tail -f /var/log/agent-app/monitor.log
```  
<img width="724" height="316" alt="스크린샷 2026-05-15 오전 1 03 01" src="https://github.com/user-attachments/assets/1cd775c4-0b1d-48d4-84fe-a5dc27f2a17b" />


---

## 3. 자동화 스크립트 소스코드 (Source Code)

    * 사람 대신 밤새 서버를 지켜볼 자동 감시 로봇(스크립트)을 만드는 단계
    - 생존 확인: "앱은 살아있나?", "문은 잘 열려있나?"를 먼저 확인합니다.
    - 정보 수집: CPU(두뇌 사용량), 메모리(작업 공간), 디스크(창고 저장량)가 얼마나 찼는지 기록합니다.
    - 위험 알림: 창고가 꽉 차거나 컴퓨터가 너무 뜨거워지면 로그에 [WARNING]이라고 적어 주인에게 알립니다.
    
### 📄 monitor.sh
```bash

# 1. 파일 생성 및 편집기 열기
sudo nano /home/agent-admin/agent-app/monitor.sh

#!/bin/bash

LOG_FILE="/var/log/agent-app/monitor.log"
APP_NAME="agent-app"
TARGET_PORT=15034
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# 1. 프로세스(PID) 및 포트 상태 확인
PID=$(pgrep -f "$APP_NAME" | head -n 1)
PORT_LISTEN=$(ss -tuln | grep -w "$TARGET_PORT")

# 2. 리소스 및 상태 판별
if [[ -n "$PID" ]]; then
    STATUS="UP"
    CPU=$(ps -p "$PID" -o %cpu --no-headers | xargs)
    MEM=$(ps -p "$PID" -o rss --no-headers | awk '{printf "%.2fMB", $1/1024}')
    
    # [포트 체크] 프로세스는 있는데 포트가 안 열려있으면 경고
    if [[ -z "$PORT_LISTEN" ]]; then
        PORT_STAT="OFFLINE"
        WARN="!! PORT_ERR !!"
    else
        PORT_STAT="ONLINE"
        WARN="NORMAL"
    fi
else
    STATUS="DOWN"
    PORT_STAT="OFFLINE"
    CPU="0.0"
    MEM="0MB"
    WARN="!! CRITICAL !!"
fi

# 3. 로그 기록 (포트와 경고 내역 포함)
echo "[$TIMESTAMP] Status: $STATUS | Port: $PORT_STAT | CPU: ${CPU:-0.0}% | Mem: ${MEM:-0MB} | Msg: $WARN" >> $LOG_FILE
```
<img width="889" height="487" alt="스크린샷 2026-05-15 오전 12 56 22" src="https://github.com/user-attachments/assets/d071713a-4b9c-4b0a-9b2f-5c2c6760db58" />





## 5. 학습 고찰 및 핵심 개념 정리 (Q&A)

### Q1. SSH 포트 변경과 Root 원격 접속 차단이 왜 기본 보안의 핵심인가?
* 전 세계의 해커들은 봇(Bot)을 이용해 전형적인 SSH 포트인 **22번**을 대상으로 무차별 대입 공격(Brute-force)을 시도합니다. 포트를 **20022**와 같은 비표준 포트로 변경하는 것만으로도 자동화된 공격의 90% 이상을 회피할 수 있습니다(Security by Obscurity). 
* **Root 계정**은 시스템의 전권을 가진 계정이므로, 원격 접속을 허용할 경우 단 한 번의 비밀번호 유출로 시스템 전체가 장악될 위험이 있습니다. 따라서 일반 계정으로 접속 후 필요한 경우에만 `sudo`를 사용하는 것이 표준 보안 절차입니다.

### Q2. "필요 포트만 허용"하는 방화벽 정책의 중요성은 무엇인가?
*   **답변**: 서버에 열려 있는 모든 포트는 잠재적인 공격 통로(Attack Surface)입니다. UFW를 통해 서비스에 꼭 필요한 **20022(SSH)**와 **15034(APP)**만 허용하고 나머지를 차단(Default Deny)함으로써, 알 수 없는 서비스의 취약점을 통한 침입을 원천 봉쇄할 수 있습니다.

### Q3. 계정/그룹과 ACL을 통해 디렉토리를 분리하는 이유는 무엇인가?
*   **답변**: **최소 권한 원칙(Principle of Least Privilege)**을 실현하기 위함입니다. `agent-test` 계정은 테스트 파일에만 접근하고, 보안이 중요한 `api_keys`나 로그 파일에는 접근할 수 없도록 격리함으로써 내부 사용자에 의한 실수나 악의적인 데이터 유출 사고를 방지할 수 있습니다.

### Q4. 환경 변수(AGENT_HOME 등)로 실행 환경을 고정하는 이유는 무엇인가?
*   **답변**: 서비스가 설치된 경로가 사용자마다 다르거나 실행 위치에 따라 상대 경로가 바뀌면 프로그램이 오작동할 수 있습니다. 환경 변수를 사용하면 시스템 내 어디서든 **동일한 환경(Context)**에서 앱이 실행되도록 보장할 수 있으며, 관리 및 이식성이 높아집니다. `echo $AGENT_HOME` 명령어로 즉시 설정값을 검증할 수 있습니다.

### Q5. 쉘 스크립트로 시스템 상태를 수집하고 로그를 남기는 흐름의 장점
*   **답변**: 장애가 발생했을 때 과거의 상태를 알 수 없다면 원인 분석은 불가능해집니다. `monitor.sh`를 통해 주기적으로 CPU, 메모리, 포트 상태를 기록해 두면, 장애 발생 시점의 로그를 분석하여 **"언제, 어떤 자원이 부족해서 문제가 생겼는지"** 객관적으로 추적하고 재발 방지 대책을 세울 수 있습니다.

### Q6. Crontab 주기 실행과 로그 보존 정책(압축/삭제)이 왜 필요한가?
*   **답변**: 모니터링은 끊김 없이 지속되어야 하므로 `crontab`을 통한 자동화가 필수입니다. 하지만 무한정 생성되는 로그는 결국 **디스크 풀(Disk Full)** 장애를 유발하여 서버를 멈추게 만듭니다. 따라서 일정 기간이 지난 로그는 압축하여 용량을 줄이고, 오래된 데이터는 삭제하는 보존 정책이 서버의 가용성을 유지하는 데 핵심적인 역할을 합니다.

