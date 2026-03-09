# 앤서블 0단계-1단계 따라하기 강의

## 문서 목적
이 문서는 앤서블을 처음 배우는 사람이 `0단계 준비`와 `1단계 초급 입문`을 실제로 따라 하면서 익힐 수 있도록 만든 실습형 강의 노트다. 단순히 명령만 실행하는 것이 아니라, 왜 이 과정이 필요한지와 실행 전후 무엇이 바뀌는지도 함께 설명한다.

## 학습 대상
- 앤서블을 처음 접하는 입문자
- 리눅스 명령어와 SSH가 아직 익숙하지 않은 학습자
- 플레이북 작성 전에 전체 흐름을 잡고 싶은 학습자

## 학습 목표
0단계를 마치면:
- 리눅스 터미널, SSH, YAML의 기본 개념을 이해한다.
- 앤서블 실습을 위한 테스트 환경을 준비할 수 있다.

1단계를 마치면:
- 인벤토리, 모듈, 태스크, 플레이북의 차이를 설명할 수 있다.
- 애드혹 명령과 플레이북으로 원격 노드를 점검하고 간단한 변경을 적용할 수 있다.
- `--check`, `-vvv`로 검증과 디버깅을 시작할 수 있다.

## 준비물
- 제어 노드 1대
  - 예: Ubuntu, WSL2 Ubuntu, Rocky Linux, macOS
- 관리 대상 노드 1대 이상
  - 예: 같은 PC의 다른 VM, VirtualBox VM, Multipass VM, 테스트용 리눅스 서버
- SSH 접속 가능 상태
- 관리 대상 노드에 Python 설치 가능 상태

경고:
- 이 문서는 테스트 환경 기준이다.
- 운영 서버에 바로 적용하지 말고, 먼저 VM 또는 로컬 테스트 환경에서 실습한다.

## 전체 실습 구조
1. 0단계 준비
2. 1단계 초급 입문
3. 최종 미니 프로젝트

---

## 0단계: 준비 단계

## 0-1. 앤서블을 배우기 전에 알아야 할 그림

### 핵심 요약
앤서블은 "제어 노드"에서 명령을 실행해 "관리 대상 노드"의 상태를 맞추는 도구다.

### 개념 설명
- 제어 노드:
  - 앤서블 명령을 실행하는 컴퓨터다.
  - 보통 개발자 노트북, 운영용 점프 서버, CI 서버가 이 역할을 한다.
- 관리 대상 노드:
  - 실제로 설정이 바뀌는 서버다.
  - 웹 서버, DB 서버, 테스트 VM 등이 여기에 해당한다.
- SSH:
  - 원격 리눅스 서버에 안전하게 접속하는 방식이다.
  - 앤서블은 기본적으로 SSH를 통해 원격 노드에 접속한다.
- Python:
  - 대부분의 앤서블 모듈은 원격 노드에서 Python을 사용해 동작한다.
  - 그래서 관리 대상 노드에 Python이 없는 경우가 자주 문제를 만든다.

비유:
제어 노드는 "작업 지시를 내리는 관리자", 관리 대상 노드는 "실제 작업이 수행되는 현장"이라고 생각하면 이해가 쉽다.

정확한 기술 설명:
앤서블은 에이전트를 미리 설치하지 않고 SSH로 연결해 모듈을 전송하고 실행한 뒤 결과를 회수한다. 이 구조 덕분에 비교적 가볍고 도입이 쉽다.

### 왜 필요한가
- 서버가 1대일 때는 수동 작업도 가능하다.
- 서버가 5대, 20대, 100대로 늘어나면 수동 작업은 누락과 실수를 만든다.
- 앤서블은 "같은 상태를 반복해서 안전하게 맞추는 것"에 강하다.

---

## 0-2. 리눅스 기본 명령 익히기

### 핵심 개념
앤서블을 잘 쓰려면 리눅스 명령어를 어느 정도 읽고 실행할 수 있어야 한다.

### 꼭 알아야 하는 명령

```bash
# 현재 위치 확인
pwd

# 파일 목록 보기
ls -al

# 디렉터리 만들기
mkdir -p ~/ansible-lab

# 파일 내용 보기
cat /etc/os-release

# 파일 편집기 예시
nano test.txt
```

### 실습
아래 명령을 한 줄씩 실행한다.

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
pwd
ls -al
cat /etc/os-release
```

### 실행 결과 확인 방법
- `pwd` 결과로 현재 작업 디렉터리를 확인한다.
- `cat /etc/os-release` 결과로 운영체제를 확인한다.

### 자주 틀리는 포인트
- 현재 디렉터리를 확인하지 않고 파일을 만들어서 위치를 잃어버리는 경우가 많다.
- `sudo`가 필요한 명령과 아닌 명령을 구분하지 못하면 권한 오류가 자주 난다.

---

## 0-3. SSH 이해와 키 기반 접속 준비

### 핵심 개념
SSH 키 기반 접속은 비밀번호 대신 공개키와 개인키로 인증하는 방식이다.

### 왜 필요한가
- 실무에서 비밀번호 반복 입력을 줄일 수 있다.
- 자동화 도구가 비대화형으로 접속하기 쉬워진다.
- 보안상 비밀번호보다 관리가 명확한 경우가 많다.

### 실습 1: SSH 키 확인 또는 생성

```bash
ls -al ~/.ssh
```

`id_ed25519`와 `id_ed25519.pub`가 없다면 생성한다.

```bash
ssh-keygen -t ed25519 -C "ansible-lab"
```

### 실습 2: 원격 접속 테스트

```bash
ssh <사용자>@<대상서버IP>
```

예시:

```bash
ssh vagrant@192.168.56.11
```

### 실행 전후 설명
- 실행 전: 내 PC는 대상 서버와 신뢰 관계가 없다.
- 실행 후: 공개키가 등록되어 있으면 비밀번호 입력 없이 접속할 수 있다.

### 검증 방법

```bash
ssh <사용자>@<대상서버IP> hostname
```

정상이라면 원격 서버의 호스트명이 출력된다.

### 자주 틀리는 포인트
- 공개키 `.pub` 파일이 아니라 개인키를 복사하는 실수
- 서버 사용자 계정을 잘못 입력하는 실수
- 방화벽이나 VM 네트워크 문제로 접속이 막히는 경우

디버깅 명령:

```bash
ssh -v <사용자>@<대상서버IP>
```

---

## 0-4. YAML 기초

### 핵심 개념
YAML은 사람이 읽기 쉬운 데이터 표현 형식이다. 앤서블 플레이북은 대부분 YAML로 작성한다.

### 왜 중요한가
앤서블 초급자가 가장 많이 막히는 부분이 YAML 들여쓰기 오류다. 문법이 복잡해서가 아니라 공백과 구조가 엄격하기 때문이다.

### 꼭 알아야 할 규칙
- 들여쓰기는 보통 2칸 사용
- 탭 대신 공백 사용
- 리스트는 `-`로 시작
- 딕셔너리는 `key: value` 형태

### 예제 1: 리스트

```yaml
# file: list-example.yml
packages:
  - nginx
  - curl
  - git
```

### 예제 2: 딕셔너리

```yaml
# file: dict-example.yml
web_server:
  package_name: nginx
  service_name: nginx
  port: 80
```

### 예제 3: 중첩 구조

```yaml
# file: nested-example.yml
users:
  - name: alice
    shell: /bin/bash
  - name: bob
    shell: /bin/zsh
```

### 실습
아래 파일 3개를 직접 만든다.

```text
~/ansible-lab/
├── dict-example.yml
├── list-example.yml
└── nested-example.yml
```

### 검증 방법
파일을 연 뒤 눈으로 구조를 확인한다.

```bash
cat list-example.yml
cat dict-example.yml
cat nested-example.yml
```

### 자주 틀리는 포인트
- 탭 문자를 써서 오류가 나는 경우
- `- name: alice` 아래 들여쓰기를 맞추지 않는 경우
- `key : value`처럼 불필요한 공백을 넣어 혼란을 만드는 경우

---

## 0-5. 실습 환경 설계

### 추천 구조
처음에는 복잡한 클라우드 환경보다 간단한 로컬 테스트 환경이 낫다.

예시:

```text
제어 노드: 내 노트북 또는 WSL
관리 대상 노드 1: Ubuntu VM
관리 대상 노드 2: Rocky Linux VM
```

### 왜 두 대가 좋은가
- 그룹 개념을 이해하기 쉽다.
- OS 차이를 비교하기 좋다.
- 같은 작업을 여러 노드에 적용하는 이유를 체감할 수 있다.

### 최소 점검 명령

```bash
ssh <사용자>@<노드1_IP> hostname
ssh <사용자>@<노드2_IP> hostname
python3 --version
```

관리 대상 노드에서 Python이 없으면 먼저 설치한다.

Ubuntu 계열:

```bash
sudo apt update
sudo apt install -y python3
```

Rocky Linux 계열:

```bash
sudo dnf install -y python3
```

### 0단계 완료 체크리스트
- SSH 키 기반 접속 가능
- 관리 대상 노드에 Python 존재
- YAML 예제 3개 직접 작성 완료
- 제어 노드와 관리 대상 노드 구분 설명 가능

---

## 1단계: 초급 입문

## 1-1. 앤서블 설치

### 핵심 개념
앤서블은 제어 노드에 설치한다. 관리 대상 노드에 별도 에이전트를 설치하는 방식이 아니다.

### 설치 예시
Ubuntu 또는 Debian 계열:

```bash
sudo apt update
sudo apt install -y ansible
```

설치 후 버전을 확인한다.

```bash
ansible --version
```

### 실행 결과 확인 방법
- `ansible [core x.y.z]` 형태의 버전 정보가 나오면 정상이다.

### 자주 틀리는 포인트
- 관리 대상 서버에 앤서블을 설치해야 한다고 오해하는 경우
- Python과 SSH만 있으면 충분한데 양쪽에 같은 도구를 모두 설치하려는 경우

---

## 1-2. 앤서블의 기본 구성 요소 이해

### 용어 정리
- inventory:
  - 앤서블이 관리할 대상 서버 목록이다.
- module:
  - 앤서블이 수행하는 실제 작업 단위다.
  - 예: 패키지 설치, 파일 복사, 서비스 시작
- task:
  - 플레이북 안에서 모듈을 한 번 호출하는 작업이다.
- playbook:
  - 여러 태스크를 순서대로 정의한 실행 문서다.
- become:
  - 관리자 권한으로 작업을 수행하는 기능이다.
- idempotency, 멱등성:
  - 같은 작업을 여러 번 실행해도 최종 결과가 같게 유지되는 성질이다.

### 왜 이 구분이 중요한가
- `inventory`는 대상 목록
- `module`은 도구
- `task`는 작업 지시 한 줄
- `playbook`은 작업 절차서

이 차이를 구분하지 못하면 플레이북 구조를 이해하기 어려워진다.

---

## 1-3. 첫 번째 인벤토리 만들기

### 실습 목표
관리 대상 노드 2대를 인벤토리에 등록한다.

### 디렉터리 구조

```text
~/ansible-lab/
├── inventory.ini
└── site.yml
```

### 예제 파일

```ini
# file: inventory.ini
[web]
node1 ansible_host=192.168.56.11 ansible_user=vagrant
node2 ansible_host=192.168.56.12 ansible_user=vagrant
```

### 개념 설명
- `[web]`:
  - 그룹 이름이다.
- `node1`, `node2`:
  - 앤서블 안에서 부를 호스트 별칭이다.
- `ansible_host`:
  - 실제 접속 IP 또는 DNS 이름이다.
- `ansible_user`:
  - SSH 로그인 계정이다.

### 검증 방법

```bash
cat inventory.ini
```

### 자주 틀리는 포인트
- 그룹 이름과 호스트 이름 사이 구조를 헷갈리는 경우
- 실제 접속 IP 대신 별칭만 적고 `ansible_host`를 누락하는 경우
- SSH 사용자 계정을 잘못 입력하는 경우

---

## 1-4. 첫 번째 애드혹 명령 실행

### 핵심 개념
애드혹 명령은 "한 번 빠르게 실행하는 단일 작업"에 적합하다.

### 왜 필요한가
플레이북을 쓰기 전에 접속, 권한, 모듈 동작을 빠르게 검증할 수 있다.

### 실습 1: 연결 확인

```bash
ansible all -i inventory.ini -m ping
```

### 예상 결과
- 성공 시 `pong`
- 실패 시 인증 오류, Python 오류, 접속 오류 메시지 표시

### 실습 2: 원격 호스트명 확인

```bash
ansible all -i inventory.ini -m command -a "hostname"
```

### 개념 설명
- `ping` 모듈:
  - 네트워크 ICMP 핑이 아니라 앤서블 접속 가능 여부를 확인하는 모듈이다.
- `command` 모듈:
  - 원격에서 명령을 실행하지만 셸 해석은 하지 않는다.

경고:
- `command`와 `shell`은 비슷해 보여도 다르다.
- 단순 명령 실행은 `command`가 더 안전하다.

### 자주 틀리는 포인트
- `ping`을 일반 네트워크 핑으로 오해하는 경우
- 셸 기능이 필요한데 `command`를 써서 실패하는 경우

---

## 1-5. 자주 쓰는 기본 모듈 체험

### 실습 1: 파일 만들기

```bash
ansible all -i inventory.ini -b -m file -a "path=/tmp/ansible_test.txt state=touch mode=0644"
```

### 개념 설명
- `file` 모듈:
  - 파일과 디렉터리의 존재 여부, 권한, 소유자 상태를 관리한다.
- `-b`:
  - `become`의 축약형이다.
  - 관리자 권한이 필요한 작업일 때 사용한다.

### 실습 2: 파일 복사

먼저 로컬에 샘플 파일을 만든다.

```bash
echo "Hello Ansible" > index.html
```

그다음 원격 노드로 복사한다.

```bash
ansible web -i inventory.ini -b -m copy -a "src=index.html dest=/tmp/index.html mode=0644"
```

### 실습 3: 서비스 상태 확인

```bash
ansible web -i inventory.ini -b -m service -a "name=nginx state=started enabled=yes"
```

### 경고
- `service` 모듈은 서비스가 이미 설치되어 있어야 정상 동작한다.
- 아직 nginx가 없다면 먼저 설치 단계가 필요하다.

---

## 1-6. 플레이북이 왜 필요한가

### 핵심 개념
애드혹 명령은 빠르지만 반복 작업을 문서화하고 재사용하기 어렵다. 플레이북은 여러 작업을 순서대로 기록하는 자동화 절차서다.

### 실무에서 줄어드는 문제
- 누가 어떤 순서로 작업했는지 모호한 상태
- 서버마다 다른 수동 설정
- 재실행 시 결과가 달라지는 문제

---

## 1-7. 첫 번째 플레이북 작성

### 실습 목표
웹 서버 패키지를 설치하고, 서비스를 시작하고, 테스트용 HTML 파일을 배포한다.

### 예제 파일

```yaml
# file: site.yml
- name: Configure basic web server
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present

    - name: Ensure nginx service is enabled and started
      service:
        name: nginx
        state: started
        enabled: true

    - name: Deploy sample index page
      copy:
        src: index.html
        dest: /var/www/html/index.html
        mode: "0644"
```

### 개념 설명
- `hosts: web`
  - 인벤토리의 `web` 그룹을 대상으로 실행한다.
- `become: true`
  - 패키지 설치와 서비스 제어는 보통 관리자 권한이 필요하다.
- `package` 모듈:
  - 운영체제별 패키지 관리자를 추상화해 공통 인터페이스로 제공한다.
- `copy` 모듈:
  - 제어 노드 파일을 원격 노드로 전송한다.

### 실행 명령

```bash
ansible-playbook -i inventory.ini site.yml
```

### 실행 전후 설명
- 실행 전: nginx가 없거나, 서비스가 내려가 있거나, 기본 페이지가 다를 수 있다.
- 실행 후: nginx가 설치되고 실행되며, 샘플 HTML 파일이 배포된다.

### 검증 방법

```bash
ansible web -i inventory.ini -m command -a "systemctl is-active nginx" -b
ansible web -i inventory.ini -m command -a "cat /var/www/html/index.html"
```

### 멱등성 확인
같은 플레이북을 한 번 더 실행한다.

```bash
ansible-playbook -i inventory.ini site.yml
```

두 번째 실행에서는 대부분 `ok`가 나오고, 불필요한 `changed`가 줄어드는 것이 이상적이다.

### 왜 중요한가
멱등성이 보장되면 "이미 설정된 서버를 다시 실행했을 때 망가질까?"라는 불안을 크게 줄일 수 있다.

---

## 1-8. `--check`와 `-vvv` 사용법

### `--check`
실제로 변경하지 않고 무엇이 바뀔지 예측하는 모드다.

```bash
ansible-playbook -i inventory.ini site.yml --check
```

### `-vvv`
상세 로그를 보여줘 디버깅에 도움을 주는 옵션이다.

```bash
ansible-playbook -i inventory.ini site.yml -vvv
```

### 언제 쓰는가
- 운영 반영 전에 변경 예상 범위를 보고 싶을 때
- SSH 인증 문제, Python 경로 문제, 권한 문제를 더 자세히 보고 싶을 때

### 자주 틀리는 포인트
- `--check` 결과를 100% 실제 결과와 동일하다고 생각하는 경우
- 일부 모듈은 체크 모드에서 완전히 같은 정보를 주지 못할 수 있다

---

## 1-9. 초급자용 디버깅 포인트

### 1. SSH 연결 실패
증상:
- `UNREACHABLE`

주요 원인:
- IP 오류
- 사용자 계정 오류
- SSH 키 문제
- 네트워크 문제

확인 명령:

```bash
ssh -v <사용자>@<대상서버IP>
```

### 2. Python 인터프리터 오류
증상:
- Python 관련 모듈 오류

주요 원인:
- 대상 서버에 Python 미설치

확인 명령:

```bash
ansible all -i inventory.ini -m raw -a "python3 --version"
```

### 3. 권한 상승 실패
증상:
- `sudo` 또는 권한 관련 오류

주요 원인:
- `become` 누락
- 대상 사용자에게 sudo 권한 없음

확인 방법:
- 수동으로 SSH 접속 후 `sudo -l` 실행

### 4. YAML 문법 오류
증상:
- 플레이북 파싱 실패

주요 원인:
- 들여쓰기 오류
- 리스트와 딕셔너리 구조 혼동

습관:
- 처음에는 짧은 파일로 시작하고 조금씩 확장한다.

---

## 1-10. 미니 프로젝트: 웹 서버 1대 초기 세팅 자동화

### 목표
한 번의 플레이북 실행으로 웹 서버를 기본 상태로 만든다.

### 파일 구조

```text
~/ansible-lab/
├── index.html
├── inventory.ini
└── site.yml
```

### index.html 예제

```html
<html>
  <body>
    <h1>Ansible Lab</h1>
    <p>Deployed by Ansible</p>
  </body>
</html>
```

### 실행 순서
1. `inventory.ini` 작성
2. `index.html` 작성
3. `site.yml` 작성
4. `ansible all -i inventory.ini -m ping` 실행
5. `ansible-playbook -i inventory.ini site.yml --check` 실행
6. `ansible-playbook -i inventory.ini site.yml` 실행
7. 서비스 상태와 파일 배포 결과 확인

### 결과 검증

```bash
ansible web -i inventory.ini -b -m command -a "systemctl status nginx --no-pager"
ansible web -i inventory.ini -m command -a "curl -I http://localhost"
```

### 실무 연결
이 미니 프로젝트는 실제 운영에서 다음 작업의 축소판이다.
- 신규 서버 부트스트랩
- 공통 웹 서버 초기 세팅
- 테스트 환경 자동 생성
- 표준 설정 반복 배포

---

## 0단계-1단계 학습 점검 질문
- 제어 노드와 관리 대상 노드의 차이를 설명할 수 있는가
- 인벤토리와 플레이북의 차이를 설명할 수 있는가
- `command`와 `shell`의 차이를 대략 설명할 수 있는가
- `become`이 왜 필요한지 설명할 수 있는가
- 멱등성이 왜 운영 자동화에서 중요한지 설명할 수 있는가

---

## 다음 학습 단계
1단계를 마쳤다면 다음으로 넘어갈 주제는 아래다.

1. 변수
2. 조건문 `when`
3. 반복문 `loop`
4. 핸들러
5. `group_vars`, `host_vars`

다음 단계부터는 "한 번 실행되는 자동화"보다 "환경별로 재사용 가능한 자동화"를 만드는 훈련이 시작된다.
