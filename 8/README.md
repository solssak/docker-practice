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