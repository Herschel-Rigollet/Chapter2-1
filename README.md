## 프로젝트

## Getting Started

### Prerequisites

#### Running Docker Containers

`local` profile 로 실행하기 위하여 인프라가 설정되어 있는 Docker 컨테이너를 실행해주셔야 합니다.

```bash
docker-compose up -d
```

## 항해 플러스 E-Commerce 과제
### 선택한 아키텍처 패턴
#### Layered + Interface 아키텍처
- 각 계층은 어플리케이션 내에서의 특정 역할과 관심사(화면 표시, 비즈니스 로직 수행, DB 작업 등)별로 구분됩니다.
- 이는 Layered Architecture 의 강력한 기능인 '관심사의 분리 (Separation of Concern)'를 의미합니다.
- 특정 계층의 구성요소는 해당 계층에 관련된 기능만 수행합니다.
- 이런 특징은 높은 유지보수성과 쉬운 테스트라는 장점이 존재합니다.
- 단순하고 대중적이면서 비용도 적게 들어 사실상 표준 아키텍처입니다.

위의 특징을 머릿속에 새기며 해당 아키텍처 패턴을 선택했고, 과제에 적용해 보고자 했습니다만...실제로 해당 아키텍처 패턴을 사용한 주된 이유는 다음과 같습니다.

1. 가장 익숙한 아키텍처 패턴이 레이어드 + 인터페이스 아키텍처였습니다.
2. 이왕 프로젝트를 진행해 보는 거 헥사고날과 같은 좀 더 효율적인 패턴을 적용해 보고 싶었습니다. 그러나 익숙하지 않은 아키텍처를 적용하려 할 때 익숙해지는 데에만 일주일 이상이 걸릴 것 같아 추후 리팩토링에 좀 더 용이할 것 같은 방향을 선택했습니다.
3. 해당 아키텍처 패턴을 사용하여 제한된 시간 내에 빠르게 완성한 후, 다른 아키텍처 패턴으로 바꿔가면 전체적인 아키텍처 패턴에 대한 이해를 하기에 좀 더 괜찮을 것 같다는 생각이었습니다.
4. 단방향 흐름에 유지하는 구조를 사용하고자 했습니다. 제가 실무에서 겪었던 구조는 Service와 ServiceImpl이 존재하나 존재 의미를 몰랐고, 모든 비즈니스 로직이 Controller에 집중된 형태였습니다.
그래서 추상화에 의존하여 DIP가 부분적으로 적용되는 구조가 어떤 것인지, 단방향 흐름이 유지될 경우 프로젝트가 점점 커질 때 어떤 효율이 나타나는지 궁금했습니다.
5. 아직 TDD에 완전히 익숙해지지 않았기 때문에 Mock 객체를 통한 단위 테스트 코드 작성에 익숙해지고자 해당 아키텍처 패턴을 선택한 것도 있습니다.

#### 각 레이어별 책임
Controller → Application → Domain ← Interface → Infra
1. Controller Layer (입출력 전담)

| 항목             | 내용                                                                 |
|------------------|----------------------------------------------------------------------|
| 책임          | HTTP 요청 수신, DTO 변환, 응답 반환                                    |
| 비즈니스 로직 | 없음 – 유즈케이스 실행만 위임                                         |
| 호출 대상     | Application Layer                                                    |

2. Application Layer (UseCase 실행)

| 항목         | 내용                                 |
| ---------- | ---------------------------------- |
| 책임       | 유즈케이스 단위 처리 (ex. 주문, 결제, 충전 등)     |
| 협력 객체   | 여러 도메인 + Repository 조합 가능          |
| 트랜잭션 처리 | 필요시 @Transactional 지정              |
| 도메인 로직   | 없음 – 검증/상태 변경은 도메인 객체에 위임          |
| 호출 대상   | Domain Layer, Repository Interface |

3. Domain Layer (핵심 비즈니스 로직)

| 항목       | 내용                                                               |
| -------- | ---------------------------------------------------------------- |
| 책임     | 잔액 차감, 재고 감소, 할인 계산 등 순수 비즈니스 로직 구현                              |
| 외부 의존  | 없음 – DB, Redis 등 어떤 기술에도 의존하지 않음                                 |
| 구성 요소 | Entity, Value Object, 도메인 메서드 (`decreaseStock`, `applyDiscount`) |

4. Infra Layer (기술 구현 담당)

| 항목         | 내용                                               |
| ---------- | ------------------------------------------------ |
| 책임       | Repository 인터페이스 구현, 외부 시스템 연동(DB, Redis, API 등) |
| 상위 계층 의존 | 없음 – 오직 인터페이스만 구현                                |

5. 테스트 코드

| 계층          | 테스트 방법                     | 외부 의존성       |
| ----------- | -------------------------- | ------------ |
| Domain      | 순수 단위 테스트                  | 없음         |
| Application | Mock 기반 단위 테스트             | 인터페이스 Mock |
| Infra       | 통합 테스트 또는 Spring Boot Test | 필요시 실환경    |


#### 패키지 구조
<img width="236" height="768" alt="스크린샷 2025-07-25 04 09 12" src="https://github.com/user-attachments/assets/2ad81c11-290c-400a-bb61-ef5c460f3afa" />

멘토링 노트에 적어주신 것 중 해당 구조를 택하게 되었습니다. 위 구조를 사용한 결과 배운점은 다음과 같습니다.
- 레이어드 + 인터페이스 아키텍처가 '레이어별로 확실히 나뉜 역할'에 집중한다는 느낌이어서 레이어별로 나누고자 했습니다.
- 이렇게 레이어대로 나눌 경우 추후 코드를 수정해야 할 때 '이 코드는 A 레이어에 있는 것이니 해당 패키지로 가서 찾으면 된다'가 잘 적용이 될 것 같았습니다.
- 기능 구현을 하는 중반까지는 위의 내용이 잘 적용되는 것 같았습니다.
- 너무 한 곳에 몰려 있어서 중간부터 패키지 구조 정리가 잘 안 되는 느낌이었습니다.
- 구조를 바꾸고자 했는데 시간이 촉박한 상황이었고, '연결되어 있는 의존들 때문에 힘들다', '필요 없는 공통 코드들이 딸려갈 수밖에 없는 구조이다'라고 말씀해 주신 단점이 그대로 적용됐습니다..
- 일부러 코치님이 권장해 주신 것의 반대로 한 건 아니고 패키지 구조를 머릿속에 그리고 전체적인 그림을 그리다 보니 해당 구조를 택하게 되었는데, 직접 해당 구조를 겪어 보니 왜 다른 구조를 권장하셨는지 깨닫게 되었습니다. 보기가 굉장히 힘든 구조인 것 같습니다.
- 실무에서는 권장해 주신 구조를 사용했던 걸로 기억하는데, 그 구조를 볼 땐 몰랐으나 다른 구조의 한계점을 경험하고 나니 확실히 도메인에 특화된 구조가 적절하다고 느껴집니다.
- 또한 기능을 구현하다 보니 코치님이 이건 정말 추천하지 않는다고 하셨던 아키텍처 관점에서의 JPA 구조를 제가 그대로 하고 있었습니다..................................정말 구제불능인 것 같습니다
- 이것 역시 일부러 그런 건 아닙니다...............기능 구현을 일단 해내야 한다는 생각에 조급해져서 그쪽에 집중하다 보니 해당 비추천 구조로 가고 있는 것조차 몰랐던 것 같습니다......
- 똥을 찍어먹어 봐야 된장인 걸 아는 한 주가 되었지만 그래도 직접 경험한 덕분에 한계점과 단점을 명확히 알게 되었습니다.
- 추후 패키지 구조를 수정하거나 리팩토링을 할 때 권장해 주셨던 방법들의 특징에 더 집중할 수 있게 된 것 같습니다.
- 테스트를 하기 용이했던 것 하나는 장점으로 다가왔습니다. 물론 다른 구조도 테스트하기 용이할 수 있겠지만요........
- 제가 장점이라고 생각했던 부분들은 크게 부각되지 않고 '이 정도면 괜찮겠지' 하고 넘어갔던 부분들이 크나큰 한계점으로 다가왔다는 점에 더 집중을 하게 된 챕터였습니다. 이렇게라도 배울 수 있어 다행이긴 합니다.


#### 20250731 수정
패키지 구조 전면 수정했습니다.
```com.example.project   
├── application
│   ├── user
│   │   ├── UserService.java
│   │   └── UserRepository.java
│   ├── product
│   │   ├── ProductService.java
│   │   └── ProductRepository.java
│   └── order
│       ├── OrderService.java
│       └── OrderRepository.java
│
├── domain
│   ├── user
│   │   └── User.java
│   ├── product
│   │   └── Product.java
│   └── order
│       ├── Order.java
│       └── OrderItem.java
│
├── infrastructure
│   ├── user
│   │   └── UserRepositoryImpl.java
│   ├── product
│   │   └── ProductRepositoryImpl.java
│   └── order
│       └── OrderRepositoryImpl.java
│
├── presentation
│   ├── user
│   │   ├── UserPointController.java
│   │   └── response
│   │       └── UserResponse.java
│   └── product
│   │   ├── ProductController.java
│   │   └── response
│   │       └── ProductResponse.java
│   └── order
│       ├── OrderController.java
│       ├── request
│       │   └── OrderRequest.java
│       └── response
│           └── OrderResponse.java
│
├── common
│   └── response
│       ├── CommonResponse.java
│       └── CommonResultCode.java
│
└── test
    └── application
        ├── user
        │   └── UserServiceTest.java
        └── product
        │   └── ProductServiceTest.java
        └── order
            └── OrderServiceTest.java   
```

- 레이어별로 수정
- 굳이 필요하지 않고 중복되는 엔티티를 삭제하고, 도메인에 집중
- 각 계층에 필요한 역할만을 수행하는 데 집중함
  - Controller: 서비스에 유즈케이스 위임, DTO로 응답 변환, 공통 응답 포맷으로 래핑, HTTP 응답 반환 등
  - Service: 도메인 로직을 직접 수행하지 않음, Repository에서 조회 후 도메인에 위임
  - Domain: 도메인 로직 구현에 집중
  - Infrastructure: JPA를 통해 DB에서 조회
