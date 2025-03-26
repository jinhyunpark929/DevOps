# 🐳 Docker Image Guide

## 🧱 Image Overview

- 컨테이너 실행에 필요한 빌드된 패키지  
- **이미지 = 설계도 / 컨테이너 = 실행 결과물**
- 이미지를 기반으로 컨테이너 생성 가능

---

## 🔍 Using Existing Docker Images

### 🔎 Image 검색

```bash
docker pull <이미지명>:<태그>
# 예시
docker pull openjdk:17
```

- Docker Hub에서 이미지 가져오기
- 로컬에 존재하면 다운로드 생략됨

### 🚀 Image 실행

```bash
docker run -it <이미지명>:<태그>
# 예시
docker run -it openjdk:17
```

- 이미지를 기반으로 컨테이너 실행

### 🔗 Local Directory 연결

```bash
docker run -it -v $(pwd):/app <이미지명>
# 예시
docker run -it -v $(pwd):/app openjdk:17
```

- 현재 디렉토리를 컨테이너 내부 `/app`과 마운트

### 🧹 Image 삭제

```bash
docker rmi <이미지명>:<태그>
```

- 사용하지 않는 이미지 제거
- 이미지명 또는 이미지 ID로 삭제 가능

## 🗂️ Docker Image Layer Structure

- Docker 이미지는 여러 개의 **레이어(layer)**로 구성됨
- Dockerfile 작성 방식에 따라 레이어 수 결정
- 불필요한 레이어가 많을수록 이미지 크기 증가  
  → 빌드 및 배포 속도에 부정적 영향

#### 이미지 크기 증가 시 영향

- 빌드/배포 시간 증가
- Kubernetes와 같은 플랫폼 사용 시 비용 증가  
  (네트워크 전송, 저장소 공간 부담 증가)

---

## ⚙️ Docker Image Optimization and Deployment

### 🎯 Optimization 목표

- 🗜️ 이미지 크기 및 성능 최적화
- 🔒 보안 및 유지보수성 향상
- 🔁 효율성 및 재사용성 개선

## 🍦 Vanilla Image 활용

### 📄 Dockerfile 예시

```docker
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 💡 주요 명령어

```bash
docker pull eclipse-temurin:17-jre
docker build -t myoptimg:1.0 .
docker run myoptimg:1.0
```

### 📦 결과

- 이미지 크기: **288MB**
- 호환성 높지만 이미지 용량이 큼

## 🏔️ Alpine Image 활용

### 📘 설명

- 경량화된 Linux 배포판 기반
- 필수 컴포넌트만 포함하여 보안성 개선

### 📄 Dockerfile 예시

```docker
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN apk update && \
    apk upgrade && \
    rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*
```

### 💡 주요 명령어

```bash
docker pull eclipse-temurin:17-jre-alpine
docker build -t myoptimg:3.0 .
```

### 📦 결과

- 이미지 크기: **206MB**
- 경량 이미지로 용량 최적화 가능

## 🔬 Distroless Image 활용

### 📘 설명

- Google Distroless는 최소 런타임 환경 제공
- 쉘, 패키지 관리자 미포함 → 보안성 극대화
- 프로덕션 환경에 적합

### 📄 Dockerfile 예시

```docker
FROM gcr.io/distroless/java17-debian11:latest
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 💡 주요 명령어

```bash
docker pull gcr.io/distroless/java17-debian11:latest
docker build -t myoptimg:4.0 .
```

### 📦 결과

- 이미지 크기: **251MB**
- 보안성과 최소 구성에 강점

## 🔁 Layer Caching 활용

### 📘 설명

- 변경되지 않은 레이어는 재사용되어 빌드 속도 단축
- CI/CD 환경에서 빌드 효율성 향상

### 📄 Dockerfile 예시

```docker
FROM eclipse-temurin:17-jre-alpine
RUN apk update && \
    apk upgrade && \
    rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

### 💡 주요 명령어

```bash
docker build -t myoptimg:5.0 .
```

### 📦 결과

- 이미지 크기: **206MB**
- 빌드 시간 단축 효과

## 🏗️ Multi-stage Build 활용

### 📘 설명

- 빌드 환경과 런타임 환경을 분리
- 최종 이미지에는 실행에 필요한 파일만 포함
- 보안성 향상 및 용량 최적화

### 📄 Dockerfile 예시

```docker
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /build
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /build/dependencies/ ./
COPY --from=builder /build/spring-boot-loader/ ./
COPY --from=builder /build/snapshot-dependencies/ ./
COPY --from=builder /build/application/ ./
CMD ["java", "-jar", "app.jar"]
```

### 💡 주요 명령어

```bash
docker build -t myoptimg:6.0 .
```

### 📦 결과

- 이미지 크기: **205MB**
- 성능 및 보안성 개선된 최적화 구조

## 🚀 DockerHub Push하기

### 💡 주요 명령어

```bash
docker tag myoptimg:6.0 jamie929/myoptimg:6.0
docker push jamie929/myoptimg:6.0
```

---

## 🧰 Troubleshooting

### ❌ Dockerfile이 없는 경우

- 오류 메시지:
  ```
  failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
  ```
- 해결 방법:

```bash
docker build -t vanila -f DockerfileVanila .
```

### ⚠️ Alpine에서 apt-get 사용 시 오류

- 오류 메시지: `/bin/sh: apt-get: not found`
- 해결 방법: `apk` 명령어 사용

```docker
RUN apk update && apk upgrade
```

### 🚫 Spring Boot 실행 오류 (ClassNotFoundException)

- 오류 메시지:
  ```
  Could not find or load main class org.springframework.boot.loader.JarLauncher
  ```
- 해결 방법: 멀티 스테이지 빌드 및 ENTRYPOINT 설정

```docker
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

---

## 📊 Summary

| Version      | Base Image                          | Size   | Feature                            |
|--------------|-------------------------------------|--------|------------------------------------|
| Vanilla      | eclipse-temurin:17-jre              | 288MB  | 기본 JRE, 호환성 좋음              |
| Alpine       | eclipse-temurin:17-jre-alpine       | 206MB  | 경량화, 빠른 배포                  |
| Distroless   | gcr.io/distroless/java17-debian11   | 251MB  | 보안성 높음, 최소 구성             |
| Layer Cache  | eclipse-temurin:17-jre-alpine       | 206MB  | 빌드 속도 향상                     |
| Multi-stage  | eclipse-temurin:17-{jdk,jre}-alpine | 205MB  | 빌드/런타임 분리, 성능 최적화     |

### 🧠 Learning Insight

> 최적화는 주어진 문제에 대해 가장 효율적인 해결 방안을 찾아가는 과정을 뜻한다.
> Docker와 Docker Image는 개발 및 운영 환경을 효율적으로 관리하기 위해 필요한 것이다.
> Dockerfile의 구성 방식이 전체적인 최적화 성능에 중요한 영향을 미친다.
> 그렇기 때문에 로커 환경에 부하를 주지 않는 방향으로 작성하는 것이 중요하다.

**🔗 DockerHub Repository:** [jamie929/myoptimg](https://hub.docker.com/repository/docker/jamie929/myoptimg/tags)  
**🔗 Reference:** [Docker Docs](https://docs.docker.com/)
