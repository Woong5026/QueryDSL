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
                .from(member)
                .leftJoin(member.team, team)
                .where(builder)
                .fetch();

    }

```

+) username, teamName과 같은 String 타입의 경우 null, "" 모두 고려해주기 위해 StringUtils.hasText()를 사용한다.

(예제에서는 StringUtils를 static import 하였다)

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
