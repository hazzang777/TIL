# Bulk Insert In Mysql  

## INTRO

jpa에서는 ID 생성 전략에는 총 4가지가 있다.

1. 직접 할당 : 기본키를 애플리케이션에서 직접 할당해주는 것
2. 자동 생성
    1. IDENTITY: 기본 키 생성을 데이터베이스에게 맡김 (ex: Mysql,,,)
    2. SEQUENCE: 데이터베이스의 시퀀스 오브젝트를 사용하여 기본키를 할당 (ex: Oracle,,,)
    3. TABLE: 기본키를 관리하는 테이블을 만들어 관리

그렇다면 우리가 개발을 하다보면 대량의 엔티티들을 `insert` 하는 경우가 생기게 된다.

하지만 만약 우리가 Mysql을 사용하고 거기에서 IDENTITY 키 생성 전략을 이용하고 JPA를 사용한다면 대량의 인서트 작업인 `bulk insert` 를 사용하는 방법은 없게 된다.

### IDENTITY 전략에서 `batch insert` 를 지원하지 않는 이유

[Why does Hibernate disable INSERT batching when using an IDENTITY identifier generator](https://stackoverflow.com/questions/27697810/why-does-hibernate-disable-insert-batching-when-using-an-identity-identifier-gen)

`IDENTITY` 전략을 사용하면  int 나 bigint 컬럼을 자동으로 증가할 수 있게 해줌

이러한 증가 프로세스는 현재 실행중인 트랜잭션 밖에서 발생하는 일이기 때문에 만약 롤백이 된다면 이미 증가되서 할당된 값들은 버려질 수 있음 (여기서 값 격차가 발생할 수 있음)

증가 프로세스는 데이터베이스 내부 경량 `locking mechanism` 을 사용하기 때문에 매우 효율적이다.

이 증가 프로세스는 유일한 단점이 있는데, INSERT 문이 실행되기 전에는 새롭게 할당된 값을 알수 없는 것이다. 이러한 제한은 하이버네이트에서 채택한 `transactional write-behind` 전략을 방해한다.

이러한 이유 때문에, 하이버네이트는 `IDENTITY` 전략을 사용하는 엔티티에 대해서 `JDBC 배치 지원을 비활성화함.`

## 의문

<aside>
💡 그렇다면 Mysql에서 키생성 전략을 Sequence or Table 전략을 사용하면 되지 않을까?

</aside>

우선, Mysql은 키 생성 전략중 하나인 `SEQUENCE` 전략을 지원하지 않는다.

그리고 각 개발을 하다보면 테이블이 많다 근데 `TABLE` 을 전략을 사용한다면 수많은 테이블 마다 

키 관리하는 테이블을 추가로 만드는 것은 많은 부담을 안게 된다.

먼저 JPA에서 기본으로 제공하는 `saveAll()` 을 이용하여 대량으로 인서트할 시 어떻게 동작하는지 확인해보자.

## JpaRepository의 saveAll() 메서드

```
테스트 환경

Spring Boot
Lombok
Spring Data JPA
DB: h2
dialect: Mysql
```

***saveAll()*** 

```java
@Transactional
	public <S extends T> List<S> saveAll(Iterable<S> entities) {

		Assert.notNull(entities, "The given Iterable of entities not be null!");

		List<S> result = new ArrayList<S>();

		for (S entity : entities) {
			result.add(save(entity));
		}

		return result;
	}
```

- `saveAll`도 결국 bulk insert 를 사용하는 것은 아니다
    - identity를 사용하면 save 함과 동시에 flush를 해서 바로 1차 캐시에 save된 엔티티를 저장하게 된다
    

Dummy.class

```java
@Entity
@Table(name = "dummy")
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Dummy {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    public static List<Dummy> createDummies() {
        return IntStream.range(1, 100)
                .mapToObj(i -> Dummy.builder()
                        .title("title " + i)
                        .build())
                .collect(Collectors.toList());
    }
}
```

DummyServiceImpl.class

```java
@Service
@RequiredArgsConstructor
public class DummyServiceImpl implements DummyService{

    private final DummyRepository dummyRepository;

    @Override
    public void createDummies() {
        List<Dummy> dummies = Dummy.createDummies();
        dummyRepository.saveAll(dummies);
    }
}
```

🤩 먼저 기본 IDENTITY 전략에서는

100개의 데이터를 Jpa Repository에서 제공해주는 기본 메소드 `saveAll()` 을 이용하여

어떻게 동작하는지 확인해보자

![Untitled](https://user-images.githubusercontent.com/68279162/188263829-ac965ce4-0c59-4d8a-b242-9b9d3e2dbabe.png)


기본키를 생성하는 것을 IDENTITY 전략은 데이터베이스에 위임하는 것이기 때문에 어플리케이션 계층에서는 알지 못해 일일이 List 개수마다 일일히 DB에 접근하는 것을 확인할 수 있다 

(🥲 정말 비효율적이다.)

그렇다면 어떻게 하면 한번에 `bulk insert` 를 할 수 있을까?

## JDBC TEMPLATE 이용하여 BULK INSERT

기본적으로 IDENTITY 전략으로는 bulk insert가 불가능하기 때문에

😀 직접 기본키를 세팅해주어야 한다.

Dummy.class

```java
@Entity
@Table(name = "dummy")
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dummy {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    public static List<Dummy> createDummies(Long startId) {
        return IntStream.range(1, 100)
                .mapToObj(i -> Dummy.builder()
                        .id(startId + i)
                        .title("title " + i)
                        .build())
                .collect(Collectors.toList());
    }
}
```

BulkRepository.class

```java
@Repository
@RequiredArgsConstructor
public class BulkRepository {

    private final JdbcTemplate jdbcTemplate;

    public void saveAll(List<Dummy> dummies) { // 1
					// 2
        jdbcTemplate.batchUpdate("insert into dummy(id, title) " +
                        "values (?, ?)", 
                new BatchPreparedStatementSetter() { //3
                    @Override
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        ps.setLong(1, dummies.get(i).getId());
                        ps.setString(2, dummies.get(i).getTitle());
                    }

                    @Override
                    public int getBatchSize() {
                        return dummies.size();
                    }
                });
    }
}
```

***설명***

1.  대량 인서트할 리스트 데이터를 매개변수로 받는다.
2.  jdbcTempate의 `batchUpdate()` 를 이용하여 실행할 쿼리를 작성해준다.
3.  그리고 인수(`?`)로 들어가는 곳에 값을 채워주기 위해 new BatchPreparedStatementSetter를 구현해주기만 하면 된다.
4. 한번에 인서트문으로 `bulk insert` 가 가능하게 된다.

![Untitled 1](https://user-images.githubusercontent.com/68279162/188263826-fdc4d627-fe4c-4fa9-ba12-dc3de057e231.png)

이렇게 실행해준다면 데이터가 잘들어간 것을 확인할 수 있다.

### 시간 비교

1. JpaRepository.saveAll() :  69 밀리초

![Untitled 2](https://user-images.githubusercontent.com/68279162/188263819-852c67c6-b5c2-4fa2-98ee-5e2a75908fad.png)

1. BulkRepository.saveAll() : 45 밀리초

![Untitled 3](https://user-images.githubusercontent.com/68279162/188263815-dd2636a9-f5a6-4e6b-99ed-3dd007ee2456.png)

100개 밖에 안되는 인서트에서도 시간차이가 약 두배 정도 나는 것을 확인할 수 있다.