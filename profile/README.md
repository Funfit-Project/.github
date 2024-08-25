# 펀핏(Funfit) 프로젝트

## 0. 목차
https://github.com/Funfit-Project/.github/edit/main/profile/README.md#1-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%86%8C%EA%B0%9C


## 1. 프로젝트 소개

펀핏은 운동인들을 위한 피트니스 플랫폼으로, 운동 관련 커뮤니티와 PT 수업 예약 및 관리 서비스를 제공합니다.

- 피티 수업 예약 기능
- 수업 일지 및 식단/운동 기록 기능
- 서비스 이용자 간의 커뮤니티 기능
- 한 시간 단위로 업데이트되는 인기글 제공

## 2. 사용 기술

### Backend

- Java
- Spring Boot
- Spring Cloud
- JPA/Hibernate
- RabbitMQ

### Database

- MySQL
- Redis

### **Infra**

- Docker
- Github Actions
- AWS EC2, RDS, Elasticache

## 3. Backend API Swagger Link

//TODO

## 4. Architecture

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/711cf447-11bb-4a07-a594-8339c2304513/6b49d407-0f96-43dd-95dd-a6065208de48/image.png)

### MSA(Micro Service Architecture)

- Spring Cloud Gateway를 통해 게이트웨이 구축 및 공통 로직 처리
- 마이크로서비스들의 관리를 용이하게 하기 위해 Netflix Eureka를 통한 Service Discovery 구현
- 게이트웨이를 Public Subnet에, 그 외 마이크로서비스들은 Private Subnet에 위치시켜 외부에서 마이크로서비스로의 직접 접근을 막고, 반드시 게이트웨이를 통해 접근하도록 함
- 마이크로서비스마다 각자 독립된 데이터베이스(MySQL)를 갖고, Community Service는 인기글 데이터 저장을 위해 Redis를 추가로 갖는다. (LINK)

### 마이크로서비스 간 통신(**FeignClient, RabbitMQ, Caffeine)**

//TODO

- 마이크로서비스 간 **FeignClient**와 **RabbitMQ**를 통한 동기/비동기 통신
- 타 마이크로서비스로부터 받은 데이터를 **로컬 캐시**에 캐싱 (LINK!!!)

### Redis(Elasticache)

//TODO

- Community Service는 인기글계산과 저장을 위해 Redis 사용 (!!)
- Local: HA를 위해 복제본 구성 및 페일오버를 자동화하기 위해 Sentinel 구성
- AWS: 배포 시  ~~때문에 Elasticache 사용

### Docker

//TODO

### CI/CD

- Docker와 Github Actions를 통한 CI/CD 파이프라인 구축
    - Github main 브랜치에 push → Github Actions를 통해 빌드 파일을 이미지화하여 Docker Hub에 업로드 → EC2 접속 후 Docker Hub의 이미지를 pull & run

## 5. 제약 조건을 통한 예약 동시성 제어

### 관련 블로그 포스팅

- [예약 동시성 제어 과정](https://olsohee.tistory.com/226)
- [MySQL의 갭 락과 넥스트 키 락에 대해](https://olsohee.tistory.com/222)

### 문제점

- 한 트레이너는 특정 시간에 하나의 예약만 받을 수 있다. 그러나 여러 회원이 동시에 요청을 보낼 경우, 2개 이상의 예약이 완료되는 문제가 발생했다.
- 예약 로직은 다음과 같다.
    1. 트레이너 이메일과 예약 시간을 기준으로 예약 내역을 조회한다. (SELECT)
    2. 조회된 예약이 없으면 새로운 예약을 생성하여 저장한다. (INSERT)

### 해결 방안

- Unique Constraint 제약 조건을 통해 문제를 해결했다.
- 예약을 의미하는 Schedule 엔티티에 @UniqueConstraint 어노테이션을 붙여서 date_time과 trainer_email을 기준으로 제약 조건을 걸었다.
- 따라서 DDL에 제약 조건이 추가되고, MySQL 내부적으로는 유니크 키가 생성된다.
- 따라서 DB에 중복 데이터가 삽입되면, 애플리케이션 로직으로 `DataIntegrityViolationException` 예외가 던져지고, 애플리케이션 로직에서 해당 예외를 잡아서, 비즈니스 예외로 변환하고 적절한 예외 응답을 내리도록 구현했다.

## 6. 반정규화 및 인덱스를 통한 게시글 페이징 조회 성능 개선

### 관련 블로그 포스팅

- [반정규화를 통한 성능 향상](https://olsohee.tistory.com/169)

### 문제점

- 게시글 페이징 조회 시 게시글 데이터 뿐만 아니라 게시글의 좋아요, 댓글, 북마크 수까지 함께 반환해야 한다. 따라서 게시글과 연관된 좋아요, 댓글, 북마크 엔티티까지 함께 조회하면서 N+1 문제가 발생했다.
- 그리고 좋아요 순 조회 시 게시글을 풀스캔하는 문제가 발생했다.
    - 좋아요, 댓글, 북마크는 게시글 엔티티의 필드 값이 아니라 별도의 엔티티이다. 따라서 좋아요, 댓글, 북마크 순으로 정렬해서 페이징 조회 시, Pageable 객체를 통한 자동 페이징 처리가 불가하다. Pageable 객체는 엔티티의 필드 값을 기준으로 정렬 및 페이징을 제공하기 때문이다.
    - 따라서 좋아요/댓글/북마크 순 정렬 페이징 조회를 위한 메서드를 각각 생성해야 한다.
    - 그리고 예를 들어, 좋아요 순 정렬 페이징 조회 시 해당 메서드가 실행될 때 예를 들어 post와 likes 테이블을 조인한다. 그리고 조인 결과가 많은(좋아요 수가 많은) post 레코드를 선별한다. 이 과정에서 post 테이블을 풀스캔한다.

### 해결 방안

- 좋아요/댓글/북마크 수를 게시글 엔티티의 필드로 추가했다. 그리고 추가된 필드들에 인덱스를 걸어, 해당 필드를 기준으로 정렬 조회 시 인덱스를 통한 성능 개선이 이뤄지도록 했다.

### 결과

- 반정규화를 통해 조인이 일어나지 않기 때문에 N+1이 일어나지 않는다.
- 좋아요/댓글/북마크 순 정렬 페이징 조회 시, Pageable 객체를 통한 자동 페이징 처리가 가능하므로 정렬 조건마다 메서드를 생성할 필요가 없다.
- 좋아요/댓글/북마크 순 정렬 페이징 조회 시, 풀스캔이 일어나지 않고 인덱스를 통해 조회한다.
- 10만 건의 데이터 중 좋아요 수가 많은 100 건을 페이징 조회하는 테스트를 진행했을 때, 기존 방법에 비해 TPS가 약 661.2%가 증가(6.7 → 51)했다.

## 7. 비관적 락을 통한 좋아요 동시성 제어

### 관련 블로그 포스팅

[좋아요 동시성 제어 과정](https://olsohee.tistory.com/113)

### 문제점

- 앞서 게시글 페이징 조회 성능 개선을 위해 좋아요 수(likeCount)를 게시글 엔티티의 필드로 추가하면서, 동시성 제어가 필요해졌다.
- 동시성 제어가 필요한 상황은 다음 두 경우가 있다.
    1. 사용자는 같은 게시글에 좋아요를 2개 이상 누를 수 없다.
    2. 여러 명의 사용자가 동시에 같은 게시글에 좋아요를 누를 경우 해당 게시글의 좋아요 수가 덮어 씌워지면 안 된다.
- 기존 코드에서 두 경우를 테스트한 결과, 두 경우 모두 ****데드락이 발생했다.
- 데드락의 원인은 MySQL이 **조회하는 레코드에 외래 키가 있을 때, 외래 키에 해당하는 테이블 레코드에 S락을 걸기 때문**이다. 즉, 게시글 테이블(post)과 좋아요 테이블(likes)을 조인하는 과정에서, 각 트랜잭션은 likes와 연관된 post 테이블의 레코드에 S락을 건디. 그리고 post의 likeCount 필드 값을 UPDATE하기 위해 배타 락을 걸어야 하는데, 상대 트랜잭션이 S락을 걸고 있기 때문에 대기하게 되면서 데드락 발생한다.

### 해결 방안

- 비관적 락을 적용하여 조회 시 SELECT FOR UPDATE 쿼리로 X락을 걸었다.

### 결과

- 쓰기 락을 걸기 때문에 데드락이 발생하지 않으며 동시성 제어에 성공한다.

## 8. Redis와 스케줄러를 통한 인기글 추출 기능

- 한 시간 동안 조회 수가 많았던 게시글 10개가 특정 시간의 인기글이 된다. 따라서 인기글을 계산하기 위해 한 시간 동안 각 게시글의 조회 수를 카운팅해야 한다.
- **인기글 카운팅**: 빠른 카운팅을 위해 **Redis의 sorted set** 사용했다. 그리고 게시글이 조회될 때마다 Redis에 조회 수가 반영된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/711cf447-11bb-4a07-a594-8339c2304513/52fc6306-829b-4105-bde5-66b738fa00d6/Untitled.png)

- **인기글 추출**: **스케줄러**를 통해 한 시간마다 조회 수가 가장 많은 상위 10개의 게시글 ID를 Redis에서 조회한 후, 해당 게시글들을 토대로 인기글 데이터를 생성하여 **Redis에 캐싱**해둔다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/711cf447-11bb-4a07-a594-8339c2304513/93ace477-c11d-44d1-851a-e09141e18fba/image.png)

- 스케줄러를 통해 인기글 추출이 이뤄질 때 네트워크 문제로 인해 실패할 경우를 대비하여, 3번까지 **재시도**한다.

## 9. 로컬 캐싱

- 사용자 정보는 Auth Service에서 관리된다. 따라서 각 마이크로서비스는 사용자 정보가 필요할 때 Auth Service로부터 데이터를 받아와야 한다.
- 기존에는 Auth Service로부터 데이터를 받아와서 **Redis에 캐싱**해뒀다. 그러나 각 마이크로서비스에서 사용자 정보는 자주 조회되는 데이터이고, 변경이 적으며, 데이터 크기가 크지 않다. 따라서 네트워크 통신이 이뤄지는 Redis보다 로컬 캐싱이 더 적합하다고 판단하여 **로컬 캐싱으로 변경**했다.
- TTL 등 캐시 관리를 위한 기능을 제공하며 높은 성능으로 알려진 Caffeine을 통해 로컬 캐싱을 구현했다.
- 사용자 정보 조회 흐름
    1. Local Cache에서 사용자 정보를 조회한다.
    2. 없으면, Auth Service로부터 사용자 정보를 받아온다. 
    3. 응답받은 데이터를 Local Cache에 캐싱해 둔다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/711cf447-11bb-4a07-a594-8339c2304513/451409d9-2c13-4812-b582-ed9c056e1723/image.png)

## 10. CircuitBreaker를 통한 장애 격리

### 관련 블로그 포스팅

- [Resilience4j의 서킷 브레이커](https://olsohee.tistory.com/187)

### 문제점

- 각 마이크로서비스는 사용자 정보가 필요할 때, Auth Service로부터 데이터를 받아온다.
- 이때 Auth Service 장애 시 Auth Service에게 데이터를 요청한 Pt, Community Service에게까지 장애가 전파되는 문제가 발생한다.

### 해결 방안

- Resilience4j의 CircuitBreaker를 사용하여 fallback 값을 정의했다. Auth 서비스에 장애가 발생하면, 사용자 명을 “알수없음”으로 반환하여 사용자 명 외 서비스는 정상 동작한다.
