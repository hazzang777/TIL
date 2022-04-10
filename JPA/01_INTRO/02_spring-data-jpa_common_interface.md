# 02. Spring-data-jpa : 공통 인터페이스

## INTRO
* Spring data jpa 는 공통 JPA 인터페이스 기능을 제공한다.
* @Repository 생략 가능
* 스프링 데이터 JPA가 구현 클래스를 대신 생성


## 1. 공통 인터페이스 설정
```kotlin
@Configuration
@EnableJpaRepositories(basePackages = "리포지토리 패키지")
class Appconfig{}
```
* 스프링 부트 사용하면 생략 가능하다.  
* 스프링 부트 사용시 @SpringBootApplication 해당 패키지와 하위 패키지를 인식

```kotlin
interface MemberRepository : JpaRepository<Member, Long>
```

😀&nbsp; 스프링 데이터 JPA에서 제공하는 인터페이스를 상속하여 Repository를 만든다.  

*  Spring Data Jpa가 확인한 후 JpaRepository를 상속받은 MemberRepository의 구현체를 만들어준다.

😆&nbsp; 확인하기)
``` kotlin
        println("memberRepository = ${memberRepository.javaClass}")
```
        출력 결과 ->    memberRepository = class com.sun.proxy.$Proxy117   
    

## 2. 공통 인터페이스 분석
* JpaRepository 인터페이스 : 공통 CRUD 제공
* 제네릭은 <엔티티 타입, 식별자 타입> 설정

``` kotlin
interface TeamRepository : JpaRepository<Team, Long>
```

### 2.1 공통 인터페이스 구조
<----&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;스프링 데이터&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---->&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <---- 스프링 데이터 JPA ---->  
Repository <- CrudRepository <- PagingAndSortingRepository <- JpaRepository

### 2.2 주요 메서드
1. save(S) : 새 엔티티 저장 및 기존 엔티티는 병합
2. delete(T) : 엔티티 삭제, EntityManager.remove() 호출
3. findById(ID) : 엔티티 조회 한다. EntityManager.find() 호출
4. getOne(ID) : 엔티티를 프록시로 조회. EntityManager.getReference() 호출
5. findAll(...) : 모든 엔티티 조회. 정렬 및 페이징 조건 파라미터로 사용 가능
