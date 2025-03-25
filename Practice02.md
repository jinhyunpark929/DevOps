# 🚀 **Automating Continuous Deployment**

## 🎯 **Goal**
Jenkins를 활용하여 자동화된 빌드 및 배포 파이프라인을 구축합니다.

- **CI 단계**: GitHub 업데이트 후 Jenkins에서 JAR 파일로 빌드하여 지정된 폴더에 저장
- **CD 단계**: 빌드된 파일을 운영 서버(user02)로 안전하게 전송하고 실행

---

## 🛠️ **1. Jenkins Bind Mount 설정**

### 📂 **로컬 디렉토리 생성**
```bash
mkdir jarappdir
```

### 🐳 **Jenkins 컨테이너 실행**
```bash
docker run --name myjenkins -p 8080:8080 -v $(pwd)/jarappdir:/var/jenkins_home/appjar jenkins/jenkins:lts-jdk17
```

- **`$(pwd)`**: 현재 작업 디렉토리
- **`$(pwd)/jarappdir`**: 호스트 디렉토리 경로 (현재 작업 디렉토리 하위의 `jarappdir` 폴더)
- **`/var/jenkins_home/appjar`**: 컨테이너 내부 디렉토리 경로 (Jenkins 컨테이너의 빌드 결과물 저장 위치)

---

## 🔧 **2. Jenkins 서버 내 inotify-tools 설치**

### 📥 **설치 명령어**
```bash
sudo apt-get update
sudo apt-get install inotify-tools
```

### 📖 **inotify-tools란?**
리눅스 파일 시스템 모니터링 도구로, 아래와 같은 기능을 제공합니다:
- 파일 생성, 수정, 삭제 감지
- 디렉토리 변경 사항 모니터링
- 파일 접근 및 속성 변경 추적

### ⚙️ **주요 명령어**
- `inotifywait`: 파일 시스템 이벤트 감시 및 대기
- `inotifywatch`: 파일 시스템 이벤트 통계 수집

### ✅ **패키지 설치 확인 명령어**
```bash
dpkg -l | grep inotify-tools
```

출력 예시:
```
ii  inotify-tools   3.22.6.0-4   amd64  command-line programs providing a simple interface to inotify
```

---

## 📦 **3. 운영 서버로 파일 자동 이관 스크립트 작성**

### 📝 **스크립트 내용**

```bash
#!/bin/bash

# 감시할 파일 경로
FISA_FILE="/home/myserver01/jarappdir/step07_cicd-0.0.1-SNAPSHOT-plain.jar"
COOLDOWN=10  # 재실행 대기 시간 (초)
LAST_RUN=0   # 마지막 실행 시간 초기화

# 운영 서버 정보
TARGET_SERVER="192.168.142.131"
TARGET_DIR="/home/myserver02/appjar/"
SSH_USER="myserver02"

# inotifywait를 사용하여 파일 변경 감지
inotifywait -m -e close_write "$FISA_FILE" |
while read -r directory events filename; do
    CURRENT_TIME=$(date +%s)
    
    if (( CURRENT_TIME - LAST_RUN > COOLDOWN )); then
        echo "$(date): $filename 파일이 수정되었습니다. 운영 서버로 전송합니다."
        
        # SCP를 사용하여 운영 서버로 파일 전송
        scp "$FISA_FILE" "$SSH_USER@$TARGET_SERVER:$TARGET_DIR"

        LAST_RUN=$CURRENT_TIME  # 마지막 실행 시간 업데이트
    else
        echo "$(date): 대기 중입니다. 일정 시간이 지나야 다시 실행 가능합니다."
    fi
done
```

### ▶️ **스크립트 실행 방법**
1. 스크립트 파일 저장:
   ```bash
   nano autoupdate.sh
   ```
   위 내용을 붙여넣고 저장합니다.
   
2. 실행 권한 부여:
   ```bash
   chmod +x autoupdate.sh
   ```

3. 스크립트 실행:
   ```bash
   ./autoupdate.sh
   ```

---

crontab 설정

![스크린샷 2025-03-25 233005](https://github.com/user-attachments/assets/6a2edba2-a9f6-41c0-9f32-87748a1cc5f9)
![스크린샷 2025-03-26 000232](https://github.com/user-attachments/assets/1ab0b69c-a492-4b1a-9f05-d1ee9ca69e93)
![스크린샷 2025-03-26 000207](https://github.com/user-attachments/assets/5fd20661-7dd3-402d-a9e9-f86825723e0b)
![스크린샷 2025-03-25 235800](https://github.com/user-attachments/assets/cc0c1587-fcae-4ea5-9735-c23ada5ed94d)
![스크린샷 2025-03-25 235619](https://github.com/user-attachments/assets/3bc5661f-26db-49bd-9b41-4f8a5b765414)
![스크린샷 2025-03-25 235117](https://github.com/user-attachments/assets/e8ab1983-452d-4cfb-a622-85a0376cface)

회고
6시간 내내 경로 하나 제대로 못 잡아서 빌드랑 배포가 계속 실패했다. 처음에는 좌절스러웠지만, 이런 시행착오가 오히려 시스템 구성에서 세세한 설정이 중요함을 직접 경험해볼 수 있었다.

이번 학습을 통해 내가 가장 크게 느낀 것은 시스템 경로 관리의 중요성이다. CI/CD 파이프라인을 실제로 구축해보니, 경로 설정 하나가 전체 프로세스의 성패를 결정한다는 것을 뼈저리게 경험했다.

아무리 자동화 스크립트를 잘 짜도 경로가 틀리면 아무 소용이 없다. 특히 실무에서는 더 복잡한 빌드 프로세스를 다뤄야 할 텐데, 그때는 수동으로 배포할 시간도 없을 것이다. 그래서 정확한 경로 설정을 기반으로 한 자동화된 배포 시스템이 정말 중요하다고 느꼈다.
