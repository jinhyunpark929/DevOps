## Docker 이미지 최적화 및 배포

### 개요
이 문서는 Docker 이미지를 최적화하여 빌드하고 배포하는 다양한 방법을 설명합니다. 각 단계별로 사용된 Dockerfile과 실행 명령어를 포함하며, 이미지 크기 비교 및 최적화 결과를 제공합니다.

---

## 1. Vanilla 이미지

### Dockerfile
```docker
# 실행 스테이지
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 주요 명령어
1. **이미지 다운로드**  
   ```bash
   docker pull eclipse-temurin:17-jre
   ```
2. **이미지 생성**  
   ```bash
   docker build -t myoptimg:1.0 .
   ```
3. **컨테이너 실행**  
   ```bash
   docker run myoptimg:1.0
   ```

### 결과
- **이미지 크기**: 288MB  
- **Docker 이미지 확인**:
  ```bash
  docker images
  ```

---

## 2. Alpine 이미지 활용

### Dockerfile
```docker
# 실행 스테이지
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN apk update && \
    apk upgrade && \
    rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*
```

### 주요 명령어
1. **이미지 다운로드**  
   ```bash
   docker pull eclipse-temurin:17-jre-alpine
   ```
2. **이미지 생성**  
   ```bash
   docker build -t myoptimg:3.0 .
   ```

### 결과
- **이미지 크기**: 206MB  
- **Docker 이미지 확인**:
  ```bash
  docker images
  ```

---

## 3. Google Distroless 사용

### Dockerfile
```docker
# 실행 스테이지
FROM gcr.io/distroless/java17-debian11:latest
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
```

### 주요 명령어
1. **이미지 다운로드**  
   ```bash
   docker pull gcr.io/distroless/java17-debian11:latest
   ```
2. **이미지 생성**  
   ```bash
   docker build -t myoptimg:4.0 .
   ```

### 결과
- **이미지 크기**: 251MB  
- **Docker 이미지 확인**:
  ```bash
  docker images
  ```

---

## 4. Layer Caching

### Dockerfile 수정 (레이어 캐싱 활용)
```docker
# 실행 스테이지 (최적화된 레이어 구성)
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
1. **이미지 생성**  
   ```bash
   docker build -t myoptimg:5.0 .
   ```

### 결과
- **이미지 크기**: 206MB  
- 레이어 캐싱을 통해 빌드 속도 향상.

---

## 5. Multi-stage Build

### Dockerfile (멀티 스테이지 빌드)
```docker
# 빌드 스테이지 (JDK 사용)
FROM eclipse-temurin:17-jdk-alpine AS builder

WORKDIR /build

COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar

RUN java -Djarmode=layertools -jar app.jar extract

# 실행 스테이지 (JRE 사용)
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=builder /build/dependencies/ ./
COPY --from=builder /build/spring-boot-loader/ ./
COPY --from=builder /build/snapshot-dependencies/ ./
COPY --from=builder /build/application/ ./

CMD ["java", "-jar", "app.jar"]
```

### 주요 명령어
1. **이미지 생성**  
   ```bash
   docker build -t myoptimg:6.0 .
   ```

### 결과
- **이미지 크기**: 205MB  
- 멀티 스테이지 빌드를 통해 최적화된 크기와 성능 제공.

---

## Dockerhub에 Push하기

### 주요 명령어 (태그 및 Push)
1. **태그 지정**
   ```bash
   docker tag myoptimg:6.0 jamie929/myoptimg:6.0
   ```
2. **Dockerhub에 Push**
   ```bash
   docker push jamie929/myoptimg:6.0
   ```

---

## Troubleshooting

### 1. Dockerfile이 없는 경우
오류 메시지: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory

원인: Dockerfile이 현재 디렉토리에 존재하지 않음. 예를 들어, 파일 이름이 DockerfileVanila인 경우, Dockerfile을 기본 이름으로 변경하거나, docker build 명령어에 -f 옵션을 사용하여 파일을 지정해야 합니다.

해결 방법:

Dockerfile을 기본 이름으로 변경하거나, 명령어에 -f 옵션을 사용합니다.

bash
docker build -t vanila -f DockerfileVanila .

참고
Dockerfile 명명 규칙
- 기본 명명 규칙: Dockerfile의 Default Name은 **Dockerfile**, Default 사용 시 docker build 명령어를 간단하게 실행 가능
- 사용자 정의 명명 규칙: 여러 Dockerfile을 사용하는 경우, <purpose>.Dockerfile 형식으로 명명

### 2. Alpine 이미지에서 apt-get 사용 시 오류
오류 메시지: /bin/sh: apt-get: not found

원인: Alpine Linux는 apt-get 대신 apk를 사용

해결 방법:

Dockerfile에서 apt-get 명령어를 apk로 변경

text
### 실행 스테이지
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN apk update && \
    apk upgrade && \
    rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*
    
# 3. Spring Boot 애플리케이션 실행 시 ClassNotFoundException
오류 메시지: Could not find or load main class org.springframework.boot.loader.JarLauncher

원인: Spring Boot 애플리케이션을 실행할 때 JarLauncher 클래스 확인 불가

해결 방법:

- 멀티 스테이지 빌드를 사용하여 Spring Boot 애플리케이션 수정
- ENTRYPOINT 명령어를 사용하여 JarLauncher를 실행할 수 있도록 설정

text
### 빌드 스테이지
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /build
COPY step01_basic-0.0.1-SNAPSHOT.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

### 실행 스테이지
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /build/dependencies/ ./
COPY --from=builder /build/spring-boot-loader/ ./
COPY --from=builder /build/snapshot-dependencies/ ./
COPY --from=builder /build/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]

## 결론 및 참고 자료

**이미지 크기 비교표**

| 버전       | 베이스 이미지                 | 크기     |
|------------|-------------------------------|----------|
| Vanilla    | eclipse-temurin:17-jre        | 288MB    |
| Alpine     | eclipse-temurin:17-jre-alpine | 206MB    |
| Distroless | gcr.io/distroless/java17-debian11 | 251MB    |
| Layer Cache | eclipse-temurin:17-jre-alpine | 206MB    |
| Multi-stage | JDK + JRE (Alpine)            | 205MB    |

- Alpine 이미지는 Vanilla보다 경량화되었으며, Distroless는 보안성과 최소한의 구성으로 유리
- 멀티 스테이지 빌드를 통해 최적화된 이미지를 생성 가능

---

**Dockerhub Repository:** [jamie929/myoptimg](https://hub.docker.com/repository/docker/jamie929/myoptimg/tags)  
**참고 링크:** [Docker 공식 문서](https://docs.docker.com/)
