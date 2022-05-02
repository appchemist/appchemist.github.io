---
layout: post
title: 자바 ORM 표준 JPA 프로그래밍 - JPA 시작
categories: [Study, Book]
tags: [JPA, ORM]
---

# Table of Contents

1.  [JPA 시작](#org94804b8)
    1.  [객체 매핑 시작](#orgb6a1fb3)
    2.  [persistence.xml 설정](#org3339df4)
        1.  [데이터베이스 방언](#orgdcf3cbc)
    3.  [애플리케이션 개발](#orgc9d1515)
        1.  [엔티티 매니저 생성](#org6f07af6)
        2.  [트랜잭션 관리](#org740277f)
        3.  [비즈니스 로직](#org3b9a43c)


<a id="org94804b8"></a>

# JPA 시작


<a id="orgb6a1fb3"></a>

## 객체 매핑 시작

-   @Entity  
    이 클래스를 테이블과 매핑한다고 JPA에게 알려준다. 이 클래스를 엔티티 클래스라 한다.
-   @Table  
    엔티티 클래스에 매핑할 테이블 정보를 알려준다.  
    이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑한다.
-   @Id  
    엔티티 클래스의 필드를 테이블의 기본 키에 매핑  
    이렇게 @Id가 사용된 필드를 식별자 필드라 한다.
-   @Column  
    필드를 컬럼에 매핑한다.
-   매핑정보가 없는 필드  
    매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑한다.


<a id="org3339df4"></a>

## persistence.xml 설정

    <?xml version="1.0" encoding="UTF-8"?>
    <persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
        <persistence-unit name="jpabook"> <!-- 영속성 유닛 설정, 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛을 등록  -->
            <properties>
                <!-- 필수 속성 -->
                <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/> <!-- JDBC 드라이버 -->
                <property name="javax.persistence.jdbc.user" value="sa"/> <!-- 데이터베이스 접속 아이디 -->
                <property name="javax.persistence.jdbc.password" value=""/> <!-- 데이터베이스 접속 비밀번호-->
                <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/> <!-- 데이터베이스 접속 URL -->
                <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" /> <!-- 데이터베이스 방언 설정 -->
    
                <!-- 옵션 -->
                <property name="hibernate.show_sql" value="true" />
                <property name="hibernate.format_sql" value="true" />
                <property name="hibernate.use_sql_comments" value="true" />
                <property name="hibernate.id.new_generator_mappings" value="true" />
            </properties>
        </persistence-unit>
    
    </persistence>

-   하이버네이트 속성  
    hibernate.dialect: 데이터베이스 방언 설정


<a id="orgdcf3cbc"></a>

### 데이터베이스 방언

JPA는 특정 데이터베이스에 종속적이지 않은 기술이다. 따라서 다른 데이터베이스로 손쉽게 교체할 수 있다.  
하지만 각 데이터베이스가 제공하는 SQL 문법과 함수가 조금씩 다르다는 문제점이 있다.  

-   데이터 타입
-   다른 함수명
-   페이징 처리

JPA 구현체들은 이런 문제를 해결하려고 다양한 데이터베이스 방언 클래스를 제공  

-   H2: org.hibernate.dialect.H2Dialect
-   오라클: org.hibernate.dialect.Oracle10gDialect
-   MySQL: org.hibernate.dialect.MySQL5InnoDBDialect

![img](/assets/img/JPA_시작/2021-04-19_22-11-56_99174F4F5B477E830D.png)  


<a id="orgc9d1515"></a>

## 애플리케이션 개발

    public class JpaMain {
    
        public static void main(String[] args) {
    
            //엔티티 매니저 팩토리 생성
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
            EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
    
            EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
            try {
    
                tx.begin(); //트랜잭션 시작
                logic(em);  //비즈니스 로직
                tx.commit();//트랜잭션 커밋
    
            } catch (Exception e) {
                e.printStackTrace();
                tx.rollback(); //트랜잭션 롤백
            } finally {
                em.close(); //엔티티 매니저 종료
            }
    
            emf.close(); //엔티티 매니저 팩토리 종료
        }
    
        public static void logic(EntityManager em) {..}
    }


<a id="org6f07af6"></a>

### 엔티티 매니저 생성

![img](/assets/img/JPA_시작/2021-04-19_22-17-16_9990C7455B477EFE37.png)  

-   엔티티 매니저 팩토리 생성  
    persistence.xml에서 jpabook인 영속성 유닛(persistence-unit)을 찾아서 엔티티 매니저 팩토리를 생성한다.
-   엔티티 매니저 생성 / 종료  
    엔티티 매니저 생성과 종료는 위의 코드를 통해 확인


<a id="org740277f"></a>

### 트랜잭션 관리

    EntityTransaction tx = em.getTransaction(); // 트랜잭션 API
    try {
      tx.begin();
      logic(em);
      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    }


<a id="org3b9a43c"></a>

### 비즈니스 로직

    public static void logic(EntityManager em) {
    
        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);
    
        //등록
        em.persist(member);
    
        //수정
        member.setAge(20);
    
        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());
    
        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());
    
        //삭제
        em.remove(member);
    }