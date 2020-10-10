# 스프링 DB 접근 기술

</br>

Menu

- [순수 JDBC](#순수-JDBC)
- [스프링 통합 테스트](#스프링-통합-테스트)
- [스프링 JDBC Template](#스프링-JDBC-Template)
- [JPA](#JPA)
- [스프링 데이터 JPA](#스프링-데이터-JPA)

</br></br>

## 순수 JDBC

</br></br>


- build.gradle

 implementation 'org.springframework.boot:spring-boot-starter-jdbc'  
 
 runtimeOnly 'com.h2database:h2' 를 추가한 후

</br>
JdbcMemberRepository를 만든 후
save, find , close 를 직접 구현해 준다.(까다로움 이후 JPA에서 해결)
</br></br>

- JdbcMemberRepository
```java
public class JdbcMemberRepository implements MemberRepository {
    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;

    }
    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;//결과값
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS); //값을 안넣어도 자동적으로 올라감
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();//db에 실제 quary가 들어감
            rs = pstmt.getGeneratedKeys();//위에 키와 매칭됨

            if (rs.next()) { //값이 있으면 꺼내오기
                member.setId(rs.getLong(1));
            } else {//실패
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally { //마지막 release 해야됨!! 리소스 반환하지 않으면 쌓여서 위험함

            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findById(Long id) {
        String sql = "select * from member where id = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null; //결과 받기
        try {
            conn = getConnection(); //커넥션 가져오고
            pstmt = conn.prepareStatement(sql); //sql넣고
            pstmt.setLong(1, id);
            rs = pstmt.executeQuery(); //조회
            if(rs.next()) {//값이 있으면 멤버 객체를 만들고 반환함
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            } else {
                return Optional.empty();
            }
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public List<Member> findAll() { //전체 반환
        String sql = "select * from member";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            List<Member> members = new ArrayList<>();
            while(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                members.add(member);
            }
            return members;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findByName(String name) {
        String sql = "select * from member where name = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, name);
            rs = pstmt.executeQuery();
            if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            }
            return Optional.empty();
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    private Connection getConnection() {
        //스프링으로 할때 utils를 통해 getConnection해야 함
        return DataSourceUtils.getConnection(dataSource);
    }
    private void close(Connection conn, PreparedStatement pstmt, ResultSet rs)
    {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (pstmt != null) {
                pstmt.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (conn != null) {
                close(conn);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    private void close(Connection conn) throws SQLException {
        //닫을때도 utils를 통해 한다.
        DataSourceUtils.releaseConnection(conn, dataSource);
    }
}
```

이후 SpringConfig에 Jdbc레퍼지토리를 추가해주면 끝이다.
```java
@Configuration
public class SpringConfig {
    private DataSource dataSource;

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource= dataSource;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());

    }

    @Bean
    public MemberRepository memberRepository(){
       // return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);//dataSource를 주입함
    }
}
```
</br>
</br>


### 객체 지향의 원리

인터페이스를 이용해 다형성을 활용한다. 스프링은 이를 스프링컨테이너로 지원해준다  

DI때문에 더 활용 가능 -> 기존 코드에는 손대지 않고 설정만으로 구현 클래스를 변경한다  

</br>
</br>
</br>


## 스프링 통합 테스트
</br>

- 스프링 컨테이너와 DB를 통합한 테스트 활용

@SpringBootTest 와 @Transactional을 주석 처리된 문구로 코드를 통해 알아보자
 
 이 TestCase의 구현체는 SpringConfig에서 가져옴
  
</br></br>

- MemberServiceIntegrationTest
```java
@SpringBootTest
@Transactional
class MemberServiceIntegrationTest {

  //test이기 때문에 이렇게 필드 기반으로 autowired사용하면 됨
  @Autowired MemberService memberService;
  @Autowired MemberRepository memberRepository;

  // afterEach와 BeforeEach를 사용할 필요가 없다.
    // ( db데이터를 다음에 영향을 받지 않기 위해 사용했음)


  /* Transactional을 사용했기 때문 (반복해서 사용해도 위에 Each문을 사용하지 않아도 오류없음)
    testcase에 달면 test 실행할때 이거 먼저 실행하고 테스트 끝나면 rollback을 해줌 
     db에 데이터가 남지 않아서 영향을 주지 않음*/
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
</br>
</br>
</br>


## 스프링 JDBC Template

</br>

위의 JDBC build.gradle환경과 동일하게 맞추면 된다  

JDBC API의 반복적인 코드를 제거해줌!(SQL은 직접 작성해야 됨)  

Template을 사용하면 얼마나 간결하게 변화되는지 아래의 코드를 통해 확인하자

</br>
 JdbcTemplateMemberRepository 를 만든 후 SpringConfig에서 실행
</br>
</br>


- JdbcTemplateMemberRepository
```java

public class JdbcTemplateMemberRepository implements MemberRepository{

    private  final JdbcTemplate jdbcTemplate;


    //dataSource를 주입함
    public JdbcTemplateMemberRepository(DataSource dataSource){
        jdbcTemplate = new JdbcTemplate(dataSource);
    }


    @Override
    public Member save(Member member) {
        //SimpleJdbc를 통해 쿼리 없이 넘길 수 있다!!
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        //excute로 key값을 받고 member에 넣어서 리턴함
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }


    //Template library를 통해 jdbc와 비교해 보면 단 두줄만으로 끝냄
    @Override
    public Optional<Member> findById(Long id) {

        List<Member> result = jdbcTemplate.query("select * from member where id = ?",memberRowMapper(),id);
        return result.stream().findAny(); //리스트로 받아 Optional로 반환
        //Rowmapper를 통해 결과를 매핑시켜줘야 됨
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?",memberRowMapper(),name);
        return result.stream().findAny(); //Optional로 반환

    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member",memberRowMapper());//Optional로 반환

    }


    //resultSet(rs) 결과를 member로 매핑시킨다
    private RowMapper<Member> memberRowMapper(){
        return new RowMapper<Member>() {
            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                Member member =  new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return member;

            }
        };

    }
}

```
</br></br>

- SpringConfig

```java
@Configuration
public class SpringConfig {
    private DataSource dataSource;

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource= dataSource;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());

    }

    @Bean
    public MemberRepository memberRepository(){
       // return new MemoryMemberRepository();
       // return new JdbcMemberRepository(dataSource);//dataSource를 주입함
        return new JdbcTemplateMemberRepository(dataSource);//dataSource를 주입함

    }
}

```
</br>
</br>
</br>


## JPA

</br>
Template을 통해 중복된 코드가 없어지고 간결해졌지만 sql쿼리문은 직접 작성해야 되는 문제가 있다. JPA는 이를 해결해준다.  

단순한 sql을 넘어서 객체 중심으로 패러다임을 전환할 수 있다.
</br>
</br>
</br>

- Member

```java

//요구사항 중에 두가지 식별자가 필요! (id 와 name)

import javax.persistence.*;

//Jpa가 관리하는 entity
@Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;//시스템이 저장하는 id

    //@Column(name="username")
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

- JpaMemberRepository

```java
public class JpaMemberRepository implements MemberRepository{
    private final EntityManager em;

    //JPA를 쓰려면 EntityManger를 주입받아야 함
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
     }

    @Override
    public Member save(Member member) {
        // jpa가 모든걸 다해줌
        em.persist(member);
         return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class,id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = name",Member.class)
                .setParameter("name",name)
                .getResultList();

        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        List<Member> result = em.createQuery("select m from Member m",Member.class)
                .getResultList();
        return result;
    }
}


```
</br></br>

- Springconfig
```java

@Configuration
public class SpringConfig {
  /*  private DataSource dataSource;
    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource= dataSource;
    }
*/
    EntityManager em;

    @Autowired
    public SpringConfig(EntityManager em){
        this.em= em;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());

    }

    @Bean
    public MemberRepository memberRepository(){
        // return new JdbcTemplateMemberRepository(dataSource);
             return new JpaMemberRepository(em);
    }
}
```
</br></br></br>



## 스프링 데이터 JPA
</br>

JPA를 다루고 나서 배워야 함  

인터페이스만으로 개발을 완료할 수 있을 정도로 간단해짐
스프링 jpa가 기본적으로 제공해줌

</br>
스프링 데이터 JPA가 SpringDataJpaMemberRepository를 스프링 빈에 자동으로 등록한 후 SpringConfig에서 inJection하여 사용한다.
</br></br>

- SpringDataJpaMemberRepository
```java
package seung.hellospring.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import seung.hellospring.domain.Member;

import java.util.Optional;

public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>,MemberRepository {
    @Override
    Optional<Member> findByName(String name);
    //메소드 이름만으로도 조회 가능
}

```

</br></br>

- SpringConfig
```java
@Configuration
public class SpringConfig {
    private final MemberRepository memberRepository;

    //인터페이스에 대한 구현체를 만든 후 스프링 빈에 등록함
    //inject
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

   

}
```
