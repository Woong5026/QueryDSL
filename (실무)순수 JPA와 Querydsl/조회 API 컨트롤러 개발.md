편리한 데이터 확인을 위해 샘플 데이터를 추가하자.

샘플 데이터 추가가 테스트 케이스 실행에 영향을 주지 않도록 다음과 같이 프로파일을 설정하자.

* src/main/resources/application.yml

```java

spring:
  profiles:
    active: local

```

<br/>

테스트는 기존 application.yml을 다음 경로로 복사하고, 프로파일을 test로 수정하자.

* src/test/resources/application.yml

```java

spring:
  profiles:
    active: test

```

이렇게 분리하면 main 소스코드와 테스트 소스 코드 실행시 프로파일을 분리할 수 있다.

<br/><br/>

* 샘플 데이터 추가

```java

@Profile("local")
@Component
@RequiredArgsConstructor
public class InitMember {

    private final InitMemberService initMemberService;

    @PostConstruct
    public void init() {
        initMemberService.init();
    }

    @Component
    static class InitMemberService {
        @PersistenceContext
        private EntityManager em;

        @Transactional
        public void init() {
            Team teamA = new Team("teamA");
            Team teamB = new Team("teamB");
            em.persist(teamA);
            em.persist(teamB);

            for (int i = 0; i < 100; i++) {
                Team selectedTeam = i % 2 == 0 ? teamA : teamB;
                em.persist(new Member("member" + i, i, selectedTeam));
            }
        }
    }
}

```

<br/>

* 조회 컨트롤러

```java

@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberJPARepository memberJPARepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJPARepository.search(condition);
    }
}

```

http://localhost:8080/v1/members?teamName=teamB&ageGoe=31&ageLoe=35 처럼 쿼리 파라미터로 조건을 추가해서 테스트할 수 있다. <br/>
-> postman을 사용하면 편리하다.

<br/>

+) <br/>
Q) 문득 pathvariable이나 RequestParam을 파라미터로 넣지 않았는데 주소창에 ? 뒤에 조건을 붙여서 매핑을 해주는 걸까? 라는 의문이 생겼다

A) 답은 @ModelAttribute가 생략되어 있는 것 <br/>
@ModelAttribute는 파라미터로 받은 객체에 요청파라미터를 바인딩하고 set하는 작업을 해주는 것 <br/>
이전에 MVC에서 배웠는데 갑자기 기억이 나질 않았다. 이래서 계속적인 반복과 코드 하나하나에 대한 의문이 중요한 것 같다



