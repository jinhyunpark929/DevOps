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

## 이미지 크기 비교표

| 버전       | 베이스 이미지                 | 크기     |
|------------|-------------------------------|----------|
| Vanilla    | eclipse-temurin:17-jre        | 288MB    |
| Alpine     | eclipse-temurin:17-jre-alpine | 206MB    |
| Distroless | gcr.io/distroless/java17-debian11 | 251MB    |
| Layer Cache | eclipse-temurin:17-jre-alpine | 206MB    |
| Multi-stage | JDK + JRE (Alpine)            | 205MB    |

---

## 결론 및 참고 자료

- Alpine 이미지는 Vanilla보다 경량화되었으며, Distroless는 보안성과 최소한의 구성으로 유리합니다.
- 멀티 스테이지 빌드를 통해 최적화된 이미지를 생성할 수 있습니다.
- Dockerhub에서 최적화된 이미지를 공유하여 재사용 가능합니다.

**Dockerhub Repository:** [jamie929/myoptimg](https://hub.docker.com/repository/docker/jamie929/myoptimg/tags)  
**참고 링크:** [Docker 공식 문서](https://docs.docker.com/)
