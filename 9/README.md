## 1. 최종: Docker 컨테이너 배포하기
로컬 머신을 넘어 실제 리모트 머신에서도 컨테이너를 실행하는 것이 목표이며,   
사용자들이 웹에서 접근 가능한 형태로 프로덕션 환경을 구성할 것.

**=> 도커의 진짜 쓰임새를 체험하자!**

### 🚀 이번 시간에 다루게 될 시나리오
- 리모트 호스트에 컨테이너를 띄우는 법
- 배포할 때 고려해야 할 설정과 보안
- Docker Audit 설치 및 운영
- 수동 관리와 관리형 서비스의 차이
- 실전에서 마주치는 수많은 문제 해결 시나리오..

### ✅ 개발에서 프로덕션으로, 컨테이너의 진짜 여행
이미 컨테이너가 얼마나 유용한지 경험했다. 개발 환경을 빠르게 설정하고,    
격리된 상태에서 안정적으로 크드를 실행할 수 있었다.

하지만 어디까지나 **개발 환경**에서의 이야기였다.
이번 시간에 그 다음 단계인 **배포**로 넘어가려 한다.

### ✅ 컨테이너는 단순한 개발 도구가 아니다
도커 컨테이너는 코드 뿐만 아니라, 그 코드가 작동하기 위한 모든 환경을 함께 담고 있다.   
이 말은 곧, 도커가 설치되어있는 곳이라면 어디에서든 동일하게 실행될 수 있다는 뜻.

이게 왜 중요할까?

현실에서는 로컬 환경에서 잘 동작하던 앱이,   
리모트 서버에 올리면 이유 없이 작동하지 않는 일이 있기 때문이다.   

하지만 컨테이너를 사용하면, 로컬에서 잘 작동하던 그대로를   
리모트 서버로 옮겨도 똑같이 작동시킬 수 있다.

Node.js든, Python이든, PostgreSQL이든   
리모트 머신에 직접 설치할 필요가 없다.

컨테이너 내부에 모든 환경이 포함되어 있기 때문에   
배포시에 컨테이너만 있으면 해결이 된다.   

이건 단순히 "편하다"를 넘어   
예측 가능성, 재현성, 확장성을 보장해준다.

### ✅ 그런데, 개발과 배포는 조금 다르다!
개발할 때 자주 쓰던 바인트 마운트 - 파일을 실시간으로 수정하고, 바로 컨테이너에 반영되게 해주는 기능.   
하지만 프로덕션 환경에서는 바인드 마운트를 사용하면 안된다.   
보안, 안정성, 일관성 측면에서 문제가 될 수 있기 때문이다.   
왜 그런지, 그리고 어떻게 대체할 수 있을지 알아보자.

### ✅ 다중 컨테이너?
간단한 서비스는 하나의 컨테이너로도 충분하지만,   
실제 서비스를 운영하다 보면 DB, 백엔드, 프론트엔드, 캐시 등   
여러 개의 컨테이너로 구성된 구조가 필요해진다.

이런 다중 컨테이너 앱을 어떻게 관리하고 배포할지   
그리고 여러 리모트 호스트에 나눠 배포할 수 있는 지 알아보자.

### ✅ 완전한 통재 vs 적절한 위임
한가지 더 고민해볼만한 게 있다.
리모트 머신을 내가 관리할 것인지,   
아니면 AWS 같은 클라우드 서비스에 일부 통제권을 넘기고 관리의 부담을 줄일 것인지.

직접 관리하면 자유도는 높지만,   
보안, 성능, 가용성 등 모든 것을 직접 세팅해야한다.

반면, 관리형 서비스(AWS 등)는 제한은 적지만 책임도 적다.   
상황에 따라 이 방식이 현명할 수도 있다.   

이 상황을 판단할 수 있는 기준에 대해서도 알아보자.

### ✅ 이번 시간의 최종 목표는?
> “로컬에서 개발하던 컨테이너를, 리모트 환경에서도 문제없이 작동하도록 만드는 것.”
- 개발과 배포 환경의 차이를 이해하고
- 컨테이너 환경을 일관되게 유지하며
- 다중 컨테이너로 프로젝트를 구성하고
- 관리형 배포 전략에 대해 고려해보자.

=> 컨테이너가 단순한 개발 보조 도구가 아니라,
진짜 '서비스를 운영하기 위한 인프라의 핵심'임을 체감해보자."

## 2. AWS EC2에 기본 Node.js 앱 배포하기
일단 DB, 프론트를 고려하지 않고 단순한 Node.js 앱 하나로 시작하자.

### ✅ EC2란?
AWS의 EC2(Elastice Compute Cloud)는   
쉽게 말해, 클라우드 위에 띄우는 나만의 리눅스 컴퓨터이다.
- 인터넷만 있다면 어디서든 접근 가능하고,
- 내가 원하는 소프트웨어를 설치하고,
- 애플리케이션을 실행할 수 있다.
이 EC2 인스턴스에 Docker를 설치하고,
그 위에서 컨테이너를 실행하게 될 것.

### ✅ 배포 흐름 잡기
1. EC2 인스턴스 생성
  - 리모트 서버를 생성하는 것.
  - VPC와 Security Group을 함께 구성.
  - Security Group을 통해 어떤 포트를 외부에 노출할지 결정.
2. SSH로 EC2에 접속
  - Secure Shell: 터미널 기반으로 원격 컴퓨터에 접속할 수 있는 방식.
  - 직접 명령어를 입력하며 Docker를 설치하고 컨테이너를 실행.
3. 도커 컨테이너 실행
  - 앱을 빌드한 뒤,
  - Docker Hub에 이미지를 업로드하고
  - 리모트 서버에서 이미지를 내려받아 실행.

### ✅ 도커 이미지 빌드 & 실행
강의에 첨부된 파일을 다운 받고 이미지를 빌드 & 실행해보자.
1. 도커 이미지 빌드
```
docker build -t node-dep-example .
```
2. 도커 컨테이너 실행
```
docker run -d --rm -p 80:80 --name node-dep node-dep-example
```
3. localhost에 접속
`welcom.html`이 보이면 성공

### ✅ 개발 vs 배포 - 같은 컨테이너, 다른 방식
>  `docker run`명령어로 로컬에서 컨테이너를 실행했다.   
> 그런데, 이번에는 바인드 마운트가 포함되어있지 않았다.
> 
> 우리가 지금 실행한 앱이 **프로덕션 배포용**이기 때문!

개발중엔 바인드 마운트가 필요하다.
컨테이너를 매번 빌드하거나 재시작하는 것을 하지 않기 위헤   
`-v`를 사용해 로컬 프로젝트 폴더를 컨테이너 내부 디렉토리에 **실시간 연결** 해주는 방식이다.

즉, 로컬에서 파일을 수정하면   
컨테이너는 자동으로 그 최신 코드를 반영하게 되는 것.   
덕분에 이미지를 다시 빌드하거나 컨테이너를 재시작하지 않아도 됨.

하지만 프로덕션에서는 다르다.
- 리모트 서버엔 개발자의 코드 폴더(디렉토리)가 존재하지 않기 때문
- 바인드 마운트를 사용하면 결국 서버 자체를 구성해야한다.
- 그럼 컨테이너가 **자체 환경을 캡슐화 한다는 원칙에 위배됨!**

그렇다면 어떻게 해야하나? -> **COPY**를 사용합니다!
```
COPY . /app
```
Dockerfile에는 보통 이런 명령이 있다.   
이게 의미하는건, 소스 코드를 이미지에 그대로 복사한다는 것.

이렇게 하면, 이미지를 빌드한 시점의 코드와 실행 환경이   
모두 하나의 이미지 안에 포함되게 된다.   
그리고 이 이미지를 리모트 서버에서 가져오기만 하면   
주변에 아무 설정 없이도 그대로 실행할 수 있는 것.

**- 바인드 마운트 vs COPY**
| 개발 환경                                | 프로덕션 환경                                  |
|------------------------------------------|------------------------------------------------|
| `-v ./src:/app/src` (바인드 마운트) 사용 | `COPY . /app` (Dockerfile 내 COPY 사용)       |
| 실시간 코드 반영                         | 정적인 코드 포함                               |
| 빠른 피드백 / 테스트                    | 재현 가능하고 독립적인 실행 환경              |
| 이미지 재빌드 불필요                     | 이미지 빌드 후 배포 필요                       |
| 유연하지만 일시적                        | 견고하고 신뢰할 수 있음                        |

정리하면, 
컨테이너의 핵심은 **이식성**과 **재현성**이다.   
어디에서 실행하든 - 내 컴퓨터든, 팀원의 컴퓨터든, AWS 든,
항상 같은 결과를 내야한다.

그러기 위해서는,   
코드와 환경이 완전히 캡슐화된 하나의 이미지로 존재해야하고,   
그 안에 모든 것이 포함되어야 한다.

그것이 바로 도커의 철학이고,   
우리가 프로덕션에서 바인드 마운트를 사용하지 않는 이유다.

### ✅ 이제 진짜 웹에 올려보기
1. 코드를 수정하고
2. 이미지를 다시 빌드하고
3. 빌드된 이미지를 원격 서버에서 실행한다.

이제 까지는 이 과정을 모두 로컬에서 진행했지만,
이제는 이 이미지를 EC2 리모트 서버에서 실행할 것이다.

세팅 과정 정리
1. `Amazon Linux AMI 64-bit x86` 선택 (운영체제)
2. 인스턴스 타입 `t3.micro` -> RAM과 CPU가 제한적이지만 무료 버전이니 사용!
3. VPC(Virtual Private Cloud) 자동 생성되어 있다면 그대로 사용 -> 없다면 새로 생성하기!
  - 대부분의 설정은 기본값으로 유지해도 충분함.
4. Key Pair - SSH 연결을 위한 열쇠
  - 'Create new key pair'
  - `.pem` 파일은 절대 공유 X, 재다운로드 안됨(새로 인스턴스 생성해야됨)
5. 인스턴스 실행 및 확인
6. SSH 접속 - 로컬에서 서버에 연결하기
  - `.pem` 키 파일이 있는 폴더에서 터미널 실행
  - `chmod 400 example-1.pem`, `ssh -i "example-1.pem" ec2-user@<EC2 퍼블릭 IP 주소>`

다음은 **EC2에 Docker를 직접 설치**하는 단계이다.   
생성한 컨테이너 이미지를 이 서버에서 실행하기 위한 필수 준비 작업.

```amazon-linux-extras install docker``` 과거에는 해당 명령어를 사용했지만,   
Amazon Linux 2 또는 최신 버전에서는 동작하지 않는다고 한다.
[참조](https://stackoverflow.com/questions/53918841/how-to-install-docker-on-amazon-linux2/61708497#61708497)

```bash
# 1. 시스템 업데이트
sudo yum update -y

# 2. Docker 설치
sudo yum -y install docker

# 3. Docker 서비스 시작
sudo service docker start

# 4. 현재 사용자(ec2-user)를 docker 그룹에 추가
sudo usermod -a -G docker ec2-user
```
`usermod`명령어로 docker 그룹에 추가한 후에는   
현재 SSH세션에는 반영되지 않는다고 한다.   
따라서 로그아웃 했다가 다시 로그인해야 변경 사항이 적용된다.
```
sudo systemctl enable docker // 재접속할 때마다 Docker가 자동 실행 되게 설정
docker version
```
이후 아래 명령어를 입력하고, 버전 확인이 가능하면 웹에 node.js를 올릴 준비가 되었다.

이제 Docker가 설치된 EC2 인스턴스에   
우리가 만든 Node.js Docker 애플리케이션 이미지를 전달해야한다.   

방법은 크게 2가지가 있다.   

1. 리모트에서 빌드하는 방식
   - 소스 코드 전체를 EC2로 복사
   - 거기서 `docker build`로 이미지 생성
   - `docker run`으로 실행
     - 불필요하게 복잡하고, 특히 보안, 빌드 환경 문제로 비효율적임
2. 로컬에서 빌드 -> Docker Hub 푸시 -> EC2에서 pull
   - 로컬에서 이미지를 빌드하고
   - Docker Hub에 푸시
   - EC2에서 `docker pull` 후 `docker run`
     - 협업이나 CI/CD에도 이상적

첫 번째 방법은 불필요하게 복잡하고, 보안, 빌드 환경 문제로 비효율적이라고 한다.   
따라서 2번째 방법으로 차근차근 해보면,   

1. Docker Hub에 저장소 만들고
2. `dockerignore`파일을 설정해 `node_modules`, `Dockerfile`, `*.pem`파일을 추가해준다.
3. 이미지 빌드 및 태그 설정
   ```
   # 이미지 빌드
   docker build -t node-dep-example-1 .

   # Docker Hub용 태그 추가 (형식: 사용자명/저장소명)
   docker tag node-dep-example-1 academind/node-example-1
   ```
4. 이미지 푸시
   ```
   docker push solssak/node-docker-1
   ```
Docker Hub에 푸시가 되었다면 EC2 인스턴스에서 해당 이미지를 실행할 수 있다.   
```
docker pull solssak/node-example-1
docker run -d -p 80:80 solssak/node-example-1
```

이후 인바운드 HTTP를 허용하면 이렇게 하면 전 세게 어디에서나   
브라우저로 EC2 퍼블릭IP에 접속해 우리의 앱을 확인해볼 수 있다.   
1. EC2 콘솔 -> 인스턴스 -> 보안 그룹에서
2. 하단 인바운드 규칙 생성
   Type: HTTP
   Port: 80 (자동)
   Source: Anywhere(IPv4)
3. 저장
   
**🚨 주의할점**   
EC2에서 `pull`명령어를 입력했을 때 아래 에러가 발생할 수 있다.
> docker: no matching manifest for linux/amd64 in the manifest list entries.

EC2 인스턴스에서 사용하는 CPU 아키텍쳐(`amd64`)에 맞는 Docker 이미지가 Docker Hub 저장소에 존재하지 않는다는 뜻이다.   
로컬에서 `docker build`시에 M1같은 ARM 환경에서는 `linux/amd64`아키텍쳐로 이미지가 생성된다.   
그런데 EC2 인스턴스는 일반적으로 `amd64`기반이다.   
따라서 Docker Hub에 업로드된 이미지가 `arm64`전용일 경우에 EC2에서 `pull`할 수 없고, 저 오류가 발생한다.   

해결 방법으로는 1. `amd64` 이미지로 재빌드 2. 멀티 아키텍쳐 이미지로 빌드 하는 방법이 있는데,   
범용성 측면에서 2번이 유리하다고 생각했다.
```
docker buildx build --platform linux/amd64,linux/arm64 -t solssak/node-docker-1 . --push
```

### ✅ 코드 수정부터 인스턴스 종료까지
코드 변경 시에는 당연하게도 리모트 서버에 자동으로 반영되지 않는다.

1. 변경된 코드 반영: 새 이미지 빌드 & 푸시
   ```
   # 1. 현재 실행 중인 컨테이너 확인
   sudo docker ps

   # 2. 해당 컨테이너 중지
   sudo docker stop <container-id>
    ```
3. EC2에서 실행 중인 컨테이너를 중지하고 최신 이미지를 가져와(pull) 다시 실행하면 변경사항이 적용된다.
    ```
    # 1. 최신 이미지 가져오기
    sudo docker pull solssak/node-example-1

    # 2. 컨테이너 재실행
    sudo docker run -d --rm -p 80:80 solssak/node-example-1
    ```
=> 서버에 Node.js가 없어도 Docker 덕분에 앱을 실행할 수 있다.   
다만, 이미지 변경 -> 재배포라는 배포 흐름이 있는데, 자동화할 수는 없을까?

### ✅ 수동 배포 방식의 마무리와 한계
우리는 Docker로 만든 Node.js 앱을   
AWS EC2 인스턴스에 배포하는 과정을 겪어봤다.   
- EC2 인스턴스 생성
- Docker 설치
- 이미지 pull& run
- 웹에서 접속
이 모든 사이클을 Node.js나 기타 런타임이 없이도
Docker만으로 앱을 실행할 수 있다는 것을 보여준다.
-> 이것이 Docker의 가장 큰 장점이다.

하지만 이렇게 직접 구현하는 방식에는 한계가 있을 것이다.
그래서 필요한 것은 **관리형 서비스**이다.
- AWS Elastic Beanstalk
- AWS ECS(Fargate 포함)
- Vercel, Heroku, Fly.io 등
이런 서비스들은 **서버 생성, 보안 그룹 생성, OS 업데이트**를 모두 자동으로 관리해준다.

즉, 직접 구현 하는 방식은 학습 측면이나, 세세하게 설정할 수 있다는 점에서 잠점이 있지만,   
대부분의 경우에는 **자동화되고 관리되는 인프라가 더 안전하고 효율적이라고 할 수 있다.**

여전히 docker와 이미지 기반으로 작업을 진행하는데,   
`docker run`대신 AWS가 제공하는 도구(UI, CLI, CloudFormation 등)을 사용하는 것 뿐.
즉, 우리가 직접 컨테이너를 생성하지 않지만, 컨테이너 단위로 앱을 관리하는 개념은 그대로 유지

### ✅ 수동 배포 방식에서 관리형 컨테이너 서비스로 전환하기
**EC2 없이, 관리형 서비스인 ECS(Fargate 모드)를 활용해   **
로컬에서 만든 Docker 이미지를 AWS에서 실행해보자.

**ECS 특징**
- EC2 없음 - 서버 관리 필요 없음
- Docker 설치 필요 없음 - 자동으로 관리됨
- 서버리스 컨테이너 필요할 때만 실행됨
- GUI 기반 설정 - CLI 대비 쉽다는 장점이 있음.

**EC2 설정 흐름 정리**
1. 컨테이너 정의
   - 이름 : `your-docker-example`
   - 이미지: `your-dockerhub-id/solssak-docker-id`
   - 포트: 80
     > 이 설정은 `docker run -d -p 80:80 ...`과 같은 역할
2. 태스크 정의
   - 역할: 컨테이너 실행에 필요한 설정
   - 런타임: `Fargate`(서버리스)
   - CPU/메모리: 기본값 사용
     > 하나의 태스크에 여러 컨테이너도 가능하지만, 이번엔 단일 컨테이너로 진행
3. 서비스 정의
   - 태스크를 지속적으로 실행/유지관리하는 역할
   - 로드 밸런서는 이번엔 생략
4. 클러스터 생성
   - 여러 서비스/태스크를 묶는 논리적 그룹
   - 자동 생성을 선택
5. 실행 및 배포
   - 모든 설정 완료 후 `Create` 클릭
   - 생성 완료 후 `View Service` -> `Tasks`탭에서 태스크 확인
   - Task ID 클릭 -> Public IP 확인 가능
     > 이후 `http://<task-public-ip>`에서 실행 확인

** ✅ AWS ECS에서 Docker 컨테이너 업데이트 하는 방법**
1. 코드 수정 & 이미지 재빌드
   ```
   # 코드 수정 (예: welcome.html에서 '!!' → '!')
   docker build -t yourname/node-example-1 .

   # 태그 지정 (필요 시)
   docker tag yourname/node-example-1 yourname/node-example-1

   # Docker Hub에 푸시
   docker push yourname/node-example-1
   ```
2. ECS Task Definition 업데이트
   - AWS Console -> ECS -> Clusters -> default -> Task Definitions
   - 기존 태스크 선택 -> `Create new revision` 클릭
   - 설정은 이전과 동일하게 유지
   - `Save`
     > 이렇게 하면 ECS가 새로운 이미지 버전을 가져올 준비가 횜
4. 서비스에 새로운 Task Revision 반영
   - ECS 클러스터의 서비스로 이동
   - `Actoins` -> `Updtae Service`클릭
   - `Skip to review` -> `Update Service`
     > 이 단계에서 ECS는 새 Task Revision을 사용하여 컨테이너를 다시 실행하며, 최신 이미지를 pull 함.
3. 실행 확인
   - `Tasks` 탭에서 새로운 태스크 상태 확인
     - `Pending` -> `Running`으로 바뀌면 실행 완료
   - `Task ID` 클릭 -> `Public IP`확인 -> 브라우저에서 접속
     > 새 Task마다 새로운 IP가 할당되는 점을 인지

> AWS ECS는 자동 업데이트를 하지 않는다.
> 명시적으로 새 이미지 사용을 지시해야 안정적인 운영이 가능
> Public IP는 변하므로, 고정 IP/도메인을 위한 별도 설정이 필요하다.

> 자동 배포가 가능하게끔 하려면 여러 방법이 있다.
> GitHub Actions + AWS CLI
> ECR + Code Pipeline
> Amazon ECS Image Updater(오픈소스 도구)

### ✅ AWS ECS에 다중 컨테이너 애플리케이션 배포 (MongoDB + Node.js API)
AWS ECS에서는 Docker Network 사용이 불가하다.
- 로컬에서는 Docker Compose가 내부 네트워크를 구성해 `mongodb`라는 이름으로 접근이 가능던 반면
- ECS 컨테이너가 서로 다른 머신에 분산될 수 있어 이름 기반 연결이 불가하다
- 다만 같은 Task Definition에 속한 컨테이너끼리는 localhost로 통신이 가능함.

1. 따라서 코드 변경이 필요.
```
// 전
const mongoUrl = `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`;

// 후
const mongoUrl = `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}:27017/course-goals?authSource=admin`;
```
```
// 환경변수
// backend/backend.env(로컬 개발용)
MONGODB_USERNAME=admin
MONGODB_PASSWORD=pass123
MONGODB_URL=mongodb

// AWS ECS Task Definition 실행 시
MONGODB_USERNAME=admin
MONGODB_PASSWORD=pass123
MONGODB_URL=localhost
```
위와 같이 env를 분리해서 로컬에서는 그대로 작동하게,   
ECS에서는 `MONGODB_URL=mongodb://localhost:27017`를 전달하여 처리한다.

2. 이후 백엔드 이미지를 별도로 빌드하고, Docker Hub에 새롭게 푸시한다.
```
# 프로젝트 루트 위치에서
docker build ./backend -t goals-node

# Docker Hub에 public repo 생성 (예: academind/goals-node)

# 로컬 이미지에 태그 지정
docker tag goals-node academind/goals-node

# 푸시
docker push academind/goals-node
```

3. ECS의 기존 리소스를 삭제한다.
- ECS Console > Clusters > 기존 클러스터 삭제
- 관련 서비스 -> 삭제
- 삭제 코드 입력: `delete me`, 대기

이후 새 클러스터, 새 태스크, 새 서비스를 생성해 백엔드 컨테이너를 실행할 수 있다.

1. 클러스터 생성
   - 유형: "Networking only"
   - 이름: `goals-app`
   - VPC: 자동생성(`Create VPC` 체크)
2. Task Definition 생성
   - Launch type: Fargate
   - Task name: `goals`
   - 역할: `ecsTaskExecutionRole`
   - 메모리/CPU: 최소사양
3. 컨테이너 추가 - Backend(`goals-backend`)
   - 이미지: `your-dockerhub-id/goals-node`
   - 포트: 80
   - 명령어: node,app.js -> 개발에서는 `npm start`, 프로덕션에서는 `node`만 사용
   - 환경 변수: 별도 작성
4. Dockerfile 및 코드 변경 사항
   - app.js 내부 MongoDB 주소 변경
   - `Dockerfile`에 env지정(선택 사항)
     - 혹은 ECS에서 실행 시 환경 변수 전달로 충분
5. 마지막으로 도커 이미지 재빌드 & 푸시
   ```
   docker build ./backend -t goals-node
   docker tag goals-node your-dockerhub-id/goals-node
   docker push your-dockerhub-id/goals-node
   ```

몽고DB 컨테이너를 추가 등록하자!

1. Task Definition
   - Container name: mongodb
   - Image: mongo -> ECS에서는 여기서 지정한 이미지 이름을 Docker Hub에서 자동으로 찾음
   - 포트: 27017
   - 환경변수: 별도 작성
2. Service 생성
   - Task Definition 기반으로 Fargate 서비스 생성
   - Task 수: 1
   - 퍼블릭 IP 자동 할당
3. 로드 밸런서 설정
   - Application Load Balancer(ALB) 수동 생성
   - ECS 서비스 생성 중 ALB와 Target Group 연결
4. 결과 확인
   - 실행 중인 태스크의 Public IP 확인
   - `http://<Public0IP>/goals`접근 -> 빈 배열 반환 -> 배포 성공 확인
   - Postman으로 API 테스트
  
🚨 문제 상황
- ECS에 다중 컨테이너(Node.js + MongoDB) 앱 배포 완료
- ALB를 통해 접근 가능한 도메인을 설정했지만,
- 실제로는 서비스가 계속 중지/재시작, ALB 도메인도 접속 불가
- 이유는:
  - ALB의 Health Check 경로 오설정 (/ -> /goals 필요)
    > Health Check 실패 → ALB가 “이 서비스 죽었음” 판단 → ECS 서비스가 태스크를 재시작함
  - Security Group 누락 (로드밸런서 -> ECS 서비스 인바운드 허용 필요)
    > Security Group 문제 → ALB가 ECS에 요청을 보낼 수 없음 → 역시 “죽었다고 판단” → 재시작
    > AWS의 모든 인바운드/아웃바운드 트래픽은 Security Group 허용 규칙에 제어됨.
    > ECS Task에 연결된 Security Group에서 ALB로 오는 요청을 허용하지 않으면 요청이 차단되어
    > 통신 실패 -> Health Check 실패 -> ECS Task 종료/재시작 단계를 거치게 되는 것

해결방법을 정리하면

1. Health Check 경로 수정
  - EC2 → Target Groups → 해당 그룹 선택
  - Health checks → Edit
  - Path를 / → /goals로 변경
  - 이유: / 경로는 앱에서 존재하지 않으므로 404 반환 → 로드밸런서가 "서비스 비정상" 판단
2. 보안 그룹 수정
  - EC2 → Load Balancer 선택 → Security Groups 탭
  - 기존의 디폴트 보안 그룹 외에, ECS 서비스에 연결된 Security Group도 추가
  - 결과적으로, 로드밸런서가 ECS 서비스의 포트(80 등)에 정상적으로 접근 가능

위 방법을 거치고 나면, 서비스가 더 이상 중지/재시작 되지 않고 안정적으로 유지된다.
- ALB의 Health Check가 통과하기 때문에 ECS가 태스크를 계속 유지.
- ECS 태스크가 재시작되지 않으면, 퍼블릭 IP도 변경되지 않게 되고,   
  ALB의 DNS name을 고정된 엔드포인트로 사용할 수 있게 된다.   
  > http://<ALB-DNS-name>/goals 처럼, 고정된 URL로 앱에 접근할 수 있게 되는 것
- 유의할점은 ALB의 DNS name은 고정되지만, ECS Task의 퍼블릭 IP는 여전히 바뀔 수 있다.   
  > 따라서 DNS name을 통해 접근해야하며 필요하다면 커스텀 도메인을 연결해서 사용할 것.
