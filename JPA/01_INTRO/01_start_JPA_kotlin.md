# 01. JPA start with Kotlin

## 1. 필요 플러그인
    1.1 no-arg : Reflection 관련
    1.2 all-open : Proxy 관련

example)
``` kotlin
    
    plugins{
        kotlin("plugin.jpa") version ...
        id ("org.jetbrains.kotlin.plugin.allopen") version ...
        id ("org.jetbrains.kotlin.plugin.noarg") version ...
    }
    
    
    // (1)
    noArg{ 
        annotation("javax.persistence.Entity")
    }
    // (2)
    allOpen{
        annotation("java.persistence.Entity")  
        annotation("java.persistence.MappedSuperclass")
        annotation("java.persistence.Embeddable")
    }
```

Description)    
(1) Entity 어노테이션이 붙은 곳에 기본 생성자 생성  
   - Hibernate는 리플렉션으로 엔티티 객체를 생성하기 때문에 no-arg 생성자를 가지고 있어야한다. (public or protected)  

(2) Entity, MappedSuperclass, Embaddable 어노테이션 붙은 클래스를 open 클래스로 만들어준다.  
   - JPA 에서는 런타임 환경 시 엔티티에 프록시 객체를 만들어서 Lazy-loading 기능을 사용하므로 final은 금지 



## 2. 주의할 점
``` kotlin
    import javax.persistence.*

    @Entity
    class Member( // (1)
        @Id
        @GeneratedValue
        var id : Long ? = null,

        @Column
        var name : String
            protected set   // (2)
    ){
        fun changeName(name : String){
            this.name = name
        }
    }

```

Description)  
(1) 코틀린에서 Entity를 생성할 때 data class로 만드는 것을 권장 하지 않는다. (일반 class로 생성)  
   - data class로 생성하게 되면 copy(), toString(), equals() 등 여러 코드들이 자동 생성된다.  
   - toString()로 인해 양방향 연관관계를 맺는 Entity들 사이에서 순환 참조 문제가 발생한다.   

(2) 기본적으로 Entity는 getter에 열려있고, setter에 닫혀있어야 한다.
   - setter가 모든 프로퍼티에 열려 있다면, 의도치 않은 상황에서의 변경 막기가 힘들며, 의도치 않은 변경 상황에서 추적하기 힘들다.
   - private set을 하는 경우 open property에서는 지정할 수 없으므로 protected 레벨로 외부에서 접근 제한되도록 지정.  



참조)  
[Spring Boot + Kotlin + JPA 적용하기 Entity 생성시 생각해볼 점들](https://blog.junu.dev/37)
