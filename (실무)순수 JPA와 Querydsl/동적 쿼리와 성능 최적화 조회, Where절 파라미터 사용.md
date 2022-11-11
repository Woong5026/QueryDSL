### 동적 쿼리와 성능 최적화 조회 - Where절 파라미터 사용

```java

public List<MemberTeamDto> search(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id,
                    member.username,
                    member.age,
                    team.id,
                    team.name))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            )
            .fetch();
}

private BooleanExpression usernameEq(String username) {
    return hasText(username) ? member.username.eq(username) : null;
}

private BooleanExpression teamNameEq(String teamName) {
    return hasText(teamName) ? team.name.eq(teamName) : null;
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe != null ? member.age.goe(ageGoe) : null;
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe != null ? member.age.loe(ageLoe) : null;

```

--- 

<br/>

동적 쿼리는 where절 파라미터 방식을 권장한다.

1. 메서드를 재사용할 수 있다. <br/>
ex) 위의 코드는 queryFactory에서 QMemberTeamDto를 조회했지만 엔티티를 조회하고 싶다면 Member를 조회해도 된다

2. 메서드를 조합할 수 있다. <br/>
ex)

```java

private BooleanExpression ageBetween(int ageLoe, int ageGoe) {
    return ageGoe(ageGoe).and(ageLoe(ageLoe));
}

```

<br/>

+) 메서드 조합시 null 처리 주의 <br/>
위 예시에서 만약 ageGoe()가 null을 반환한다면 NullPointException이 발생하게 된다. <br/>
메서드를 조합할 때는 이와 같은 null 처리를 주의해야 한다. 공부하면서 알게 된 두 가지 방법을 소개한다.

<br/>

1. 메서드 조합시 BooleanBuilder 사용

```java

private BooleanBuilder ageBetween(Integer ageGoe, Integer ageLoe) {
    BooleanBuilder booleanBuilder = new BooleanBuilder();
    return booleanBuilder
            .and(ageGoe(ageGoe))
            .and(ageLoe(ageLoe));
}

```

ageGoe()가 null이더라도 무시하기 때문에 안전하게 조합할 수 있다.

<br/>

2. 각 메서드마다 BooleanBuilder 모두 사용

```java

        private BooleanBuilder usernameEq(String username) {
          return nullSafeBuilder(() -> member.username.eq(username));
        }

        private BooleanBuilder teamNameEq(String teamName) {
          return nullSafeBuilder(() -> team.name.eq(teamName));
        }

        private BooleanBuilder ageGoe(Integer ageGoe) {
          return nullSafeBuilder(() -> member.age.goe(ageGoe));
        }

        private BooleanBuilder ageLoe(Integer ageLoe) {
          return nullSafeBuilder(() -> member.age.loe(ageLoe));
        }

        private BooleanBuilder nullSafeBuilder(Supplier<BooleanExpression> f) {
          try {
            return new BooleanBuilder(f.get());
          } catch (Exception e) {
            return new BooleanBuilder();
          }
        }

        private BooleanBuilder ageBetween(Integer ageLoe, Integer ageGoe) {
          return ageLoe(ageLoe).and(ageGoe(ageGoe));
        }

```

모든 메서드의 반환 타입을 BooleanBuilder로 한다. null 조건은 nullSafeBuilder() 메서드로 검증한다.

