week 4
# 회원 웹 기능( 웹 MVC 개발 )

</br>

Menu

 - [홈 화면 추가](#홈-화면-추가)
 - [회원 등록 1](#회원-등록(Get))
 - [회원 등록-2](#회원-등록(Post))
 - [회원 조회](#회원-조회)

</br></br>

## 홈 화면 추가

</br>

![5](https://user-images.githubusercontent.com/40445477/94328895-464eec00-fff1-11ea-988c-0b02e25c31b9.PNG)

</br>


HomeController
```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home(){
        return "home";
    }
}
```
</br>

home.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<body>

<div class ="container">
    <div>
        <h1>Hello Spring</h1>
        <p>회원 기능</p>
        <p>
            <a href="/members/new">회원 가입</a>
            <a href="/members">회원 목록</a>
        </p>
    </div>
</div>

</body>
</html>
```
</br>
</br>
</br>

## 회원 등록(Get)


![6](https://user-images.githubusercontent.com/40445477/94328899-5070ea80-fff1-11ea-9887-14bd9f27a675.PNG)


- Get Mapping

```java
@Controller
public class MemberController {  // Spring Container가 관리를 한다.
    private final MemberService memberService;


    public MemberController(MemberService memberService){
        this.memberService=memberService;
    }

    @GetMapping("/members/new")
    public String createForm(){
        return "members/createMemberForm";
    }
}
```
</br></br></br></br>




## 회원 등록(Post)

 - Post Mapping



![3](https://user-images.githubusercontent.com/40445477/94328805-86fa3580-fff0-11ea-95bc-9aafe35b75b6.PNG)


```java

 @PostMapping("/members/new")
    public String create(MemberForm form){ // post 방식 : 데이터를 form에 넣어서 이동
        //MemberForm 클래스에 있는 name에 값을 setName을 통해 넣어줌
        Member member = new Member();
        member.setName(form.getName()); // form 에 있는 name을 꺼내옴

        memberService.join(member); //회원 가입

        return "redirect:/"; //홈 화면으로
    }
```
</br></br></br>


- get은 일반 호출 / post는 action해서 데이터(name) 넘겨줌


createMemberForm.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <form action="/members/new" method="post">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" placeholder="이름을 입력하세
요">
        </div>
        <button type="submit">등록</button>
    </form>
</div> <!-- /container -->
</body>
</html>
```

</br>
</br>
</br></br>



## 회원 조회

![4](https://user-images.githubusercontent.com/40445477/94328808-89f52600-fff0-11ea-8e43-ca42d34d7d2f.PNG)


</br>

```java
    @GetMapping("/members")
    public String list(Model model){ //회원 정보 보기
        List<Member> members = memberService.findMembers(); //멤버를 다 가져옴
        model.addAttribute("members", members); //list를 model에 담아서 viewtemplate에 넘김
        return "members/memberList";
    }
```
</br>

memberList.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
 <div>
 <table>
 <thead>
 <tr>
 <th>#</th>
 <th>이름</th>
 </tr>
 </thead>
 <tbody>
 <tr th:each="member : ${members}">
 <td th:text="${member.id}"></td>
 <td th:text="${member.name}"></td>
 </tr>
 </tbody>
 </table>
 </div>
</div> <!-- /container -->
</body>
</html>
```

메모리가 없어지기 때문에 db관리하는 방법을 다음시간에 배우자 !

</br></br>

ThymeLeaf 반복문 처리 방법 !
https://ifuwanna.tistory.com/200
