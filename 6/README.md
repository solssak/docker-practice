## 이전 강의에서 진행한 작업
- Node Express API(백엔드), React SPA(프론트엔드), MongoDB(데이터베이스)로 구성된 애플리케이션을 도커 컨테이너로 구축함.
- 각 컨테이너가 서로 통신할 수 있도록 설정하고, 데이터 영구 저장 및 소스 코드 업데이트 기능 추가.
- 해당 애플리케이션을 실행하려면 여러 개의 도커 명령을 실행해야 하는 번거로움이 있음.

## 현재 설정의 문제점
- 실행 과정이 복잡하며, 다음과 같은 작업을 수동으로 수행해야 함.
  - 네트워크 생성
  - MongoDB 컨테이너 실행 (환경 변수 및 볼륨 추가)
  - Node API 이미지 빌드 (한 번만 실행)
  - Node 백엔드 컨테이너 실행 (환경 변수 설정, 볼륨 추가, 네트워크 연결)
  - React SPA 이미지 빌드
  - React 컨테이너 실행 (바인드 마운트 설정, 포트 게시, 인터랙티브 모드 실행)
  - 모든 컨테이너 종료 및 네트워크, 볼륨 제거
- 현재는 컨테이너가 3개뿐이지만, 더 많은 컨테이너가 추가되면 복잡성이 더욱 증가할 것.

## 이번 학습 목표
1. Docker Compose란 무엇인지 이해하기.
2. Docker Compose를 사용하는 방법 배우기.
3. 기존 애플리케이션을 Docker Compose로 변환하여 더 편리하게 실행하는 방법 익히기.

## Docker Compose란?
- Docker Compose는 다중 컨테이너 애플리케이션을 쉽게 관리할 수 있도록 도와주는 도구.
  - `docker build` 및 `docker run` 명령을 대체하는 도구.
- 설정 프로세스를 자동화하고, 단 하나의 명령으로 모든 컨테이너를 실행 및 종료할 수 있도록 함.
- 개별적인 환경 설정을 유지하면서 전체 애플리케이션을 단순하게 관리 가능.

## Docker Compose의 장점
- 하나의 파일로 모든 설정 관리 → 공유와 재사용이 용이함.
- 반복적인 명령 실행 불필요 → 개발 생산성 향상.
- 단순한 명령 실행 → `docker compose up` 한 줄로 전체 앱 실행 가능.
- 멀티 컨테이너 환경에서 강력한 효과 → 컨테이너가 많아질수록 관리가 편리함.

### 헷갈릴 수 있는 부분
- Dockerfile을 대체하지 않음 → 여전히 필요함.
- 개별 이미지나 컨테이너를 대체하지 않음 → 컨테이너 실행을 더 쉽게 할 뿐임.
- 다중 호스트 컨테이너 관리 도구가 아님 → 단일 호스트에서 다중 컨테이너 관리에 적합.

## Docker Compose로 기존 MongoDB 컨테이너 구성하기
**1. 기존 MongoDB 컨테이너 실행 명령**
```bash
docker run -d --rm --name mongodb \
  -v data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=max \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  --network app-network \
  mongo
```
**2. 위 명령을 docker-compose.yml 파일로 변환**
```bash
version: '3.8'

services:
  mongodb:
    image: "mongo"
    volumes:
      - data:/data/db
    environment: // 환경변수를 .env 파일로 분리 가능
      MONGO_INITDB_ROOT_USERNAME: max
      MONGO_INITDB_ROOT_PASSWORD: secret

volumes:
  data:

networks: // Compose는 모든 컨테이너를 동일한 네트워크에 자동 추가. 기본 네트워크 외에 특정 네트워크 추가 가능.
  custom-network:
```
**3.한줄로 모든 서비스 실행하기** 
`docker-compose up`
1. 필요한 컨테이너를 한 번에 실행할 수 있고
2. 로컬에 없는 이미지가 있다면 자동으로 다운을 진행하며
3. 지정된 네트워크와 볼륨도 자동으로 설정해서
4. 프로젝트별로 독립적인 환경을 생성할 수 있다.

기본적으로 attached 모드로 실행되며 `-d`키워드를 통해 Detached 모드에서 실행이 가능하다.

서비스를 중지하고 싶다면 `docker-compose down`를 사용하면 된다.
실행중인 컨테이너가 중지되고, 자동 생성된 네트워크가 삭제되지만, **볼륨은 삭제되지 않는다.**
볼륨까지 삭제하고 싶다면 `-v` 키워드를 사용할 수 있다.

## Docker Compose로 기존 Backend 컨테이너 구성하기
**1. docker-compose.yml 작성**
```bash
version: '3.8'

services:
  mongodb:
    ...

  backend:
    build: ./backend
    ports:
      - "80:80"
    volumes:
      - logs:/app/logs // 로그 데이터를 저장하는 named volume
      - ./backend:/app // bind mount로 로컬 코드와 동기화
      - /app/node_modules // anonymous volume 유지
    env_file:
      - ./env/backend.env
    depends_on: // Mongo DB가 먼저 실행되도록 보장
      - mongodb

volumes:
  data:
  logs:

```

이후 동일하게 `docker-compose up -d` 키워드를 입력하면
- MongoDB 컨테이너 실행
- Backend 컨테이너도 자동 빌드 & 실행
- 이후 두 컨테이너가 자동으로 네트워크 연결

## Docker Compose로 기존 Frontend 컨테이너 구성하기
**1. docker-compose.yml 작성**
```bash
version: '3.8'

services:
  mongodb:
    ...

  backend:
    ...

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
    // interactive(-it) 모드 활성화하기
    stdin_open: true // 표준 입력 열어두기
    tty: true // 터미널 연결 활성화
    depends_on: // 백엔드가 먼저 실행되도록 보장
      - backend

volumes:
  data:
  logs:

```

## Docker Compose 조금 더 자세히 알아보기
`docker-compose up`과 `docker-compose down`과 함께 사용할 수 있는 추가 기능을 알아보자.

### 1. `--build`옵션: 강제로 이미지를 다시 빌드하기
기본적으로 `docker-compose up`을 실행하면
한 번 빌드된 이미지는 다시 빌드되지 않고 그대로 사용됨
코드 변경 이후에는 강제로 다시 빌드를 해줘야함.

`docker-compose up --build -d` 명령어를 사용하면 
1. 빌드를 다시 실행하고
2. 변경된 소스 코드가 포함된 새로운 이미지를 생성 후
3. 컨테이너를 자동으로 다시 실행한다.

### 2. `docker-compose build` 명령어: 컨테이너 실행 없이 이미지 빌드만 수행
만약, 컨테이너를 실행하지 않고, 이미지 빌드만 미리 해두고 싶다면
`docker-compose build`명령어를 사용해
1. 컨테이너 실행 없이, 빌드만 수행한다.
2. 커스텀 이미지를 미리 생성해놓을 때 유용하다.
3. 이후 `docker-compose up -d`를 사용하면 기존에 빌드된 이미지를 사용한다.

### 3. 컨테이너 이름 관리: `container_name` 옵션 사용하기
기본적으로 Docker Compose는 컨테이너 이름을 자동으로 생성한다.
예를 들어, 프로젝트 폴더가 `/docker-complete` 라면:

|서비스 이름|자동 생성된 컨테이너 이름|
|---|---|
|frontend|docker-complete_frontend_1|
|backend|docker-complete_backend_1|

직접 컨테이너 이름을 설정할 수 있다.
```bash
services:
  mongodb:
    image: "mongo"
    container_name: mongodb  # 컨테이너 이름을 'mongodb'로 지정
```
### 요약 정리
**명령어 요약**
|명령어|설명|
|---|---|
|docker-compose up -d|컨테이너 실행 (백그라운드 모드)|
|docker-compose up --build -d|이미지를 강제로 다시 빌드 후 실행|
|docker-compose build|컨테이너 실행 없이, 빌드만 수행|
|docker-compose down|모든 컨테이너 정리 (볼륨 제외)|
|docker-compose down -v|모든 컨테이너 & 볼륨까지 삭제|

**컨테이너 이름 직접 설정하는 법**
```bash
services:
  mongodb:
    image: "mongo"
    container_name: mongodb // docker-compose up -d 실행 후, docker ps에서 컨테이너 이름이 mongodb로 변경됨
```

## 마무리
길고 복잡한 `docker run` 명령어 대신 하나의 `docker-compose.yml`파일로 간단하게 컨테이너의 실행을 정의할 수 있다.

여러개의 컨테이너를 다룰 다룰 때에만 유용하다고 생각할 수도 있지만, 단일 컨테이너 환경에서도 유용하다.
- `docker run` 명령어를 반복 입력할 필요가 없다.
- bind mount 설정이 더 간편해진다.
- 환경 변수 세팅이 가능하다.
- 프로젝트 실행 방식을 선언적으로 문서화할 수 있다. (마치 Dockerfile 처럼)
**즉, 유지보수와 협업 측면에서 장점이 있다.**

Docker Compose는 Dockerfile을 대신하는 것이 아니라 함께 작동하는 도구.
Dockerfile은 개별 컨테이너의 빌드 과정을 정의하고,
Docker Compose는 여러 컨테이너를 조합하여 실행할 수 있도록 도와준다.

즉,
✅ Dockerfile → 이미지 빌드를 위한 설정 파일
✅ Docker Compose → 이미지를 조합하여 실행하는 도구
