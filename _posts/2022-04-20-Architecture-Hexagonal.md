# Hexagonal Architecture

## 헥사고날 아키텍쳐란

- 객체 지향 소프트웨어 설계의 알려진 구조적 함정( 레이어 간의 원하지 않는 종속성이나 비즈니스 로직으로 인한 사용자 인터페이스 코드의 오염과 같은)을 피하기 위해 Alistair Cockburn에 의해 발명


## 장점
- 아키텍쳐 확장이 용이
- SOLID 원칙을 쉽게 적용 할 수 있다.
- 모듈 일부를 배포하기 용이
- 테스트를 위해 모듈을 가짜로 바꿀 수 있음 -> 테스트 안정성 
- 더 큰 비지니스적 가치를 갖고 지속적 성장을 할 수 있느 도메인 모데에 관심을 둠

## 구성
- 내부(도메인)과 외부(인프라)로 구분
  - 내부 영역 - 순수한 비지니스 로직을 표현하고 캡슐화된 영역, 기능적 요구사항에 따라 먼저 설계
  - 외부 영역 - 내부 영역에서 기술을 분리하여 구성한 영역


### 포트와 어댑터

- 포트 - 내부 비지니스 영역을 외부에 노출한 API, (Inbound / Outbound) 로 구분
  - 인바운드 포트 - 내부 영역 사용을 위해 노출된 API
  - 아웃바운드 포트 - 외부 역역을 사용하기 위한 API
- 어댑터 - 외부세계와 포트간 교환을 조정, (Inbound / Outbound) 로 구분
  - 인바운드 어댑터 - 외부 애플리케이션/서비스와 내부 비지니스 영역(인바운드 포트) 간 데이터 교환을 조정
  - 아웃바운드 어댑터 - 내부 비지니스 영역(아웃바운드포트)과 외부 애플리케이션/서비스 간 데이터 교환을 조정

### 핵심

비지니스 로직이 표현로직이나 데이터 접근 로직에 의존하지 않는 것



---

참고사항

https://mesh.dev/20210910-dev-notes-007-hexagonal-architecture/
