# 🚀 강의 목표
1. **MongoDB 도커화**
2. **백엔드(Node.js) 도커화**
3. **프론트엔드(React) 도커화**
4. **도커 네트워크를 활용한 컨테이너 연결**
5. **데이터 지속성을 위한 볼륨 적용**
6. **실시간 코드 변경 적용 (Live Reload)**

---

## 1. DB(MongoDB) 도커화
### MongoDB 컨테이너 실행
```
docker run --name mongodb --rm -d -p 27017:27017 mongo
```

### ▶백엔드 디렉토리에서 `app.js` 실행
```
node app.js
```

---

## 2. 백엔드(Node.js) 도커화
### Dockerfile 작성
```dockerfile
# Node.js 공식 이미지 사용
FROM node:latest

# 작업 디렉토리 설정
WORKDIR /app

# package., package-lock. 복사
COPY package*. ./

# 의존성 설치
RUN npm install

# 나머지 소스 코드 복사
COPY . .

# 애플리케이션이 사용할 포트 지정
EXPOSE 80

# 컨테이너 실행 시 app.js 실행
CMD ["node", "app.js"]
```

### Docker 이미지 빌드 및 실행
```
docker image prune -a
docker build -t goal-node .
docker run --name goals-backend --rm goal-node
```

🚨 **주의:** 백엔드에서 `localhost` 대신 `host.docker.internal` 사용 필요.

```
const MONGO_URL = "mongodb://host.docker.internal:27017/goalsdb";
```

### 코드 수정 후 다시 빌드 및 실행
```
docker build -t goal-node .
docker run --name goals-backend --rm goal-node
```

---

## 3. 프론트엔드(React) 도커화
### `frontend/Dockerfile` 작성
```dockerfile
# 1. Node.js 이미지 사용
FROM node:latest

# 2. 작업 디렉토리 설정
WORKDIR /app

# 3. package. 복사 후 의존성 설치
COPY package*. ./
RUN npm install

# 4. 전체 애플리케이션 코드 복사
COPY . .

# 5. React 개발 서버 실행
EXPOSE 3000
CMD ["npm", "start"]
```

### 프론트 컨테이너 빌드 및 실행
```
docker build -t goals-react ./frontend
docker run --name goals-frontend --rm -it -p 3000:3000 goals-react
```
- -it: React 개발 서버가 종료되지 않도록 **인터랙티브 모드** 실행.
- -p 3000:3000: 컨테이너 내부 포트를 로컬 호스트의 3000번 포트에 매핑.

---

## 4. 도커 네트워크 및 컨테이너 구성하기
### 새로운 네트워크 생성
```
docker network create goals-net
```

### MongoDB 컨테이너 실행 (네트워크 연결)
```
docker run --name mongodb --network goals-net -d mongo
```

🚨 **MongoDB 접속 주소 변경:** `host.docker.internal` → `mongodb`
이 때, host.docker.internal → mongodb로 MongoDB 접속 주소 변경이 필요
```
const MONGO_URL = "mongodb://mongodb:27017/goalsdb";
```

### 백엔드 컨테이너 실행
```
docker build -t goals-node ./backend
docker run --name goals-backend --network goals-net -d -p 80:80 goals-node
```

🚨 **프론트엔드의 API URL 변경:** `http://goals-backend` → `http://localhost`
프론트엔드는 브라우저에서 실행되기에 백엔드를 컨테이너 이름이 아닌 localhost를 통해 접근하도록 수정
```
const API_URL = "http://localhost";
```

### 프론트엔드 컨테이너 실행
```
docker build -t goals-react ./frontend
docker run --name goals-frontend --network goals-net -it -p 3000:3000 goals-react
```
### 정리
| **컨테이너**       | **네트워크**  | **포트**        | **통신 방식** |
|------------------|-------------|--------------|--------------|
| **MongoDB**     | `goals-net` | ❌ (포트 노출 없음) | `goals-backend` 컨테이너가 `mongodb` 컨테이너에 직접 연결 |
| **백엔드(Node.js)** | `goals-net` | `80:80`        | **MongoDB와 네트워크로 연결, 프론트엔드와는 `localhost`로 통신** |
| **프론트엔드(React)** | ❌ (브라우저에서 실행) | `3000:3000`    | **브라우저에서 `http://localhost`를 통해 백엔드에 요청** |


- **도커 네트워크(goals-net)를 활용하여 컨테이너끼리 통신하도록 설정**.
- **MongoDB 컨테이너는 네트워크 내에서 접근할 수 있도록, 백엔드에서 mongodb 컨테이너 이름을 사용**.
- **프론트엔드는 브라우저에서 실행되므로 http://localhost를 사용하여 백엔드와 통신**.
- **백엔드는 포트 80을 노출하여 프론트엔드가 접근 가능하도록 설정**.

## 볼륨을 사용해 MongoDB 데이터 지속하기
컨테이너 내부에 저장된 데이터는 컨테이너가 삭제되면 함께 제거된다. 따라서 MongoDB 컨테이너를 중지하면, 데이터가 사라진다. 이때 볼륨을 사용하면 컨테이너가 중지되어도 데이터를지속할 수 있다.
```
docker run --name mongodb --network goals-net -v data:/data/db -d mongo
```
커맨드는 위와 같은데, -v data:/data/db → MongoDB 데이터 저장소를 data라는 볼륨에 매핑한다는 의미이다.

🚨 **보안 설정 추가:** 환경 변수로 계정 및 비밀번호 설정
보안은 언제나 중요한 문제이다. 현재 아무런 인증 없이 누구나 내 MongoDB에 접근이 가능하다. 이때는 환경 변수로 계정과 비밀번호를 설정해 인증을 활성화해주는 방법이 있다.
```
docker run --name mongodb --network goals-net -v data:/data/db -d \
  -e MONGO_INITDB_ROOT_USERNAME=max \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo
```

### 백엔드에서 인증 추가
그럼 이제 백엔드에서 인증을 해주어야한다. 기존 MONGO_URL에 파라미터 형식으로 max:secret@mongodb를 작성하면, 관리자 인증을 사용해 인증을 진행한다. 변경사항이 있으니 백엔드 이미지 재빌드 후 컨테이너를 실행하면 백엔드와 MongoDB는 안전하게 연결된다.

```
const MONGO_URL = "mongodb://max:secret@mongodb:27017/goalsdb?authSource=admin";
```

### 백엔드 컨테이너 재실행
```
docker stop goals-backend
docker build -t goals-node ./backend
docker run --name goals-backend --network goals-net \
  -v /absolute/path/to/backend:/app \
  -v logs:/app/logs \
  -v /app/node_modules \
  -e MONGODB_USERNAME=max \
  -e MONGODB_PASSWORD=secret \
  -d goals-node
```

---

## 백엔드 컨테이너 로그 유지 및 실시간 코드 업데이트
🚨 **로그 데이터 유지** (named volume)
```
docker run --name goals-backend --network goals-net -v logs:/app/logs -d goals-node
```

🚨 **코드 변경 사항 즉시 반영** (바인드 마운트)
```
docker run --name goals-backend --network goals-net \
  -v /absolute/path/to/backend:/app \
  -v logs:/app/logs \
  -v /app/node_modules \
  -d goals-node
```

🚨 **nodemon 설정** (자동 반영)
```
"devDependencies": {
  "nodemon": "2.0.4"
}
```
```
"scripts": {
  "start": "nodemon app.js"
}
```

🚨 **환경 변수 적용**
```
docker stop goals-backend
docker build -t goals-node ./backend
docker run --name goals-backend --network goals-net \
  -v /absolute/path/to/backend:/app \
  -v logs:/app/logs \
  -v /app/node_modules \
  -e MONGODB_USERNAME=max \
  -e MONGODB_PASSWORD=secret \
  -d goals-node
```

---

## 프론트엔드 컨테이너 개선
🚨 **실시간 코드 업데이트** (바인드 마운트 적용)
```
docker run --name goals-frontend --network goals-net \
  -v /absolute/path/to/frontend/src:/app/src \
  -p 3000:3000 -it goals-react
```

🚨 **불필요한 파일 복사 방지** (`.dockerignore` 사용)
```
node_modules
.git
Dockerfile
```
