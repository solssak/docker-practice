이번주차 목표:
1. 네트워크를 사용해 다수의 컨테이너를 연결하기
2. 컨테이너가 서로 통신하게 하기
3. 컨테이너에서 실행 중인 애플리케이션을 로컬 호스트 머신에 연결하기
---

# 도커 컨테이너 내부에서 외부 API와 통신하기
- HTTP 요청을 다른 웹사이트나 웹 API로 전송
- 컨테이너 내부에서 외부 API로 요청 보내면 잘 작동
- 컨테이너는 별도 설정 필요 없이 도커화된 애플리케이션 내부에서 웹 API 및 웹 페이지와 통신 가능

# 호스트 머신에서 MongoDB와 통신하기
- 도커 컨테이너 내부에서 외부 API와 통신하는 것은 기본적으로 가능하다.
- 하지만 로컬 호스트 머신의 서비스(Mongo DB 등)와 통신하려면 host.docker.internal을 사용해야한다.
- host.docker.internal은 호스트 머신의 IP 주소로 변환되는 특수 도메인으로, 이를 통해 컨테이너 내부에서 호스트 머신의 데이터베이스 및 서버에 접근 가능.

# 컨테이너끼리 직접 통신하기
MongoDB 이미지 실행
```
docker stop favorites // 기실행된 컨테이너 정리
docker run --name mongodb -d mongo // 도커 허브에서 공식 MongoDB 이미지 실행하기
```
📌 방법 1. 컨테이너 IP주소를 사용한 연결(비효율적)
```
docker inspect mongodb // 위 명령을 통해 MongoDB 컨테이너의 IP 주소를 확인

// 확인한 IP주소를 Node.js 애플리케이션 코드에 직접 입력
mongoose.connect('mongodb://<컨테이너_IP>:27017/favoritesDB', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
```
⛔️ 컨테이너를 다시 실행할 때마다 IP 주소가 바뀌므로, 매번 수동으로 변경해야 한다는 문제가 있음

📌 방법 2. 도커 네트워크를 사용한 연결
```
docker network create my-network // 새로운 도커 네트워크 생성
docker run --name mongodb --network my-network -d mongo // MongoDB 컨테이너를 네트워크에 연결하여 실행
docker run --name favorites --network my-network -d -p 3000:3000 favorites-node // 	Node.js 애플리케이션도 같은 네트워크에서 실행
// Node.js 애플리케이션 코드에서 mongodb라는 컨테이너 이름을 사용하여 MongoDB에 연결
mongoose.connect('mongodb://mongodb:27017/favoritesDB', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
```
이후 Postman으로 API 요청 테스트 이후 컨테이너 간 통신이 정상적으로 동작하는지 확인

📌 정리
- 컨테이너 간 통신을 위해 IP 주소를 직접 사용하면 비효율적.
- 도커 네트워크를 생성하여 컨테이너 이름으로 통신하는 것이 더 효율적.
- docker network create <네트워크 이름>을 사용하면 컨테이너 간 쉽게 연결 가능.

# 도커 네트워크를 활용한 컨테이너 간 통신
도커 네트워크란?
- 컨테이너 간 통신을 허용하는 가상 네트워크.
- 동일한 네트워크에 속한 컨테이너들은 서로의 이름을 사용하여 접근 가능.
- 이를 통해 IP 주소를 직접 조회하지 않고도 컨테이너 간 통신이 가능.

```
// 실행중인 컨테이너 중지 및 실행
docker stop favorites mongodb
docker container prune -f

// 도커 네트워크 생성
docker network create favorites-net

// 네트워크가 정상적으로 생성되었는지 확인:
docker network ls

// MongoDB 컨테이너 실행 (네트워크 포함)
// MongoDB 컨테이너를 favorites-net 네트워크에 포함시켜 실행
docker run --name mongodb --network favorites-net -d mongo

// Node.js 애플리케이션에서 MongoDB 컨테이너에 연결
// 이전에는 host.docker.internal을 사용했지만, 이제 컨테이너 이름(mongodb)을 사용하여 자동으로 연결가능
mongoose.connect('mongodb://mongodb:27017/favoritesDB', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
// ✅ 더 이상 IP 주소를 직접 조회할 필요 없음!

// Node.js 애플리케이션 컨테이너 실행
docker run --name favorites --network favorites-net -d -p 3000:3000 favorites-node
// ✅ 이제 Node.js 애플리케이션과 MongoDB 컨테이너가 같은 네트워크에서 실행됨.
```
이후 docker ps 로 실행 중인 컨테이너를 확인해보면
- favorites (Node.js 컨테이너)
- mongodb (MongoDB 컨테이너)
✅ 두 컨테이너가 실행 중이고, 같은 네트워크(favorites-net)에 속해 있다는 것을 알 수 있음.

📌 정리
- 도커 네트워크를 생성하여 컨테이너 간 통신을 쉽게 설정할 수 있음
- 컨테이너 이름을 사용하여 MongoDB와 Node.js 애플리케이션을 자동 연결 가능
- 더 이상 IP 주소를 직접 찾을 필요가 없음
- Node.js 컨테이너에서 MongoDB 컨테이너로 데이터 저장 및 조회 가능

🚀 추가로 공부하면 좋을 것
- 도커 네트워크 설정을 더 쉽게 관리하기 위해 docker-compose를 공부하면 좋음
