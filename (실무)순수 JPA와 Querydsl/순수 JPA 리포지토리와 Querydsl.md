순수 JPA 리포지토리

* MemberJPARepository

<br/>

```java

@Repository
public class MemberJPARepository {

    private final EntityManager em;
    
    //@RequiredArgsConstructor를 사용하면 생성자를 따로 만들지 않아도 자동으로 bean이 주입된다.
    //private final로 여러가지 bean을 생성해도 알아서 모두 주입된다.
    public MemberJPARepository(EntityManager em) {
        this.em = em;
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username")
                .setParameter("username", username)
                .getResultList();
    }
}

```

<br/>

JPA로 만든 일반적인 리포지토리 형태이다. 여기에 Querydsl을 사용해보자.

```java

@Repository
public class MemberJPARepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJPARepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username")
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}

```

<br/><br/>

1. JPAQueryFactory 주입

2. JPQL을 사용하는 메서드(findAll(), findByUsername()): Querydsl 버전 추가<br/>
-> Querydsl 장점: 자바 코드 어시스턴트가 편리하고 문법 오류에 대해 컴파일 오류로 잡을 수 있다.

<br/>

+) JPAQueryFactory 스프링 빈 등록<br/>
다음과 같이 JPAQueryFactory를 스프링 빈으로 등록해서 주입 받아 사용해도 된다.

```java

@Bean
JPAQueryFactory jpaQueryFactory(EntityManager em) {
	return new JPAQueryFactory(em);
}

```

<br/>

이 방식을 사용하면 생성자 주입을 다음과 같이 수정하면 된다.

```java

@Repository
public class MemberJPARepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;
    
    public MemberJPARepository(EntityManager em, JPAQueryFactory queryFactory) {
        this.em = em;
        this.queryFactory = queryFactory; // new가 아닌 bean으로 등록된 JPAQueryFactory를 인젝션 해주면된다
    }
}

```

<br/>

+) 동시성 문제는 걱정하지 않아도 된다. 스프링이 주입해주는 엔티티 매니저는 가짜 프록시 매니저이다.  <br/>
이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.
