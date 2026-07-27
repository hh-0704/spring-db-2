# spring-db-2

김영한님의 [**스프링 DB 2편 - 데이터 접근 활용 기술**](https://www.inflearn.com/course/스프링-db-2) 강의를 수강하며 클론 코딩한 학습용 레포지토리입니다.

하나의 상품 관리 예제(`itemservice-db`)를 두고, `ItemRepository` 인터페이스의 구현체만
**메모리 → JdbcTemplate → MyBatis → JPA → 스프링 데이터 JPA → Querydsl** 순으로 교체해가며
각 데이터 접근 기술의 사용법과 트레이드오프를 비교합니다.
후반부에서는 스프링 트랜잭션의 동작 원리와 전파(propagation) 옵션을 다룹니다.

## 기술 스택

| 구분 | 내용 |
| --- | --- |
| Language | Java 25 |
| Framework | Spring Boot 4.0.6 |
| Build | Gradle |
| View | Thymeleaf |
| DB | H2 |
| Data Access | JdbcTemplate, MyBatis, JPA(Hibernate), Spring Data JPA, Querydsl |
| Etc | Lombok, JUnit5 |

## 핵심 설계

- **인터페이스 기반 저장소** — 서비스 계층은 `ItemRepository` 인터페이스에만 의존합니다. 실제 구현체는 `config` 패키지의 설정 클래스가 빈으로 등록하므로, 데이터 접근 기술을 바꿔도 서비스·웹 계층 코드는 그대로입니다(OCP).
- **컴포넌트 스캔 범위 제한** — `@SpringBootApplication(scanBasePackages = "hello.itemservice.web")`로 웹 계층만 스캔하고, 나머지는 `@Import`로 수동 등록해 설정 교체 지점을 한 곳에 모았습니다.
- **프로파일 분리** — `@Profile("local")`을 적용한 `TestDataInit`으로 로컬 실행 시에만 테스트 데이터를 주입하고, 테스트 실행 시에는 제외합니다.
- **DTO 분리** — 등록/수정/검색 각각의 목적에 맞는 DTO를 두어, 하나의 객체가 여러 역할을 겸하면서 생기는 결합을 피합니다.

## 강의 내용

### 1. 데이터 접근 기술 - 시작
프로젝트 구조와 도메인 설계를 잡고, 메모리 저장소로 동작하는 상품 관리 애플리케이션을 만듭니다. 이후 모든 구현체가 공유할 `ItemRepository` 인터페이스와 설정 클래스 기반 구현체 교체 구조를 정의합니다.

### 2. 스프링 JdbcTemplate
반복적인 JDBC 코드(커넥션 획득/해제, 예외 변환, 결과 매핑)를 제거하는 `JdbcTemplate`을 적용합니다. 이름 지정 파라미터를 쓰는 `NamedParameterJdbcTemplate`, INSERT를 간소화하는 `SimpleJdbcInsert`, 동적 쿼리 작성 방식까지 단계적으로 개선합니다.

### 3. 데이터 접근 기술 - 테스트
테스트를 실제 DB와 연동하며 겪는 문제들을 다룹니다. 테스트 전용 데이터베이스 분리, 임베디드 모드 H2 활용, 그리고 `@Transactional`을 이용해 테스트 종료 후 자동 롤백함으로써 테스트 간 격리를 보장하는 방법을 학습합니다.

### 4. MyBatis
SQL을 XML 파일로 분리해 관리하는 SQL 매퍼 기술입니다. `@Mapper` 인터페이스와 XML 매핑, `<if>`·`<where>` 같은 동적 쿼리 문법을 익히고, MyBatis가 인터페이스 구현체를 프록시로 만들어 스프링 빈으로 등록하는 원리를 확인합니다.

### 5. JPA
객체와 관계형 데이터베이스를 매핑하는 ORM 표준입니다. `@Entity`, `@Id`, `@Column` 매핑과 `EntityManager`를 통한 CRUD, 영속성 컨텍스트의 변경 감지(dirty checking), 객체 지향 쿼리인 JPQL을 다룹니다. 예외를 스프링 표준 예외로 변환하는 `@Repository`의 역할도 함께 살펴봅니다.

### 6. 스프링 데이터 JPA
인터페이스만 선언하면 구현체를 만들어주는 스프링 데이터 JPA를 적용합니다. 공통 CRUD 제공, 메서드 이름으로 쿼리를 생성하는 쿼리 메서드, 복잡한 쿼리를 위한 `@Query`를 사용해 반복 코드를 크게 줄입니다.

### 7. Querydsl
문자열 기반 JPQL의 한계(컴파일 시점에 오류를 못 잡는 문제)를 해결하는 타입 세이프 쿼리 빌더입니다. Q타입 생성 설정, 자바 코드로 작성하는 동적 쿼리, `BooleanExpression`을 활용한 조건 조합을 학습합니다.

### 8. 데이터 접근 기술 - 활용 방안
각 기술의 장단점을 정리하고, 실무에서 자주 쓰이는 **스프링 데이터 JPA + Querydsl** 조합을 구성합니다. 단순 CRUD와 복잡한 조회 쿼리를 어떻게 분리해 설계할지, 그리고 트레이드오프를 어떻게 판단할지 다룹니다.

### 9. 스프링 트랜잭션 이해
`@Transactional`이 AOP 프록시로 동작하는 원리를 파헤칩니다. 프록시 내부 호출 시 트랜잭션이 적용되지 않는 문제, 적용 대상 메서드의 제약(public 등), 트랜잭션 옵션(`readOnly`, `rollbackFor`, `isolation`), 예외 종류에 따른 커밋·롤백 정책을 정리합니다.

### 10. 트랜잭션 전파 1 - 기본
트랜잭션이 이미 진행 중일 때 또 다른 트랜잭션이 시작되면 어떻게 동작하는지 학습합니다. 물리 트랜잭션과 논리 트랜잭션 개념, 외부/내부 트랜잭션의 커밋과 롤백 조합, `rollback-only` 표시, 그리고 `REQUIRES_NEW`로 트랜잭션을 분리하는 방법을 다룹니다.

### 11. 트랜잭션 전파 2 - 활용
회원 등록과 로그 저장 예제를 통해 전파 옵션을 실제 시나리오에 적용합니다. 로그 저장 실패가 회원 가입 전체를 롤백시키는 문제를 재현하고, `REQUIRES_NEW`나 별도 처리로 해결하면서 각 방식의 부작용까지 검토합니다.

## 참고

- 강의: [스프링 DB 2편 - 데이터 접근 활용 기술](https://www.inflearn.com/course/스프링-db-2)
- 선행 학습 레포: [spring-mvc-2](https://github.com/hh-0704/spring-mvc-2)
