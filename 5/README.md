# 🚀 강의 목표
1. DB(MongoDB) 도커화
2. 백엔드(Node.js) 도커화
3. 프론트엔드(React) 도커화
4. 네트워크 구성

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

# package.json, package-lock.json 복사
COPY package*.json ./

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

# 3. package.json 복사 후 의존성 설치
COPY package*.json ./
RUN npm install

# 4. 전체 애플리케이션 코드 복사
COPY . .

# 5. React 개발 서버 실행
EXPOSE 3000
CMD ["npm", "start"]
```

### 🔨 프론트 컨테이너 빌드 및 실행
```
docker build -t goals-react ./frontend
docker run --name goals-frontend --rm -it -p 3000:3000 goals-react
```

---

## 🔗 4. 도커 네트워크 및 컨테이너 구성하기
### 🌐 새로운 네트워크 생성
```
docker network create goals-net
```

### 🗄️ MongoDB 컨테이너 실행 (네트워크 연결)
```
docker run --name mongodb --network goals-net -d mongo
```

🚨 **MongoDB 접속 주소 변경:** `host.docker.internal` → `mongodb`

```
const MONGO_URL = "mongodb://mongodb:27017/goalsdb";
```

### 🔧 백엔드 컨테이너 실행
```
docker build -t goals-node ./backend
docker run --name goals-backend --network goals-net -d -p 80:80 goals-node
```

🚨 **프론트엔드의 API URL 변경:** `http://goals-backend` → `http://localhost`
```
const API_URL = "http://localhost";
```

### 🎨 프론트엔드 컨테이너 실행
```
docker build -t goals-react ./frontend
docker run --name goals-frontend --network goals-net -it -p 3000:3000 goals-react
```

---

## 💾 볼륨을 사용해 MongoDB 데이터 지속하기
```
docker run --name mongodb --network goals-net -v data:/data/db -d mongo
```

🚨 **보안 설정 추가:** 환경 변수로 계정 및 비밀번호 설정
```
docker run --name mongodb --network goals-net -v data:/data/db -d \
  -e MONGO_INITDB_ROOT_USERNAME=max \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo
```

### 🔒 백엔드에서 인증 추가
```
const MONGO_URL = "mongodb://max:secret@mongodb:27017/goalsdb?authSource=admin";
```

### 🔄 백엔드 컨테이너 재실행
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

## 📜 백엔드 컨테이너 로그 유지 및 실시간 코드 업데이트
🚨 **로그 데이터 유지**
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
```json
"devDependencies": {
  "nodemon": "2.0.4"
}
```
```json
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

## 🎨 프론트엔드 컨테이너 개선
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