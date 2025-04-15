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
**EC2 없이, 관리형 서비스인 ECS(Fargate 모드)를 활용해**   
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
  
### ✅ ECS에서 EFS 볼륨을 활용해서 데이터 유지하기
현재 배포한 방식으로는 서비스 업데이트(=컨테이너 재시작)시 기존 데이터가 사라진다.   
MongoDB 컨테이너가 종료되면 내부 데이터가 같이 소멸되기 때문이다.

그래서 MongoDB 데이터가 ECS 서비스 재배포시에도 유지되도록 설정하는 것이 목표

**AWS EFS(Elastic File Sytem)과 마운트 설정**
1. EFS 파일 시스템 생성
   - EFS 콘솔 → Create file system
   - 동일한 VPC 선택 (ECS에서 사용 중인 VPC)
   - 보안 그룹은 ECS 컨테이너의 SG에서 NFS 포트(2049)를 허용해야 함
   - 이를 위해 별도 efs-sg를 생성하고, Inbound에 ECS SG 추가
2. ECS Task Definition 수정
   - Create new revision → 기존 정의 기반으로 새 개정판 작성
   - Volumes 섹션에서 EFS 유형의 볼륨 추가 (data 이름 추천)
   - mongodb 컨테이너의 /data/db에 마운트 (MongoDB의 기본 저장 경로)
3. 서비스 업데이트
   - Update Service → Force new deployment
   - Platform version은 1.4.0 사용 (EFS 지원을 위해)

🚨 주의 사항
ECS는 롤링 배포 방식을 사용한다. (기존 태스크 유지 + 새 태스크 병행 실행)   
MongoDB는 동시에 하나의 파일 시스템에만 접근 가능해서   
동시에 두 MongoDB 컨테이너가 /data/db 접근 시 충돌이 발생한다. (mongod.lock 에러)   

그래서 새 태스크 배포 전, 기존 태스크를 수동으로 중지하여 충돌 방지하는 동작이 필요하다.

**아키텍처 요약**
1. AWS ECS 사용
   - Fargate 런타임 기반, EC2 인스턴스 직접 관리 없음
   - 하나의 Task에 두 개의 컨테이너 포함
     - `Node.js REST API`
     - `MongoDB`
2. 데이터 영속성 보장
   - MongoDB 컨테이너에 EFS 볼륨 마운트
   - AWS EFS
   - 컨테이너 재시작/재배포 시에도 데이터 유지
     - EFS는 AWS가 관리하는 네트워크 스토리지므로, 컨테이너와 무관하게 유지
3. 외부 접근
   - Application Load Balancer (ALB) 사용
     - 고정된 DNS 이름 제공
     - 들어오는 요청을 Node.js API 컨테이너로 라우팅
   **- 유저 -> ALB -> Node.js 컨테이너 -> MongoDB 컨테이너 -> 데이터 처리 및 응답 변환**

아키텍처 시각화
```
사용자 요청 (HTTP)
        │
        ▼
Application Load Balancer (고정 DNS)
        │
        ▼
ECS Service (Fargate)
        │
        └──────────┬────────────┐
                   ▼            ▼
      Node.js Backend     MongoDB (DB 컨테이너)
             │                     │
             └──────────┐          │
                        ▼          ▼
                  Amazon EFS (Persistent Volume)
```

### ✅ MongoDB를 ECS에 직접 배포하는 대신 MongoDB Atlas를 사용하기
MongoDB Atlas를 사용하면 직접 컨테이너나 인스턴스에 MongoDB를 설치할 필요가 없다.
> MongoDB Atlas는 MongoDB 공식에서 제공하는 클라우드 기반 관리형 데이터베이스 서비스이다.
- 주요 기능:
  - 자동 확장, 백업, 복구, 보안 패치 등 관리 자동화
  - 글로벌 클러스터링 및 고가용성 제공
  - 쉬운 연결 (MongoDB URI 제공)
- 사용자는 단순히 애플리케이션에서 연결 URI만 설정하면 됨

즉, **인프라를 직접 제어하고 싶은 경우**에는 ECS + MongoDB 컨테이너   
빠르게 서비스를 구축하고 싶고, **MongoDB를 깊이 모르거나 운영 리소스가 부족한 경우** MongoDB Atlas

**MongoDB -> MongoDB Atlas 전환 흐름**
1. MongoDB Altas 클러스터 생성
   - 사용자, 비밀번호 설정
   - IP 접근 제어
3. 환경 변수 수정
   - `MONGODB_URL` -> Atlas에서 제공하는 연결 URI로 변경
   - `MONGODB_NAME` 등으로 DB 이름 유연하게 설정
   - 개발/프로덕션 분기처리는 환경변수로 통제
5. 개발 환경(docker-compose.yaml)정리
   - MongoDB 서비스 제거 (이제 Atlas 사용)
   - 볼륨, depends_on 등 관련 설정 제거
   - `.env`파일 수정(Atlas 자격 정보 반영)
7. 연결 오류 발생 시 확인 사항
   - Atlas의 IP 화이트리스트 설정
   - 사용자 권한 및 비밀번호 확인
9. 테스트
    - `docker-compose up`
    - `/goals`요청 시 빈 배열 반환 -> DB 연결 성공

**✅ ECS에서 MongoDB Atlas 전환하기**

1. 기존 설정 정리
   - ECS 태스크 정의에서 `mongodb`컨테이너 제거
   - 연결되어 있던 `EFS 볼륨`, `보안 그룹`, `파일 시스템` 정리
2. 환경 변수 구성 변경
3. Docker 이미지 갱신
   - 코드에서 Atlas용 환경 변수를 반영했으면, -> 반드시 이미지를 새로 빌드하고 Docker Hub에 푸시
   - 그렇지 않으면 ECS는 여전히 이전 이미지로 컨테이너 실행
4. ECS 서비스 업데이트
   - `Force new deployment`로 새 Task 생성
   - 이제 MongoDB 컨테이너 없이, 외부 MongoDB Atlas에 연결

**MongoDB -> MongoDB Atlas로 전환 후 얻을 수 있는 장점**

- 데이터 영속성 확보: 컨테이너 재시작해도 데이터 유지
- 운영 비용/시간 절감: 백업, 보안, 복제 등을 Atlas가 관리
- 개발 환경과 일치: Atlas를 dev/prod에서 모두 사용 가능

### ✅ 프론트엔드(React SPA)를 추가하여 완전한 다중 컨테이너 애플리케이션 구성
이전까지 한 일
- 기존에는 Node.js API + MongoDB 컨테이너 → ECS에 배포
- 이후 MongoDB 컨테이너를 제거하고 MongoDB Atlas로 전환
- 현재는 Node.js 백엔드 API 하나만 ECS에서 실행 중

새로운 목표
- 프론트엔드(React SPA)를 추가하여 완전한 다중 컨테이너 애플리케이션 구성

핵심 흐름
1. React SPA는 빌드 단계를 필요로 함
   - 개발 중에는 npm start 또는 vite/webpack dev server로 로컬 개발
   - 배포 시에는 반드시 npm run build → 정적 파일 생성 필요
   **이 빌드 결과물을 서빙할 환경(Nginx 등)**이 필요함
2. SPA는 SSR이 아님 → 정적 파일 서빙이 필요
   - Node.js API처럼 서버에서 실행되는 게 아님
   - 단순 정적 파일을 클라이언트로 전달하는 구조
   따라서 별도의 서버(Nginx) 또는 Node.js Express static 미들웨어 등을 사용하여 서빙해야 함

**✅ 프론트엔드(React) 앱의 "빌드 단계" 필요성과 도커 배포 전략
React, Vue, Angular 등의 프론트엔드 앱 특징   
브라우저에서 직접 실행할 수 없는 JSX, TypeScript, 최신 JS 문법 등을 사용   

→ 이 코드들은 브라우저 친화적 코드로 '변환'(컴파일) 되어야 함

**현재 React 프로젝트 구조**

npm start: 개발용. 내부적으로 react-scripts start 실행 → 개발 서버 실행   
npm run build: 프로덕션용. 내부적으로 react-scripts build 실행 → /build 폴더에 최적화된 정적 자산 생성

**🚨 문제점**

개발용 Dockerfile은 npm start 기반 → 프로덕션에서 쓸 수 없음   
npm run build는 정적 파일을 내보내기만 하고, 서버는 실행하지 않음   

따라서, 프로덕션에서는:   
정적 파일을 제공할 웹 서버 (예: Nginx) 가 필요   
또는 Express 같은 Node 서버를 직접 구성할 수도 있음   

**해결 방향**

새로운 Dockerfile 구성   
1단계: 빌드 컨테이너에서 npm run build 실행 → /build 정적 자산 생성   
2단계: Nginx 컨테이너를 이용해 /build 정적 자산을 서빙   
도커 멀티스테이지 빌드(Multi-stage Build) 활용 (추천)   

**✅ React 앱의 프로덕션 컨테이너 만들기 (빌드 & 멀티 스테이지 빌드)

1. Dockerfile.prod 생성
   - 기존 개발용 Dockerfile과 구분
   - npm run build 실행만 포함됨 (정적 파일 생성 목적)
   - 하지만 이 파일만으로는 작동 안 함 – 서버가 없음
2. 멀티 스테이지 빌드(Multi-stage Build) 활용
   1단계: Node 환경에서 npm run build → /build 정적 자산 생성   
   2단계: Nginx 환경에서 /build 결과물을 /usr/share/nginx/html에 복사   
   - 최종 이미지는 Nginx 기반의 가볍고 최적화된 정적 웹 서버

**정리하면..**

React 앱은 개발과 프로덕션이 반드시 다르게 실행되어야 함   
프로덕션에선 npm run build로 생성된 정적 자산 + 이를 서빙할 웹 서버(Nginx 등)가 필요   
이를 위한 최선의 Docker 전략은 멀티 스테이지 빌드   
👉 이 구조로 Dockerfile.prod을 완성하고, ECS에 배포할 예정   

**✅ React 애플리케이션을 위한 멀티 스테이지 Dockerfile**

멀티 스테이지 빌드를 활용
1. build 스테이지 (Node 환경)
   - Node 이미지에서 npm install, npm run build 실행
   - /app/build 경로에 최적화된 정적 파일 생성
   - 목적: 최종 결과물(build 폴더)만 생성
2. 웹 서버 스테이지 (Nginx 환경)
   - nginx:stable-alpine 이미지를 사용하여
   - 위에서 생성한 /app/build → /usr/share/nginx/html 로 복사
   - 목적: Nginx 서버로 최종 파일을 정적으로 서빙

**멀티 스테이지 빌드로 해결하는 것들**
1. (문제) 불필요한 파일, 툴, 용량 포함 문제
   - 일반적으로 빌드에 사용되는 의존성(devDependencies)이나 툴(node, npm, webpack 등)은 실행 시 필요 없음
   - 하지만 단일 Dockerfile로 빌드하면 최종 이미지에 다 들어감 → 용량 커짐 + 보안 리스크 증가
2. (해결) 빌드는 첫 번째 스테이지에서만 하고, 최종 결과물만 두 번째 스테이지에 복사
   → 경량화된 안전한 최종 이미지 생성

1. (문제) 보안 문제
   - 빌드 도구(Node.js 등)가 남아 있으면 공격 벡터가 늘어남
   - 애플리케이션 런타임에 필요 없는 패키지가 컨테이너에 존재하게 됨
2. (해결) 빌드 도구를 아예 포함하지 않음 → 최소한의 실행 환경 구성

1. (장점) 클린 빌드 이미지
결과물만 있는 상태에서 컨테이너를 실행하므로   
불필요한 캐시, 소스 코드, 임시 파일이 없다.

이미지 용량 ↓, 보안 ↑, 관리 편의성 ↑

**✅ 멀티 스테이지 빌드 기반의 React 프론트엔드 앱 ECS 배포 준비**

1. React 앱의 빌드 환경 고려
   - React 앱은 브라우저에서 실행되기 때문에, 내부 HTTP 요청에서 localhost 미사용   
   - 개발 중엔 localhost가 맞지만, 프로덕션에선 브라우저가 실행되는 사용자의 컴퓨터를 가리킴   
해결: 요청 URL을 /goals처럼 상대 경로로 변경 → 같은 도메인 내 백엔드 API와 통신 가능   
상황에 따라 REACT_APP_BACKEND_URL 같은 환경변수 도입 가능 (강의 후반에 나올 예정)
2. Dockerfile.prod에 멀티 스테이지 빌드 적용
   - 첫 번째 스테이지: node 기반으로 npm run build 실행
   - 두 번째 스테이지: nginx 기반으로 빌드된 정적 파일을 /usr/share/nginx/html에 복사 → 웹 서버로 제공
3. Docker 이미지 빌드 & 푸시
   ```
   docker build -f frontend/Dockerfile.prod -t academind/goals-react ./frontend
   docker push academind/goals-reac
   ```
   - 주의: -f 옵션에는 Dockerfile의 전체 경로, 마지막 인자는 컨텍스트 디렉토리 (보통 소스 폴더)

**✅ AWS ECS에 React 프론트엔드 배포 정리**
1. 두 개의 웹 서버 → 하나의 태스크에 배포 불가
   - 백엔드(Node)와 프론트엔드(React)는 각자 포트 80을 사용
   - 하나의 ECS 태스크 내에서는 같은 포트를 두 컨테이너가 공유할 수 없음
   - 따라서 각각 별도의 태스크로 분리 필요
2. React 앱에서 API 요청 경로 문제
   - React 앱은 브라우저에서 실행되므로 localhost는 ECS가 아닌 사용자 PC를 가리킴
   - 따라서 정적 도메인을 통한 요청 필요
   - ```
     const backendUrl = process.env.NODE_ENV === 'development'
     ? 'http://localhost'
     : 'http://<BACKEND-LOAD-BALANCER-DNS>';

     fetch(`${backendUrl}/goals`);
     ```
3. React 앱에 환경변수 전달은 빌드 시점에만 가능
   - 브라우저는 Docker 환경변수를 모름
   - 빌드 시 REACT_APP_ prefix를 붙여 환경 변수 주입
   - ```
     REACT_APP_BACKEND_URL=http://my-api.com npm run build
     ```
4. 프론트엔드를 위한 로드 밸런서도 별도 생성
   - 두 개의 ECS 태스크 → 두 개의 ALB 필요
   - 각각의 ALB는 다른 도메인 (또는 서브도메인)에 바인딩 가능
5. React 이미지 다시 빌드 & 푸시
   - API 요청 도메인 변경 → 코드 변경 → 반드시 다시 빌드 & 푸시
   ```
   docker build -f frontend/Dockerfile.prod -t academind/goals-react ./frontend
   docker push academind/goals-react
   ```
6. 프론트엔드용 ECS 서비스 구성
   - 기존과 동일한 방식으로 ECS 서비스 생성
   - Fargate, ALB, 서브넷, Security Group 설정 동일
   - 새로 만든 React 태스크 정의 기반

**이렇게 하면..**

React 프론트엔드 + Node 백엔드 각각 독립된 ECS 서비스 & ALB로 배포 완료   
서로 다른 URL에서 정상 작동 + 연동 성공   
프론트엔드는 정적 파일, 백엔드는 REST API 제공

**✅ 정리 요약**
1. 로컬 개발 환경(docker-compose up)도 여전히 잘 작동
   - React 앱이 localhost:3000에서 정상 구동됨
   - 개발 DB와 연결되어 있으므로 목표 데이터는 별도로 존
   - 로컬에서도 도커 기반으로 실행되므로 여전히 일관된 개발 환경을 유지 중
2. 개발 vs 프로덕션 환경의 차이
   - Docker가 환경 통일을 돕는 도구라는 점은 여전히 유효
   - 하지만 프로젝트 성격에 따라 환경별 차이는 불가피:
       - 예: React는 개발용(start)과 배포용(build) 실행 방식이 다름
       - MongoDB는 개발용 데이터베이스와 프로덕션용 Atlas를 따로 둬야 안전
3. 멀티 스테이지 빌드로 빌드 + 실행 환경을 분리
   - 하나의 Dockerfile에서:
       - React 소스 빌드 (Node 필요)
       - 빌드된 파일을 Nginx로 서빙
   - 이를 통해 하나의 Dockerfile로 운영용 이미지 생성 가능
   - 따라서 Dockerfile.dev, Dockerfile.prod를 나누지 않아도 됨
4. "동일한 환경"의 의미
   - 동일한 환경이라는 건:
       - 언어, 프레임워크, 실행 방식이 같은 코드베이스에서
       - 환경변수나 대상 외부 서비스만 다른 상황을 의미
   - React 앱은 브라우저에서 실행되므로 도커 환경변수는 의미 없음
   - 대신 빌드시 .env.production 등의 환경 파일을 통해 URL 차이만 조정
5. 코드가 바뀌어도 도커의 장점은 유효함
   - Node 버전만 통일하면, Docker로 빌드된 실행 환경은 같음
   - 도커의 장점:
       - OS 환경 차이 제거
       - 실행 방식 및 의존성 고정
       - CI/CD 용이성 확보
       - 로컬/서버 재현성 보장

**결론**

>도커를 활용해 React + Node + MongoDB 앱을 로컬과 AWS ECS 양쪽에서 동일하게 실행 가능한 구조로 완성.   
>프로덕션은 멀티 스테이지 빌드와 Atlas, ECS Fargate 등 클라우드 친화적 스택으로 구성하고,   
>개발은 docker-compose 기반 빠르고 가벼운 환경으로 유지하며,   
>전체적으로 확장성과 안정성을 모두 잡는 방향으로 마무리했다.


**✅ 멀티 스테이지 빌드 돌아보기**
- 하나의 Dockerfile에서 여러 단계(build, test, deploy 등)를 정의할 수 있음
- FROM ... AS build 처럼 각 단계에 이름 지정
- 최종 단계에서 필요한 결과물만 가져와 가볍고 효율적인 이미지 생성

`--target` 옵션이란?
> 특정 스테이지만 빌드하고 싶을 때 사용하는 옵션
```
docker build -f frontend/Dockerfile.prod --target build -t my-build-image .
```
- build라는 스테이지까지만 빌드됨
- 최종 nginx 실행 환경은 포함되지 않음
- 결과적으로, 정적 파일만 포함된 이미지가 생성됨 (서버 없음)

용도
- 테스트 용도: 테스트만 정의된 스테이지를 대상으로 테스트 이미지 빌드할 수 있음
- 개별 단계 디버깅: 전체 빌드 과정 중 특정 스테이지만 검증 가능
- CI 파이프라인 최적화: 각 단계별로 캐시 활용, 조건부 실행 가능

>`--target`은 멀티 스테이지 Dockerfile을 유연하게 활용하게 해주는 고급 기능.   
>하지만 프로덕션에서는 보통 전체 빌드를 하고, `--target`은 개발/CI 목적으로 사용하는 것이 일반적입니다.

### ✅ 최종 정리
1. 로컬 개발
   - Dockerfile, docker-compose를 통해 개발 환경 표준화
   - 바인드 마운트로 라이브 코드 반영 가능
   - 개발 서버 (예: React npm start)는 프로덕션과 다름
2. 이미지 빌드 & 컨테이너 실행
   - 애플리케이션을 이미지로 패키징 → 컨테이너로 실행
   - 로컬, 원격, 클라우드 어디서든 동일하게 실행 가능
3. 멀티 스테이지 빌드
   - React 같은 빌드 과정이 필요한 앱을 위한 도커 최적화
   - build 단계 → 소스코드 컴파일
   - serve 단계 → 빌드 결과물만 복사하여 nginx로 서비스
   - `--target` 옵션으로 특정 스테이지만 선택 빌드 가능
4. ☁️ AWS ECS 배포
   - EC2 직접 세팅 vs. ECS(Fargate) 관리형 서비스 사용
   - 컨테이너를 태스크 정의 → 서비스로 배포
   - ALB(Application Load Balancer)로 고정된 URL 제공
   - Health Check 설정, 보안 그룹 설정 중요
   - EFS 볼륨 연결로 데이터 영속성 확보
5. 단일 vs. 다중 컨테이너 구성
   - 하나의 태스크 내 여러 컨테이너 배포 가능 (단, 포트 충돌 주의)
   - 프론트엔드와 백엔드가 모두 웹서버일 경우 → 태스크 분리 필요
   - 서로 다른 도메인 / ALB 연결
6. 관리형 서비스 도입 판단
   - MongoDB 컨테이너 직접 운영 ❌ (백업/가용성/보안 문제)
   - → MongoDB Atlas 같은 관리형 DB로 전환 권장
   - 관리형 서비스는 복잡성 줄이고 운영 안정성 증가
7. 도커의 진짜 가치
   - 개발 = 프로덕션과 동일한 환경
   - 로컬/리모트/클라우드 어디든 동일한 이미지 실행
   - 단순한 로컬 테스트 도구를 넘어 전체 배포 전략의 핵심
8. 우리가 얻은 것
   - Dockerfile, Docker Compose 사용법
   - 멀티 스테이지 빌드
   - AWS ECS (Fargate) + ALB를 통한 배포
   - EFS, MongoDB Atlas 통합
   - 개발/운영/배포 전체 흐름 경험
