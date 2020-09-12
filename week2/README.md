# week 2(회원 관리 예제)

</br></br>

 Menu

 - [회원 도메인 만들기](#회원-도메인-만들기)
 - [회원 레포지토리 만들기](#회원-레포지토리-만들기)
 - [회원 레포지토리 테스트 케이스 만들기](#회원-레포지토리-테스트-케이스-만들기)
 - [회원 서비스 개발](#회원-서비스-개발)
 - [회원 서비스 테스트 만들기](#회원-서비스-테스트-만들기)
</br></br>



## 회원 도메인 만들기
```java
//요구사항 중에 두가지 식별자가 필요! (id 와 name)
public class Member {
    private Long id;//시스템이 저장하는 id
    private String name;//고객의 이름

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```
</br></br>


## 회원 레포지토리 만들기

- 인터페이스 
```java
//회원 레포지토리 인터페이스
public interface MemberRepository {
    Member save(Member member);//저장소에 저장

    //Optional은 java8에 있는 기능인데  null을 감싸는 역할을 한다. null 값이 있어도 사용할 수 있도록 함
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();//전체 반환
}
```
</br>


- 인터페이스를 활용한 레포지토리 만들기
```java

public class MemoryMemberRepository implements MemberRepository{

    //Map HashMap 사용법 알아보기
    private static Map<Long,Member> store = new HashMap<>();
    private static long sequence =0L;

    @Override
    public Member save(Member member){
        member.setId(++sequence); //id 1 증가
        store.put(member.getId(), member); // store저장
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id)); //id값을 꺼내오는데 null이 반환될 가능성이 있음으로 Optional로 감싼다.
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream().filter(member -> member.getName().equals(name)).findAny();
        // getName이 파라미터로 전달받은 name과 같은지 비교 (같은경우 findAny로 반환함)
        // 없으면 null로 반환
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values()); //List에 맞춰서 값을 넣어줌
    }

    public void clearStore(){
        store.clear();
    }
}

```
</br>
</br>


## 회원 레포지토리 테스트 케이스 만들기
```java
class MemoryMemberRepositoryTest {

     MemoryMemberRepository repository = new MemoryMemberRepository();


     //Test 끝날 때마다 코드를 지워주는 함수
     @AfterEach
     public void afterEach(){
          repository.clearStore();
     }

     @Test
     public void save(){
          Member member = new Member();
          member.setName("spring");

          repository.save(member);

          Member result = repository.findById(member.getId()).get();
//          System.out.println("result = " +(result == member));
//          Assertions.assertEquals(member, result);
          assertThat(member).isEqualTo(result); //출력을 안하고 비교할 수 있다.
     }

     @Test
     public void findByName(){
          Member member1 = new Member();
          member1.setName("spring1");
          repository.save(member1);

          Member member2 = new Member();
          member2.setName("spring2");
          repository.save(member2);

          Member result = repository.findByName("spring1").get();
          //

          assertThat(result).isEqualTo(member1);
     }

     @Test
     public void findAll(){
          Member member1 =new Member();
          member1.setName("spring1");
          repository.save(member1);

          Member member2 =new Member();
          member2.setName("spring2");
          repository.save(member2);

          List<Member> result = repository.findAll();

          assertThat(result.size()).isEqualTo(2);
     }


}
```
</br>
</br>


## 회원 서비스 개발
```java
public class MemberService {
    //memberRepository와 memory~차이 ,optional찾아보기

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository){
        this.memberRepository=memberRepository;
    }

        //회원 가입
            public Long join(Member member){

        //같은 이름이 있는 중복 회원 x
                validateDuplicateMember(member); // 중복 회원 검증
                memberRepository.save(member);
                return member.getId();
            }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName()).ifPresent(m->{
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });
    }

    //전체 회원 조회
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }
}
```
</br>
</br>


## 회원 서비스 테스트 만들기

```java
class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;


    @BeforeEach
    public void beforeEach(){
        //외부에서 memberRepository를 넣어줌 .DI라고 부름 
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach(){
        memberRepository.clearStore();

    }
    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("Hello");

        //when
        Long saveId = memberService.join(member);
        //then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }


    @Test
    public void 중복_회원_예외(){
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        //when
        memberService.join(member1);
        IllegalStateException e=assertThrows(IllegalStateException.class,()->memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

        /*
        try {
            memberService.join(member2); //예외 발생하게 됨
            fail();
        }catch (IllegalStateException e){

        }
        */


        //then
    }

    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```
