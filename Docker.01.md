# Docker.01

## 🐳 Docker Basic Concepts

### 🔧 Docker

- 컨테이너 생성 및 실행을 위한 **플랫폼 겸 도구**
- **컨테이너 생성, 실행, 관리 기능 제공 소프트웨어**

### 📦 Container

- 가상화 기술의 일종으로 **기존 가상머신 방식보다 경량화 및 고속화된 형태**
- 애플리케이션과 실행에 필요한 라이브러리, 설정 등을 **하나의 독립 실행 단위로 패키징한 구조**
- 자체 파일시스템, 환경변수, 네트워크, 실행 중 프로세스를 포함한 **가벼운 가상 실행 환경**

### 🧱 Image

- 애플리케이션 실행에 필요한 구성 요소(앱, 라이브러리, 설정 등)를 포함한 **파일 모음**
- **이미지 = 설계도 / 컨테이너 = 이미지 실행 결과물**
- **컨테이너 생성을 위한 기반 역할 수행**

---

## ⚙️ Basic Docker Installation Process

### 🔍 Prerequisites Installation

```bash
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```

- `ca-certificates` : SSL 인증서 관련 패키지
- `curl` : URL 기반 데이터 수신 도구
- `gnupg` : GPG(GNU Privacy Guard) 설치
- `lsb-release` : 배포판 정보 확인용 유틸리티

### 🗂️ Docker Repository Addition

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- 시스템 아키텍처와 우분투 버전에 맞는 **Docker 저장소 자동 추가**
- APT 저장소 : 우분투에서 소프트웨어 패키지를 다운로드 및 설치 가능한 서버 목록

### 🛠️ Docker Engine Installation

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
```

- `docker-ce` : Docker 커뮤니티 에디션
- `docker-ce-cli` : Docker CLI 도구
- `containerd.io` : 컨테이너 실행용 런타임
- `docker-buildx-plugin` : 고급 빌드 지원 플러그인
- `docker-compose-plugin` : Compose 기능 확장 플러그인

---

## 🚀 Basic Docker Commands

### 🧭 Command Format

```bash
docker run [옵션] <레지스트리>/<이미지이름>:<태그>
```

- 일반적으로 `docker run -it 이미지명` 사용
- 예: `openjdk:17` → `docker.io/library/openjdk:17`

### 🖥️ Execution Example

```bash
myserver01@ubuntu:~$ docker start 1310b5c7db24 : Container Execution
1310b5c7db24

myserver01@ubuntu:~$ docker ps : Check Running Containers
CONTAINER ID   IMAGE        COMMAND    CREATED         STATUS          PORTS     NAMES
1310b5c7db24   openjdk:17   "jshell"   7 minutes ago   Up 49 seconds             blissful_hopper

myserver01@ubuntu:~$ docker exec -it 1310b5c7db24 bash : Execute Container Internal Shell 
bash-4.4#
```

---

### 🧠 Learning Insight

> 도커는 리눅스 기반의 명령어로 제어하기 때문에 실습을 통해 체화하는 것이 필수적이다.
> 특히 도커의 컨테이너와 이미지는 추상적인 개념으로 존재하기 때문에, pwd, ls -l 같은 기본 명령어를 활용해 파일 시스템을 체계적으로 파악해야 한다.
