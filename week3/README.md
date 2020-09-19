# week 3(스프링 빈과 의존관계)


## 스프링 빈을 등록하는 방법

</br>

- 컴포넌트 스캔 방식

Annotation으로 Component를 설정한다.

각각의 @Controller,@Service,@Repository 안에 @Component가 있다.

이 Component는 각각 스프링 객체로 등록하게 한다.(스프링 빈에 등록)

Autowired는 각각의 클래스들이 서로 의존관계를 할 수 있게 한다.(DI)
</br></br></br></br>

DI(의존관계)

helloController ----> memberService -----> memberRepository
</br></br></br>

 - helloController
```java
@Controller
public class MemberController {  // Spring Container가 관리를 한다.
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService){
        this.memberService=memberService;
    }
}
```
</br>

 - memberService
```java
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository){
        this.memberRepository=memberRepository;
    }
}
```
</br>

- memberRepository
```java

@Repository
public class MemoryMemberRepository implements MemberRepository{
}

```

memberRepository는 누구한테도 의존하지 않기 때문에 Autowired를 사용할 필요가 없다!

-----------------------------------------------------------------------------

</br></br></br>

- 자바 코드로 직접 등록하기

</br></br>
Controller는 그대로! 나머지는 Config클래스를 만든 후 직접 장성

</br>

상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록해야 한다. 즉 변경이 필요할 때 자바 코드로 직접 등록하는 것이 편하다.

이 예제는 interface로 데이터 저장소를 메모리로 만들었기 때문에 
변경할 가능성이 큼으로 자바 코드로 작성하는 것이 좋다!
</br></br>

- helloController
```java
@Controller
public class MemberController {  // Spring Container가 관리를 한다.
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService){
        this.memberService=memberService;
    }
}
```
</br></br>

 - SpringConfig.Class
```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());

    }

    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
        //만약 다른 레포지토리를 사용하고 싶으면 이 부분만 변경하면 된다! new 변경할Repository()
    }
}
```
</br>
</br>
</br>

---------------------------------------------
</br>

## 개념 정리

</br>

- Spring Container  

Spring FrameWork의 핵심으로 Bean을 관리해주는 역할을 한다.  
Java 코드나 Annotation를 남기면 Container가 Bean 객체를 호출하여 관리한다.


</br>


- DI(의존성 주입)

예를들어 컨트롤러가 서비스를 통해서 회원가입,데이터 조회를 할수 있기 때문에 의존을 해야 한다. 이를 DI(의존성 주입)라고 한다.
</br></br>
장점:  

코드의 재활용성을 줄이고 객체간의 의존성을 없앨 수 있다. 
</br>

방법:

1.생성자 전달  

2.setter 전달  

3.변수 전달

</br>


스프링 컨테이너와 DI에 관한 URL 

https://gmlwjd9405.github.io/2018/11/09/dependency-injection.html
