# empty-AI-SW-basic
# 📋 시스템 관제 자동화 프로젝트 수행 내역서
24시간 쉬지 않는 인공지능 경비원 설치 과정
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

    * 집주인 보호 (Root 접속 차단): 모든 권한을 가진 집주인 계정으로 직접 들어오는 것을 막아, 비밀번호 하나로 집 전체가 털리는 것을 방지합니다.
    
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
## 📖 SSH와 Root: 서버 관리의 기초 개념

서버라는 '디지털 요새'를 관리하기 위해 반드시 이해해야 하는 두 가지 핵심 요소입니다.

---

### 1. SSH (Secure Shell) - "안전한 원격 통로"

| 항목 | 상세 설명 |
| :--- | :--- |
| **의미** | **S**ecure(보안) **Sh**ell(명령어 창)의 약자로, 네트워크를 통해 다른 컴퓨터에 안전하게 접속하기 위한 프로토콜입니다. |
| **왜 필요한가?** | 예전의 접속 방식은 데이터가 평문으로 전송되어 해킹에 취약했습니다. SSH는 모든 데이터를 **암호화**하여 보안을 유지합니다. |
| **언제 사용하는가?** | 원격지에 있는 서버를 내 컴퓨터에서 제어하고 싶을 때(파일 전송, 설정 변경 등) 언제나 사용합니다. |
| **어디서 작동하나?** | 서버의 네트워크 입구(기본 22번 포트)에서 대기하며 외부 사용자의 신호를 기다립니다. |
| **어떻게 기능하나?** | 공개키/개인키 암호화 방식을 사용하여 접속하려는 사람이 본인이 맞는지 확인하고, 연결된 후의 모든 대화를 암호화합니다. |



---

### 2. Root (루트) - "절대 권한의 뿌리"

| 항목 | 상세 설명 |
| :--- | :--- |
| **의미** | 시스템의 최상위 계정으로, 나무의 **'뿌리'**처럼 모든 권한이 시작되는 리눅스의 절대 관리자 계정(Superuser)입니다. |
| **왜 필요한가?** | 일반 사용자는 시스템 설정이나 중요한 파일을 건드릴 수 없습니다. 시스템 전체를 관리하거나 보안을 설정하려면 반드시 Root 권한이 필요합니다. |
| **언제 사용하는가?** | 프로그램 설치, 사용자 관리, 시스템 업데이트, 보안 설정 등 **관리자 전용 업무**를 수행할 때만 잠시 사용합니다. |
| **어디서 작동하나?** | 리눅스 시스템 전체에서 통용되며, 파일 시스템의 최상위 경로(`/`)부터 모든 자원에 접근할 수 있습니다. |
| **어떻게 기능하나?** | 모든 보안 검사를 통과하며, 어떤 파일이든 읽고 쓰고 실행할 수 있는 무소불위의 권한을 행사합니다. |



---

### 3. 왜 이 둘의 보안이 그토록 중요한가요?

해커에게 **"SSH를 뚫고 Root 권한을 얻는다"**는 것은 서버의 심장을 완전히 장악했다는 뜻입니다.

1.  **공격의 관문 (SSH)**: 해커가 서버 내부로 들어오는 유일한 길입니다. 이 길이 뚫리면 우리 집 안방까지 해커가 들어오게 됩니다.
2.  **공격의 목표 (Root)**: 해커가 가장 탐내는 보물입니다. Root 권한만 얻으면 서버 내의 모든 정보를 빼가거나 서버를 완전히 파괴할 수 있습니다.

---

### 💡 요약: 왜 보안 설정을 하나요?
* **SSH 포트 변경**: 해커가 우리 집으로 들어오는 **'통로' 자체를 못 찾게** 하기 위해 (비밀 통로 만들기)
* **Root 접속 차단**: 설령 해커가 통로를 찾았더라도, **'가장 강력한 열쇠(Root)'**는 외부인이 절대 만질 수 없게 안방 금고에 숨겨두기 위해 (보안 강화)

## 🔐 SSH 보안 강화 (포트 변경 및 Root 접속 차단)

서버의 기본 출입문인 SSH 설정을 변경하여 해커의 공격 경로를 차단하고 관리자 권한을 보호하는 핵심 보안 작업입니다.

---

### 🧐 왜 SSH 설정을 변경해야 하나요?

서버가 인터넷에 연결되는 순간, 전 세계의 해킹 봇들이 **'22번 포트'**와 **'root'** 계정을 타겟으로 공격을 시작합니다. 
* **포트 변경**: 모두가 아는 정문을 닫고 나만 아는 **비밀 통로**를 만듭니다.
* **Root 차단**: 집의 모든 권한을 가진 **마스터 키(Root)**가 외부로 노출되지 않도록 원천 봉쇄합니다.

---

### 📝 명령어 한 줄 해석 (언제, 어디서, 어떻게, 왜)

1. `sudo sed -i 's/^#\?Port .*/Port 20022/' /etc/ssh/sshd_config`
   * **언제/어디서**: 서버 초기 세팅 시, SSH 설정 파일 내에서 접속 번호를 바꿀 때 사용합니다.
   * **어떻게/왜**: 기본 22번 포트를 20022번으로 수정하여 자동화된 해킹 공격을 피합니다.
   * **명령어 구조**: `sed -i 's/기존포트패턴/새포트설정/' 파일경로`

2. `sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config`
   * **언제/어디서**: 포트 변경과 동시에, 관리자 계정의 직접 노출을 막기 위해 실행합니다.
   * **어떻게/왜**: 외부에서 Root 계정으로 바로 로그인하는 것을 금지(no)하여 보안 사고를 예방합니다.
   * **명령어 구조**: `sed -i 's/기존루트로그인패턴/루트로그인금지설정/' 파일경로`

3. `sudo systemctl restart ssh`
   * **언제/어디서**: 설정 파일의 텍스트를 수정한 직후, 시스템에 반영하기 위해 사용합니다.
   * **어떻게/왜**: SSH 서비스를 재시작하여 경비원이 바뀐 보안 규칙(20022번 포트 사용)을 읽게 합니다.
   * **명령어 구조**: `systemctl [명령] [서비스명]`

4. `ss -tuln | grep 20022`
   * **언제/어디서**: 모든 설정을 마친 후, 실제로 문이 잘 열렸는지 확인할 때 사용합니다.
   * **어떻게/왜**: 20022번 포트가 현재 '대기(Listen)' 상태인지 내 눈으로 직접 검증합니다.
   * **명령어 구조**: `네트워크상태확인 | 특정글자찾기`

---

### 🔍 명령어 상세 분석 (약자 및 의미)

| 명령어/옵션 | 약자 및 뜻 | 사용법 및 왜 사용하는가? | 결과 |
| :--- | :--- | :--- | :--- |
| **`sed`** | **S**tream **Ed**itor | 파일을 열지 않고 텍스트를 자동으로 수정할 때 사용 | 설정 파일 내용 변경 |
| **`-i`** | **i**n-place | 수정 내용을 파일에 즉시 저장하기 위해 사용 | 파일 덮어쓰기 완료 |
| **`Port`** | 포트 (통로) | 데이터가 드나드는 문 번호를 지정함 | 접속 통로 번호 확정 |
| **`RootLogin`** | 루트 로그인 | 최고 관리자 계정으로의 접속 허용 여부 | 관리자 접근 권한 설정 |
| **`systemctl`** | **system** **c**on**t**rol | 시스템 서비스(SSH 등)를 제어하고 관리함 | 서비스 상태 변경 권한 |
| **`restart`** | 재시작 | 변경된 설정값을 실제 시스템에 적용하기 위해 사용 | 서비스 초기화 및 재가동 |
| **`ss`** | **s**ocket **s**tatistics | 현재 컴퓨터의 네트워크 연결 통계를 보여줌 | 열린 포트 목록 출력 |
| **`grep`** | **G.R.E.P** (찾기) | 방대한 데이터 중 내가 원하는 글자만 필터링할 때 사용 | 20022 포트 존재 확인 |

---

### 🧩 `sed` 명령어 속 정규 표현식 해부하기

명령어에 포함된 `'s/^#\?PermitRootLogin .*/PermitRootLogin no/'`의 의미를 쪼개어 분석합니다.

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
* **이유**: `PermitRootLogin` 뒤에 기존에 어떤 설정값이 있든 상관없이 **그 줄 전체를 싹 다 잡아서 지우기 위해** 사용합니다.

#### 4. `/PermitRootLogin no/` : 새로운 값으로 덮어쓰기
* **의미**: 위에서 찾은 지저분한 줄 전체를 지우고, 그 자리에 아주 깔끔하게 `PermitRootLogin no`라는 글자를 딱 써 넣으라는 명령입니다.

> **💡 정규식 한 줄 요약**
> "주석(`#`)이 달려 있든 아니든, `PermitRootLogin`으로 시작하는 줄을 통째로 찾아서, 깨끗하게 `PermitRootLogin no`로 바꿔버려!"

---

### 💡 최종 요약
이 작업은 **"모두가 아는 22번 대문을 폐쇄하고 20022번 비밀 쪽문을 만든 뒤, 집주인(Root)은 정면으로 들어오지 못하게 못 박아 보안을 2중으로 강화하는 과정"**입니다.

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
## 🛡️ 방화벽(Firewall)의 이해

### 1. 방화벽의 정의와 의미
* **사전적 의미**: 건물에 화재가 발생했을 때 불길이 번지지 않도록 막는 **'방화벽'**에서 유래되었습니다.
* **디지털적 의미**: 외부에서 들어오는 네트워크 트래픽(데이터)과 내부에서 나가는 트래픽을 감시하고, **허가되지 않은 접근을 차단**하는 보안 시스템입니다.

### 2. 방화벽은 왜 필요한가요? (Why)
* **무단 침입 방지**: 서버의 모든 문(Port)을 열어두는 것은 대문을 열어놓고 외출하는 것과 같습니다. 방화벽은 허가된 사람만 들어오게 합니다.
* **불필요한 노출 차단**: 서버 내부에서 돌아가는 서비스 중 외부로 보여줄 필요가 없는 통로들을 숨겨서 해커의 타겟이 되지 않도록 보호합니다.
* **안전한 통신 보장**: 신뢰할 수 있는 IP 주소나 특정 통신 방식(TCP/UDP)만 골라서 허용할 수 있습니다.

### 3. 언제, 어디서, 어떻게 사용하나요?
* **언제 (When)**: 서버를 인터넷에 연결하는 순간부터 **24시간 내내** 가동되어야 합니다. 특히 새로운 서비스를 설치하거나 포트 번호를 바꿨을 때 설정을 변경합니다.
* **어디서 (Where)**: 서버의 **네트워크 입구**에서 작동합니다. 외부 세계와 우리 서버가 만나는 접점에서 모든 데이터를 검사합니다.
* **어떻게 (How)**: 
    1. 기본 정책을 **'모든 접속 차단(Deny)'**으로 설정합니다.
    2. 그중 우리가 꼭 사용해야 하는 통로(예: 20022, 15034)만 **'허용(Allow)'** 목록에 추가합니다.
    3. 이렇게 작성된 **'화이트리스트(허용 명부)'**를 기준으로 데이터를 필터링합니다.

### 4. 주요 기능 (Function)
1. **패킷 필터링**: 들어오고 나가는 데이터(패킷)의 주소와 포트 번호를 확인하여 통과시킬지 결정합니다.
2. **접근 제어**: 특정 IP 주소나 국가에서의 접속을 막거나 허용합니다.
3. **로깅(Logging)**: 누가 언제 우리 서버의 문을 두드렸는지 기록을 남겨, 나중에 침입 흔적을 분석할 수 있게 합니다.

---

### 💡 비유로 이해하기
> 방화벽은 클럽 입구를 지키는 **'보안 요원(가드)'**과 같습니다.
> 
> * **클럽 내부**: 우리 서버
> * **입구**: 포트(Port)
> * **출입 명부**: 방화벽 규칙(UFW Rule)
> 
> 보안 요원은 명부에 이름이 있는 사람(허용된 포트)만 들여보내고, 명단에 없거나 수상해 보이는 사람은 입구에서 즉시 돌려보냅니다.

## 🧱 방화벽(UFW) 설정 및 포트 허용 가이드

서버 보안의 핵심인 방화벽을 설치하고, 우리가 사용할 전용 통로(20022, 15034)를 안전하게 개방하는 과정입니다.

---

### 🧐 왜 방화벽 포트를 허용해야 하나요?

방화벽은 서버의 **'출입 통제소'**입니다. 
기본적으로 방화벽은 모든 문을 꽉 잠가두는데, 우리가 변경한 **SSH 포트(20022)**나 **서비스용 포트(15034)**를 방화벽에서 허용(`allow`)해주지 않으면, 우리 자신도 서버에 접속할 수 없게 됩니다. 즉, **"안전하게 검증된 문만 열어주기 위해"** 사용합니다.

---

### 📝 방화벽(UFW) 설정 명령어 상세 해석

각 명령어가 어떤 구조로 이루어져 있고, 언제 왜 사용하는지 정리한 내역입니다.

#### 1. `sudo ufw status verbose`
* **사용 구조**: `ufw status [옵션]`
* **언제/어디서**: 작업을 시작하기 전 터미널에서 현재 방화벽 상태를 확인할 때 사용합니다.
* **어떻게/왜**: 현재 어떤 문이 열려 있는지 상세히(verbose) 파악하여 중복 설정을 방지합니다.

#### 2. `sudo apt update && sudo apt install ufw -y`
* **사용 구조**: `apt [명령] [패키지명] [옵션]`
* **언제/어디서**: 서버에 방화벽 프로그램(UFW)이 없을 때 설치하기 위해 사용합니다.
* **어떻게/왜**: 최신 패키지 목록을 불러오고(`update`), 자동으로 '예'를 선택(`-y`)하여 UFW를 설치합니다.

#### 3. `sudo ufw allow 20022/tcp`
* **사용 구조**: `ufw allow [포트번호]/[프로토콜]`
* **언제/어디서**: SSH 포트를 20022로 변경한 후, 이 문을 통과시켜야 할 때 사용합니다.
* **어떻게/왜**: 20022번 통로를 TCP 프로토콜(신뢰성 있는 연결)로 열어 원격 접속을 허용합니다.

#### 4. `sudo ufw allow 15034/tcp`
* **사용 구조**: `ufw allow [포트번호]/[프로토콜]`
* **언제/어디서**: 특정 서비스(애플리케이션)가 15034 포트를 사용할 때 실행합니다.
* **어떻게/왜**: 서비스 운영을 위해 해당 포트로 들어오는 신호가 차단되지 않도록 통로를 개방합니다.

#### 5. `sudo ufw --force enable`
* **사용 구조**: `ufw [옵션] enable`
* **언제/어디서**: 모든 포트 설정을 마친 후, 방화벽을 실제 가동할 때 사용합니다.
* **어떻게/왜**: 사용자에게 다시 묻지 않고(`--force`) 즉시 방화벽을 활성화하여 모든 보안 규칙을 적용합니다.

#### 6. `sudo ufw status`
* **사용 구조**: `ufw status`
* **언제/어디서**: 모든 작업이 끝난 후 최종적으로 상태를 점검할 때 사용합니다.
* **어떻게/왜**: 내가 허용(`ALLOW`)한 포트들이 목록에 정상적으로 등록되었는지 내 눈으로 직접 확인합니다.

---

### 🔍 명령어 상세 분석 (약자 및 의미)

| 명령어/옵션 | 약자 및 뜻 | 사용법 및 왜 사용하는가? | 결과 |
| :--- | :--- | :--- | :--- |
| **`ufw`** | **U**ncomplicated **F**ire**w**all | 복잡한 방화벽 설정을 쉽게 하기 위해 사용 | 방화벽 제어 권한 획득 |
| **`apt`** | **A**dvanced **P**ackage **T**ool | 리눅스에서 프로그램을 설치/삭제할 때 사용 | 프로그램 설치 관리 |
| **`status`** | 상태 (Status) | 현재 방화벽이 켜져 있는지, 어떤 포트가 열렸는지 확인 | 포트 리스트 출력 |
| **`allow`** | 허용하다 (Allow) | 특정 포트로 들어오는 데이터를 통과시키기 위해 사용 | 해당 포트 개방 |
| **`tcp`** | **T**ransmission **C**ontrol **P**rotocol | 데이터 전송 시 '신뢰성'을 보장하는 통신 방식 지정 | 안정적인 연결 설정 |
| **`enable`** | 활성화 (Enable) | 설정된 규칙에 따라 방화벽 감시를 시작함 | 보안 시스템 가동 |
| **`--force`** | 강제로 (Force) | 실행 중 묻는 질문(Y/N)을 생략하고 즉시 실행 | 즉각적인 명령 수행 |
| **`verbose`** | 상세히 (Verbose) | 기본 정보보다 더 자세한 설정 내역을 보고 싶을 때 사용 | 상세 로그 출력 |

---

### 💡 요약
이 과정은 **"우리 집 출입 통제소(UFW)를 설치하고, 우리가 새로 만든 비밀문(20022)과 서비스문(15034)만 통과할 수 있도록 명부에 등록한 뒤, 경비원에게 정식 근무를 시작(enable)하라고 명령하는 것"**과 같습니다.  

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
## 📚 주요 개념 이해하기

서버 보안 설정을 이해하기 위해 반드시 알아야 할 두 가지 핵심 개념입니다.

---

### 1️⃣ SSH (Secure Shell)
> **"안전하게 원격으로 접속하는 비밀 터널"**

* **의미**: **S**ecure(보안) **Sh**ell(명령어 창)의 약자입니다.
* **역할**: 멀리 떨어져 있는 서버에 내 컴퓨터로 접속하여 명령을 내릴 수 있게 해주는 **'통로'**입니다.
* **특징**: 옛날 방식(Telnet 등)과 달리 모든 데이터를 **암호화**하여 주고받습니다. 해커가 중간에 데이터를 가로채도 내용을 알 수 없어 안전합니다.
* **사용처**:
    * 집이나 외부에서 학교/회사 서버에 접속할 때
    * AWS, Google Cloud 등 클라우드 서버를 관리할 때
    * 서버실에 가지 않고 내 노트북으로 서버 설정을 변경할 때

---

### 2️⃣ Root (루트)
> **"모든 것을 할 수 있는 절대 권한자"**

* **의미**: 식물의 **'뿌리'**라는 뜻으로, 리눅스의 모든 파일과 설정이 시작되는 지점을 의미합니다.
* **역할**: 시스템의 **최고 관리자 계정**입니다. 윈도우의 관리자 권한보다 훨씬 강력한 힘을 가집니다.
    * 시스템 전체 삭제 가능
    * 사용자 생성 및 삭제 권한
    * 모든 보안 설정 변경 가능
* **주의점**: 너무 강력해서 위험합니다. 실수로 `rm -rf /` (전체 삭제) 명령을 내리면 서버가 즉시 파괴되므로 주의가 필요합니다.
* **사용처**:
    * 새로운 소프트웨어를 설치할 때
    * 시스템 보안 설정(포트 변경 등)을 바꿀 때
    * 다른 사용자들의 권한을 관리할 때

---

### 3️⃣ 왜 이 둘을 함께 설정해야 하나요?

해커에게 서버 해킹이란 **"SSH라는 통로를 통해 Root라는 절대 권한을 뺏는 것"**을 의미합니다. 이를 막기 위해 이중 잠금 장치를 합니다.

1.  **SSH 포트 변경 (통로 은폐)**: 해커가 우리 집으로 들어오는 **'통로' 자체를 못 찾게** 하여 공격 시도 자체를 줄입니다.
* 모르는 번호의 문(20022)으로 들어와서, 일반인 열쇠로 먼저 확인받아야만 서버 입장이 가능하도록 설계한 것입니다.
* 전 세계의 해커들은 봇(Bot)을 이용해 전형적인 SSH 포트인 **22번**을 대상으로 무차별 대입 공격(Brute-force)을 시도합니다.
* 포트를 **20022**와 같은 비표준 포트로 변경하는 것만으로도 자동화된 공격의 90% 이상을 회피할 수 있습니다(Security by Obscurity).
  
2.  **Root 접속 차단 (열쇠 보호)**:
   설령 해커가 통로를 찾아냈더라도, **'가장 강력한 열쇠(Root)'**는 아예 문 밖으로 가지고 나갈 수 없게 못 박아버리는 것입니다.
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

