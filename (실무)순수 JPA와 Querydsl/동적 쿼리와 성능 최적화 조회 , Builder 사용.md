이번 장은 QueryDSL을 이용한 동적쿼리처리방법과 Dto를 이용한 성능 최적화를 만들어보겠다

Member와 Team에서 원하는 정보만 가져올 수 있도록 MemberTeamDto 생성


* 조회용 DTO 추가

```java

@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}

```

+) @QueryProjection

@QueryProjection을 이용하면 불변 객체 선언, 생성자를 그대로 사용할 수 있기 때문에 권장되는 패턴

이 방식은 new QDTO(QType의 클래스를 생성)로 사용하기 때문에 런타임 에러뿐만 <br/>
아니라 컴파일 시점에서도 에러를 잡아주고, 파라미터로도 확인할 수 있다는 장점이 있다

DTO는 Repository 계층의 조회 용도뿐만 아니라 Service, Controller 모두 사용되기 때문에 <br/>
@QueryProjection 어노테이션을 DTO에 적용시키는 순간, 모든 계층에서 쓰이는 DTO가 Querydsl에 의존성을 가지는 단점도 있다

<br/>

* 검색 조건용 DTO 추가

```java

@Data
public class MemberSearchCondition {
    //회원명, 팀명, 나이(ageGoe, ageLoe)
    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}

```

<br/>

* 동적 쿼리 - Builder 사용

```java

public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition){

        BooleanBuilder builder = new BooleanBuilder();
        // StringUtils.hasText는 파마미터가 null이나 "" 으로 들어올 수도 있기 때문에 사용
        if(hasText(condition.getUsername())){
            builder.and(member.username.eq(condition.getUsername()));
        }
        if(hasText(condition.getTeamName())){
            builder.and(team.name.eq(condition.getTeamName()));
        }
        // 나이가 특정나이 이상이거나 이하이거나 의 조건
        if(condition.getAgeGoe() != null){
            builder.and(member.age.goe(condition.getAgeGoe()));
        }
        if(condition.getAgeLoe() != null){
            builder.and(member.age.loe(condition.getAgeLoe()));
        }

        return queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")
                ))
                // dto와 member의 정보연동은 QMemberTeamDto의 정보와 QMember의 정보를 자동으로 조회하기에
                // .from(member)의 member와 select 내의 QMemberTeamDto를 자동으로 매핑해준다
                .from(member) 
                .leftJoin(member.team, team) // team의 데이터도 검색조건에 있기에 조인
                .where(builder)
                .fetch();

    }

```

+) username, teamName과 같은 String 타입의 경우 null, "" 이 들어올 수도 있기 때문에 StringUtils.hasText()를 사용한다.

(예제에서는 StringUtils를 static import 하였다)

<br/>

+) 엔티티를 먼저 조회하지 않고, 바로 DTO로 조회하는 경우?

위 예제를 보면 member나 team으로 조회하지 않고 별도의 dto를 생성해서 만드는데 <br/>
api스펙 변경이 되는것도 아니고 검색조건만 만드는 건데 dto로 조회하는 이유가 있을까?

DTO로 직접 조회하는 이유는 엔티티를 무시하고, **조회용 모델을 바로 만드는 것이 목표**
따라서 중간에 번거롭게 엔티티를 만들 이유가 없다


<br/>

* 테스트

```java

@Test
public void searchTest() {
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);
    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);
    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);

    MemberSearchCondition condition = new MemberSearchCondition();
    condition.setAgeGoe(35);
    condition.setAgeLoe(40);
    condition.setTeamName("teamB");

    List<MemberTeamDto> result = memberJPARepository.searchByBuilder(condition);

    assertThat(result).extracting("username").containsExactly("member4");
}

```

+) 만약 MemberSearchCondition 필드가 전부 null이어서 condition이 비어있다면 동적 쿼리는 모든 데이터를 불러오게 된다. <br/>
이는 트래픽이 많은 서버에서는 부담이 될 수 있다. 따라서, 동적 쿼리는 기본 초기 조건이나 페이징 처리를 해주는 것이 좋다.  
