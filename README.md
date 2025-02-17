# 이미지란?
- 컨테이너의 청사진
- 실제로 코드와 코드를 실행하는데 필요한 도구를 포함하는 **공유 가능한 패키지**
- 이미지는 한번만 정의하면 여러개의 컨테이너를 생성 가능
=> 이미지를 통해 필요한 환경을 설정하고, 컨테이너를 실행하여 애플리케이션을 구동.

이미지 = 코드 및 실행 환경 포함 / 컨테이너 = 실행 중인 애플리케이션
## 컨테이너 실행 및 상태 확인
- 기본적으로 docker run node 실행 시, 컨테이너가 생성되지만 **격리된 상태**라 직접 상호작용할 수 없음.
- docker ps -a 명령으로 생성된 모든 컨테이너 목록 확인 가능.
## 대화형 모드로 컨테이너 실행
- docker run -it node
- -it 플래그 추가 시 **인터랙티브 모드(터미널에서 직접 실행 가능)** 로 컨테이너 실행.
- 실행된 컨테이너 내부에서 **Node.js REPL** 사용 가능 (예: 1 + 1 입력 시 결과 출력).
## 컨테이너와 로컬 환경의 독립성
- 컨테이너 내부에서 실행되는 **노드 버전(Node.js 14.9)** 과, 로컬 시스템에 설치된 노드 버전(Node.js 14.7)이 다름.
- 즉, **컨테이너 내부 환경은 호스트 시스템과 독립적**.
- 따라서, **로컬에 Node.js를 설치하지 않고도** 컨테이너 내부에서 Node.js 실행 가능.
## 여러 개의 컨테이너 실행 가능
- 동일한 이미지 기반으로 **여러 개의 컨테이너 생성 가능**.
- 예: docker run node 명령을 여러 번 실행하면, 같은 Node.js 이미지를 기반으로 한 컨테이너가 여러 개 실행됨.
- docker ps -a 실행 시, 생성된 컨테이너 목록을 확인할 수 있음.
# 커스텀 이미지 빌드하기
```
// Dockerfile

FROM node

WORKDIR /app

COPY . /app

RUN npm install

EXPOSE 80

CMD ["node",  "server.js"]
```
| 명령어 | 설명 |
|--------|---------------------------------------------------|
| `FROM node` | Node.js가 설치된 기본 이미지를 사용 |
| `WORKDIR /app` | 컨테이너 내부의 작업 디렉토리를 `/app`으로 설정 |
| `COPY . /app` | 호스트(로컬)에서 컨테이너의 `/app` 디렉토리로 모든 파일 복사 |
| `RUN npm install` | 패키지 의존성 설치 (`node_modules` 포함) |
| `EXPOSE 80` | 컨테이너 내부에서 사용할 포트(80) 지정 (외부 접근은 `-p` 옵션 필요) |
| `CMD ["node", "server.js"]` | 컨테이너가 실행될 때 `node server.js`를 실행하여 서버 시작 |

#### 1. Dockerfile 기반으로 이미지 생성하기
```
docker build -t my-app
```
#### 2. 이미지가 정상적으로 생성되었는지 확인
```
docker images
```
#### 3. 컨테이너 실행
```
docker run -p 3000:80 my-app
```
✅ 중요한 점
Dockerfile의 EXPOSE 80은 **도큐먼테이션 용도**이며, **실제 포트 연결을 위해 -p 옵션을 추가해야 함.**
## 정리

| 명령어                                                   | 설명                                           |
| ----------------------------------------------------- | -------------------------------------------- |
| `docker build -t my-app .`                            | `Dockerfile`을 기반으로 `my-app`이라는 이름의 이미지를 생성   |
| `docker images`                                       | 생성된 도커 이미지 목록 확인                             |
| `docker run -p 3000:80 my-app`                        | `my-app` 이미지를 기반으로 컨테이너 실행 (포트 3000 → 80 매핑) |
| `docker ps`                                           | 현재 실행 중인 컨테이너 목록 확인                          |
| `docker ps -a`                                        | 실행 중 + 중지된 모든 컨테이너 목록 확인                     |
| `docker stop <컨테이너 ID>`                               | 실행 중인 컨테이너 중지                                |
| `docker rm <컨테이너 ID>`                                 | 중지된 컨테이너 삭제                                  |
| `docker rmi <이미지 ID>`                                 | 도커 이미지 삭제                                    |
| `docker logs <컨테이너 ID>`                               | 특정 컨테이너의 로그 확인                               |
| `docker exec -it <컨테이너 ID> bash`                      | 실행 중인 컨테이너에 접속하여 Bash 셸 실행                   |
| `docker run -d -p 3000:80 --name my-container my-app` | 백그라운드에서 `my-container` 이름으로 컨테이너 실행          |

# Docker의 레이어 기반 아키텍쳐 및 최적화 방법
## 레이어 개념
docker의 이미지는 여러개의 레이어로 구성된다.
이미지 재빌드 속도는 캐싱되기 때문에 속도가 빨라짐

### Dockerfile 최적화 하기
🚀 최적화 전 (비효율적인 Dockerfile)
```
FROM node
WORKDIR /app
COPY . /app
RUN npm install
EXPOSE 80
CMD ["node", "server.js"]
```
• **문제점:** COPY . /app으로 모든 파일을 복사한 후 RUN npm install을 실행하므로, **코드가 변경될 때마다 npm install이 다시 실행됨**.
✅ 최적화 후 (효율적인 Dockerfile)
```
FROM node
WORKDIR /app

COPY package.json /app
RUN npm install

COPY . /app

EXPOSE 80
CMD ["node", "server.js"]
```
`COPY package.json /app`라인을 추가해서 의존성 변화가 없다면 의존성 설치도 `image layer`에 캐싱됨

| 개념              | 설명                                                              |
| --------------- | --------------------------------------------------------------- |
| **레이어 기반 아키텍처** | Docker는 각 명령어(`FROM`, `COPY`, `RUN` 등)를 **별도의 레이어**로 저장         |
| **캐시 활용**       | 변경되지 않은 레이어는 **다시 실행하지 않고 캐시 사용**                               |
| **비효율적 구조**     | 모든 파일을 복사 후 `npm install` 실행 → **코드 변경 시 `npm install` 다시 실행됨** |
| **최적화 방법**      | `package.json`만 먼저 복사 → `npm install` 실행 → 나머지 파일 복사            |
| **결과**          | 코드 변경 시에도 `npm install`을 재실행하지 않음 → **빌드 속도 개선**                |


# Docker 컨테이너 실행 모드

| 실행 모드                       | 설명                                   |
| --------------------------- | ------------------------------------ |
| **Attached 모드 (기본값)**       | 터미널에 연결된 상태에서 컨테이너 실행, **로그 실시간 출력** |
| **Detached 모드 (-d 플래그 사용)** | 백그라운드에서 컨테이너 실행, 터미널과 분리됨            |

#### `docker run`과 `docker start` 모드 차이

| 명령어               | 실행 방식                           | 특징                    |
| ----------------- | ------------------------------- | --------------------- |
| `docker run`      | **새 컨테이너를 생성하여 실행**             | 기본적으로 **Attached 모드** |
| `docker run -d`   | **새 컨테이너를 백그라운드에서 실행**          | **Detached 모드로 실행됨**  |
| `docker start`    | **중지된 컨테이너를 재시작**               | 기본적으로 **Detached 모드** |
| `docker start -a` | **중지된 컨테이너를 재시작 (Attached 모드)** | 실행 즉시 컨테이너의 출력 확인 가능  |
# Docker에서 웹 서버가 아닌 애플리케이션 실행
#### 도커는 웹 서버 전용이 아니다
- 일반적으로 도커는 **웹 애플리케이션과 웹 서버**를 실행하는 데 자주 사용됨.
- 하지만 도커는 **웹 서버뿐만 아니라 CLI 기반 애플리케이션**도 실행 가능.
- 예제: **Python 기반의 랜덤 숫자 생성기**(입력값을 받고 결과를 출력하는 프로그램).

CLI 애플리케이션을 `attached` 모드로 실행하는 것이 포인트!
# Docker 이미지 상세 정보 조회
- docker image inspect는 이미지의 세부 정보를 JSON 형식으로 제공
- 이미지의 생성 날짜, 기본 실행 명령, 환경 변수, 노출된 포트, 레이어 구조 등을 확인할 수 있음
 - --format 옵션을 사용하면 필요한 정보만 필터링 가능
- 레이어 기반 아키텍처를 확인하여 이미지 최적화를 위한 인사이트를 얻을 수 있음

| 활용 목적               | 명령어 |
|------------------------|----------------------------------------------|
| 이미지 전체 정보 확인  | `docker image inspect <이미지 ID 또는 이름>` |
| 이미지 생성 날짜 확인  | `docker image inspect <이미지> --format='{{ .Created }}'` |
| 이미지 크기 확인       | `docker image inspect <이미지> --format='{{ .Size }}'` |
| 운영 체제 확인         | `docker image inspect <이미지> --format='{{ .Os }}'` |
| 기본 실행 명령 확인    | `docker image inspect <이미지> --format='{{ .Config.Cmd }}'` |
| 노출된 포트 확인      | `docker image inspect <이미지> --format='{{ .Config.ExposedPorts }}'` |
| 이미지 환경 변수 확인  | `docker image inspect <이미지> --format='{{ .Config.Env }}'` |
| 이미지의 모든 레이어 확인 | `docker image inspect <이미지> --format='{{ .RootFS.Layers }}'` |
# Docker 컨테이너와 이미지의 이름 및 태그 관리
## 컨테이너 이름 관리
docker run을 실행하면 컨테이너가 실행될 때 **자동으로 생성된 이름**을 가짐

실행 후 docker ps를 실행하면, 자동으로 생성된 **무작위 컨테이너 이름**을 볼 수 있음

`docker run -d --name goalsapp my-image` 를 통해 컨테이너의 이름을 지정할 수 있음

## 이미지 태그 관리
컨테이너와 마찬가지로 무작위의 태그가 생성됨 

`docker build -t <이미지이름>:<태그>` 로 태그를 생성할 수 있는데, 이는 버전 관리에 장점을 가짐
*이미지 이름은 `Repository`라고도 부름

`docker build -t goals:v1`, `docker build -t goals:v2 ` 로 여러 버전을 설정할 수 있음

 `docker run` 시 태그를 따로 설정해주지 않으면? -> 기본적으로 `latest`태그가 붙은 이미지가 실행됨
