# 펀핏(Funfit) 프로젝트
## 1. 프로젝트 소개 및 주요 기능
펀핏은 운동인들을 위한 피트니스 플랫폼으로, 운동 관련 커뮤니티와 PT 수업 예약 및 관리 서비스를 제공합니다.
- 피티 수업 예약 기능
- 수업 일지 및 식단/운동 기록 기능
- 서비스 이용자 간의 커뮤니티 기능
- 한 시간 단위로 업데이트되는 인기글 제공

## 2. Tech Stack
<img src="https://img.shields.io/badge/Java-7D929E?style=plastic&logo=Java&logoColor=white"> <img src="https://img.shields.io/badge/Spring Boot-6DB33F?style=plastic&logo=spring&logoColor=white"> <img src="https://img.shields.io/badge/Spring Cloud-6DB33F?style=plastic&logo=spring&logoColor=white"> <img src="https://img.shields.io/badge/JPA/Hibernate-1A1A1A?style=plastic&logo=JPA&logoColor=white">

<img src="https://img.shields.io/badge/MySQL-4479A1?style=plastic&logo=mysql&logoColor=white"> <img src="https://img.shields.io/badge/Redis-FF4438?style=plastic&logo=redis&logoColor=white"> <img src="https://img.shields.io/badge/MongoDB-47A248?style=plastic&logo=mongodb&logoColor=white">

<img src="https://img.shields.io/badge/RabbitMQ-FF6600?style=plastic&logo=rabbitmq&logoColor=white"> <img src="https://img.shields.io/badge/Docker-2496ED?style=plastic&logo=docker&logoColor=white"> 

<img src="https://img.shields.io/badge/GitHub Actions-2088FF?style=plastic&logo=githubactions&logoColor=white"> <img src="https://img.shields.io/badge/AWS EC2-FF9900?style=plastic&logo=amazonec2&logoColor=white">

## 3. Architecture
![image](https://github.com/Funfit-Project/community-service/assets/108605017/e4124db2-39ee-49ae-94b7-a3a61ec681e5)

### 마이크로 서비스 아키텍처(MSA)
- Spring Cloud Gateway를 통한 게이트웨이 구축 및 공통 로직 처리
- RabbitMQ와 FeignClient를 통한 마이크로 서비스 간 통신
- Redis 기반 캐싱
### Fault Tolerant & High Availability
- 서킷 브레이커를 통한 내결함성 시스템 구축
- Redis 복제와 센티널을 통한 고가용성 시스템 구축
### CI / CD
- Docker와 Github Actions를 통한 CI/CD 파이프라인 구축


