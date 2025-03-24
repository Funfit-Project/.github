# 펀핏(Funfit) 프로젝트

## 프로젝트 개요 및 소개

- 펀핏(Funfit)은 운동인들을 위한 피트니스 플랫폼으로 운동 관련 커뮤니티와 PT 수업 예약 서비스를 제공합니다.
- 서비스 특성에 따라 User, Pt, Community 서비스로 나눠 분산 환경을 구성했고, 분산 환경에서의 통신 방식, 장애 격리, 데이터 일관성 등을 고민했습니다.

## 사용 기술

### Backend

- Java
- Spring Boot
- Spring Cloud
- JPA/Hibernate
- Kafka
- Docker

### Database

- MySQL
- Redis

## Architecture
<p align="center"><img src="https://github.com/user-attachments/assets/78c23268-9ca5-4dbb-a041-84a929d41f3c" width="650" height="450"></p>

- Spring Cloud Gateway를 통해 게이트웨이 구축 및 인증 처리
- 각 마이크로서비스는 독립된 RDB(MySQL)를 바라봄
- 마이크로서비스 간 비동기 통신에 Kafka 사용
- Refresh Token 관리, 인기글 캐싱, 동시성 제어 등의 목적으로 Redis 사용
