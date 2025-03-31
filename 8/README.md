## Laravel & PHP 프로젝트 Docker로 세팅하기
오직 Docker만 사용해서 Laravel과 PHP를 세팅하고,
이전에 배운 Docker 개념들(이미지, 컨테이너, Compose, 유틸리티 컨테이너 등)을 실제 예제에 적용한다.

**중요한 포인트**
- Docker Compose의 새로운 활용법
- 여러 Dockerfile을 활용한 이미지 간 상호작용
- Lavavel/PHP 지식은 필요 없음

NodeJS vs Lavael & PHP (스택 선정 이유)
- NodeJS는 예제가 많아 익숙함
  -  서버 + 애플리케이션 코드가 모두 JS로 통합됨
- Laravel & PHP는 복잡한 개발 환경 설정이 필요해서 Docker의 장점을 보여주기 좋음
  - PHP만 설치해서는 작동하지 않음.
  - 별도의 웹 서버 (Nginx), PHP 인터프리터, 데이터베이스가 필요

### Docker가 필요한 이유
- Laravel & PHP 환경은 로컬 설치가 복잡함
- Docker를 사용하면 로컬 설치 없이 완전한 환경 구성 가능
- 도커 외에는 호스트 머신에 아무것도 설치할 필요가 없음

**Laravel 소스 코드를 에디터에서 열 수 있도록 그리고 총 6개의 컨테이너를 구성하는 것이 목표**
컨테이너 항목
1. Laravel 코드 해석 및 실행을 위한 **PHP 인터프리터**
2. 클라이언트 요청을 PHP로 전달해줄 **Nginx 웹서버**
3. Laravel 앱과 연결된 **MySQL DB**
(유틸리티 컨테이너 3개)
4. Composer
  - PHP용 패키지 관리자(npm과 유사)
  - Laravel 프로젝트 생성 및 의존성 설치
5. Artisan
  - Laravel CLI 도구(마이그레이션, 시드 등)
6. npm
  - Laravel 뷰에서 사용하는 JS빌드를 위해 사용

### 최종 목표
- 이 모든 컨테이너를 Docker Compose로 구성
- Laravel 예시를 통해 복잡한 설정을 Docker로 간단히 해결하는 방법을 익히기

## Docker Compose로 Laravel 개발환경 구축하기 - nginx
### docker-compose.yaml 구성하기
1. `server` – nginx 웹 서버
2. `php` – PHP/Laravel 실행 컨테이너
3. `mysql` – 데이터베이스
4. `composer` – PHP 패키지 관리자
5. `artisan` – Laravel CLI 도구
6. `npm` – 프론트엔드 빌드 도구

1단계: nginx 서버 컨테이너 구성
```
version: "3.8"

services:
  server:
    image: "nginx:stable-alpine"
    ports:
      - "8000:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
```
- `image`: 공식 nginx 이미지 사용(경량화된 `stable-alpine` 태그 사용)
- `ports`: 호스트 8000 -> 컨테이너 내부 80포트에 바인딩
- `volumes`: nignx 설정 파일을 바인딩해서 **커스터마이징된 nginx.conf사용하기**

**왜 nginx.conf 파일을 구성하나요?**
기본 nginx 설정은 Laravel을 지원하지 않기 때문.
HTML 정적 파일을 서빙할 순 있어도, PHP 파일을 실행할 수 있는 로직은 전혀 들어 있지 않음.

Laravel은 PHP 프레임워크이고, 모든 요청을 `index.php`를 통해 처리하도록 설계되어 있기 때문에,
이를 가능하게 해주는 설정을 nginx에 직접 추가해야함.

즉, nginx.conf 파일과 함께 nginx 컨테이너는 Laravel 앱의 진입점 역할을 하며, 나중에 PHP 컨테이너와 연결될 예정

## Docker Compose로 Laravel 개발환경 구축하기 - PHP
이전에 만든 `nginx`컨테이너가 클라이언트의 요청을 받는데,
`nginx`는 `PHP`코드를 직접 실행할 수 없음.
따라서 `PHP`해석을 담당하는 컨테이너가 필요함.

**Dockerfile 작성하기**
공식 `PHP` 도커 이미지에는 `pdo`, `pdo_mysql`이라는 게 빠져 있어서,
커스텀 Dockerfile을 사용해 직접 이미지를 만들어야함.
```
// Dockerfile.
FROM php:7.4-fpm-alpine // Laravel과 호환성이 좋은 PHP + FPM 이미지

RUN docker-php-ext-install pdo pdo_mysql // 공식 이미지에 누락된 pdo, pod_mysql

WORKDIR /var/www/html
```

이후 docker-compose에 php 서비스를 등록해야함.
```
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated // delegated(성능 최적화 옵션): 동기화를 나중에 해서 속도를 빠르게 함.
```

커스텀 Dockerfile과 docker-compose를 통해 `Laravel` 코드가 `php` 컨테이너 안에서 코드를 읽고 실행할 수 있게 된다.

**Nginx와 PHP의 연결**
이제 nginx가 php 컨테이너에 요청을 넘겨야하는데,
Docker Compose에서는 같은 네트워크 안에 서비스들은 '이름'으로 통신할 수 있다.

nginx는 `php:9000`로 요청을 보낸다.
```
// nginx.conf
fastcgi_pass php:9000;
```
**유의할점**
- nginx가 말하는 `php`는 docker-compose에서 설정한 서비스 이름과 정확히 일치해야함.
- 포트번호 9000은 PHP FPM이 내부적으로 사용하는 기본 포트
- 포트를 외부로 노출할 필요는 없음.(nginx가 내부 네트워크에서 바로 연결함.)

**정리**
- `PHP`가 단독으로 작동하는 게 아니라, `Laravel`이 필요로 하는 환경을 완전히 갖춘 컨테이너를 구성함.
- `Nginx`는 자신이 처리하지 못하는 `PHP` 요청을 자연스럽게 `PHP` 컨테이너로 넘기고
- `PHP`는 그 요청을 해석해 응답을 반환할 수 있게 됨.
- 그리고 `Laravel` 앱을 `src/` 폴더에 작성하면서, 컨테이너 내부에서도 바로 적용될 수 있도록 연결까지 마침

일반 로컬 환경에서는 Laravel과 php를 연결하기 어렵지 않은데, 도커 환경 내에서 세팅하려니 약간 복잡한 느낌!
Laravel이 돌아가려면 PHP 확장, 웹서버, DB, 패키지 매니저가 필요하므로,
이를 Docker로 쪼개서 연결한 환경을 구성하는 과정이다!

## Docker Compose로 Laravel 개발환경 구축하기 - mysql
지금까지 구성한 것은
1. `nginx` 컨테이너: 요청을 받고
2. `PHP` 컨테이너: `Laravel` 코드를 실행하고
3. `src/` 폴더: 실제 `Laravel` 소스 코드가 저장되는 곳

이제, DB 컨테이너를 구성할 차례
MySQL 컨테이너는 docker-compose.yaml에 추가함.
```
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
```
`.env` 설정
```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```
Laravel에서 DB에 연결하려면 DB_HOST를 설정해줘야하는데, `mysql`이라는 이름으로 통신이 가능함.
-> 같은 docker-compose.yaml에서 정의된 컨테이너는 이름으로 통신할 수 있기 때문! (복습)

## Docker Compose로 Laravel 개발환경 구축하기 - Composer 컨테이너로 앱 생성하기 (유틸리티 컨테이너 활용하기)
- `Laravel`은 `composer create-projcet` 명령을 통해 설치함.
- 로컬에 composer를 설치하지 않고, Docker 컨테이너 안에서 composer 명령을 실행하려는 것.

```
# ./dockerfiles/composer.dockerfile

FROM composer:latest

WORKDIR /var/www/html

ENTRYPOINT ["composer", "--ignore-platform-reqs"]
```
- `--ignore-platform-reqs`: 호스트 OS나 PHP 버전이 다르더라도 오류 없이 실행
- ENTRYPOINT 덕분에 `docker compose run composer create-project ...` 처럼 깔끔하게 명령 실행 가능

이후 아래 명령어를 통해 앱 생성 가능!
```
docker compose run --rm composer create-project laravel/laravel .
```
`/src`디렉토리에 생성된 Laravel 프로젝트 확인 가능

## Docker Compose로 Laravel 개발환경 구축하기 - Composer로 Laravel 프로젝트 실행하기(생성)
이전 강의에서 유틸리티 컨테이너를 활용해서 Laravel의 환경을 구성했다면,
이제는 실행할 단계.

공식문서에서 설치 명령어를 복사한 후, Docker를 이용해 Composer 컨테이너에서 실행하기
```
docker compose run --rm composer create-project laravel/laravel .
```
이후 컨테이너 내부 `/var/www/html`에 Laravel 코드가 생성되고
바인드 마운트 방식으로 로컬 `src/`폴더에 반영될 것.
`src/`안에 Laravel 리렉토리 구조가 보이면 성공!

`http://localhost:8000`에 접속해서 페이지가 보이는지 확인하기

지금까지의 폴더구조
```bash
project-root/
│
├── docker-compose.yaml
├── dockerfiles/
│   ├── php.dockerfile
│   ├── composer.dockerfile
├── nginx/
│   └── nginx.conf
├── env/
│   └── mysql.env
└── src/
    └── (Laravel 프로젝트 전체가 여기에 생성됨)
```
