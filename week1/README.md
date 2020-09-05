# WEEK 1 (코드 구현 및 설명)


</br></br>

 Menu

 - [정적 컨텐츠](정적-컨텐츠)
 - [MVC와 템플릿 엔진](MVC와-템플릿-엔진)
 - [API](API)

</br></br>


## 정적 컨텐츠



</br>

&nbsp; 원하는 html파일을 정적 파일로 웹브라우저에게 넘긴다.  
(스프링 부트에서 해당 파일을 받았을 때 그와 관련된 컨트롤러가 없을 경우 바로 html파일을 웹 브라우저에게 넘긴다.)

</br>

* 실행하기 -> http://localhost:8080/hello-static.html

> resources -> static ->hello-static.html

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>static content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
SeungWon의 정적 컨텐츠입니다.
</body>
</html>
```
</br>
---------------------------------------------------------------------------
</br></br>

## MVC와 템플릿 엔진

</br>

MVC(model,view,controller)는 Controller에게 전달되어 return값과 model에서 받은 값들을 viewReslover에게 넘겨준다.  
viewResolover는 return값에 해당된 html을 Thymeleaf 템플릿 엔진에게 넘겨주고 그것을 변환하여 웹 브라우저에게 전달한다.

</br></br>

### 1. model을 통해 key와 value를 해당 html로 넘겨준다.
- 실행하기 -> http://localhost:8080/hello

```java
public class HelloController {

    @GetMapping("hello") // localhost:8080/hello를 주소창에 입력
    public String hello(Model model)
    {
        model.addAttribute("data","hello SeungWon");
        // model를 통해 value값인 hello SeungWon 를 넘겨줌
        return "hello";
        // hello.html로 이동함
    }

}

```
> resources -> templates -> hello.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```
</br></br>
### 2.파라미터를 전달받아 전달받은 값을 넘겨준다.

 * 실행하기-> http://localhost:8080/hello-mvc?name=seungwon

```java
public class HelloController {

 @GetMapping("hello-mvc")  // localhost:8080/hello-mvc?name=seungwon
    public String helloMvc(@RequestParam("name") String name,Model model)
    {
        model.addAttribute("name",name); //name 위치에 seungwon이 들어감.
        return "hello-template"; //hello-template.html로 이동
    }
}
```

> resources -> templates -> hello-templete.html

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">SeungWon's page is empty !!</p>
</body>
</html>
```
</br>
----------------------------------------------------------------------
</br></br>

## API

</br>

### 1. String 값을 받을 경우 StringConverter를 통해 값을 넘겨준다.(html 과정을 거치지 않음)

* 실행하기 -> http://localhost:8080/hello-string?name=seungwon
```java
public class HelloController {

@GetMapping("hello-string") 
    @ResponseBody //http의 body부에 이 데이터를 직접 넣겠다.
    public String helloString(@RequestParam("name") String name)
    {
        //name = spring이면
        return "hello " + name ; //hello spring으로 바뀜
    }
}
```


</br>

### 2.객체를 받을 경우 JsonConverter를 통해 객체를 넘겨준다.

* 실행하기-> http://localhost:8080/hello-api?name=seungwon

```java
public class HelloController {

 @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name)
    { //객체 생성
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
        //json방식으로 key : value 형태로 나옴
    }

    static class Hello{
        private String name;

        public String getName(){
            return name;
        }
        public void setName(String name){
            this.name=name;
        }
    }
}
```






