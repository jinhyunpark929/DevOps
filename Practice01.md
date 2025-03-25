# 🐳 Docker 이미지 최적화 및 배포

## 📌 Overview
Spring Tool Suite에서 Gradle로 생성한 .jar 파일을 Docker 이미지로 만들고 배포하는 과정을 다룹니다. 이미지 생성 시 최적화 방법을 탐구하고, 각 단계별 Dockerfile과 실행 명령어, 이미지 크기 비교 및 최적화 결과를 제공합니다.

## 🎯 Docker Image Optimization 목표

- 🗜️ **이미지 크기 및 성능 최적화**
- 🔒 **보안 및 유지보수성 향상**
- 🚀 **효율성 및 재사용성 개선**

---

## 1. 🍦 Vanilla 이미지

### Dockerfile
```docker
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 주요 명령어
```bash
docker pull eclipse-temurin:17-jre
docker build -t myoptimg:1.0 .
docker run myoptimg:1.0
```

### 결과
- **이미지 크기**: 288MB
- 기본 JRE를 제공하여 호환성이 좋지만, 용량이 큰 편편

---

## 2. 🏔️ Alpine 이미지 활용

### 설명
- Alpine Image : 경량화된 Linux 배포판 기반 
- 필수적인 컴포넌트만 포함하여 보안성 개선

### Dockerfile
```docker
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN apk update && \
    apk upgrade && \
    rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*
```

### 주요 명령어
```bash
docker pull eclipse-temurin:17-jre-alpine
docker build -t myoptimg:3.0 .
```

### 결과
- **이미지 크기**: 206MB
- 경량화된 Alpine Linux 기반으로, 용량 줄일 수 있음

---

## 3. 🔬 Google Distroless 사용

### 설명
- Google Distroless 이미지는 최소한의 런타임 환경만 제공이
- 보안성 극대화 및 공격 표면 최소화
- 불필요한 셸이나 패키지 관리자가 없어 보안 취약점 감소 가능
-  고도의 보안이 요구되는 프로덕션 환경에 적합

### Dockerfile
```docker
FROM gcr.io/distroless/java17-debian11:latest
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 주요 명령어
```bash
docker pull gcr.io/distroless/java17-debian11:latest
docker build -t myoptimg:4.0 .
```

### 결과
- **이미지 크기**: 251MB
- 최소한의 구성으로 보안성이 향상되며, 불필요한 패키지 제거

---

## 4. 🧱 Layer Caching

### 설명
- Docker 빌드 프로세스를 최적화함
- 변경되지 않은 레이어는 재사용되어 빌드 시간 단축에 용이
- CI/CD 파이프라인에서 유용, 빈번한 빌드와 배포가 필요한 환경에서 효율성 향상

### Dockerfile
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

### 주요 명령어
```bash
docker build -t myoptimg:5.0 .
```

### 결과
- **이미지 크기**: 206MB
- 레이어 캐싱을 통해 빌드 속도가 향상

---

## 5. 🏗️ Multi-stage Build

### 설명
- 멀티 스테이지 빌드는 빌드 프로세스와 런타임 환경을 분리
- 최종 이미지에는 실행에 필요한 컴포넌트만 포함되어 이미지 크기 감고 및 보안성 향상
- 빌드 도구와 런타임 환경의 분리로 인해 유지보수가 용이

### Dockerfile
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

### 주요 명령어
```bash
docker build -t myoptimg:6.0 .
```

### 결과
- **이미지 크기**: 205MB
- 빌드와 실행 환경을 분리하여 최적화된 크기와 성능을 제공

---

## 🚀 Dockerhub에 Push하기

### 주요 명령어
```bash
docker tag myoptimg:6.0 jamie929/myoptimg:6.0
docker push jamie929/myoptimg:6.0
```

---

## 🔧 Troubleshooting

### 1. Dockerfile이 없는 경우
- 오류: `failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory`
- 해결: Dockerfile 이름 변경 또는 `-f` 옵션 사용
  ```bash
  docker build -t vanila -f DockerfileVanila .
  ```

### 2. Alpine 이미지에서 apt-get 사용 시 오류
- 오류: `/bin/sh: apt-get: not found`
- 해결: `apt-get` 대신 `apk` 사용
  ```docker
  RUN apk update && apk upgrade
  ```

### 3. Spring Boot 애플리케이션 실행 시 ClassNotFoundException
- 오류: `Could not find or load main class org.springframework.boot.loader.JarLauncher`
- 해결: 멀티 스테이지 빌드 사용 및 ENTRYPOINT 설정
  ```docker
  ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
  ```

---

## 📊 결론 및 참고 자료

**이미지 크기 비교표**

| 버전       | 베이스 이미지                 | 크기     | 특징 |
|------------|-------------------------------|----------|------|
| Vanilla    | eclipse-temurin:17-jre        | 288MB    | 기본 JRE, 호환성 좋음 |
| Alpine     | eclipse-temurin:17-jre-alpine | 206MB    | 경량화, 빠른 배포 |
| Distroless | gcr.io/distroless/java17-debian11 | 251MB | 보안성 높음, 최소 구성 |
| Layer Cache | eclipse-temurin:17-jre-alpine | 206MB   | 빌드 속도 향상 |
| Multi-stage | JDK + JRE (Alpine)            | 205MB    | 최적화된 크기와 성능 |

**Dockerhub Repository:** [jamie929/myoptimg](https://hub.docker.com/repository/docker/jamie929/myoptimg/tags)  
**참고 링크:** [Docker 공식 문서](https://docs.docker.com/)
