## 개념 정리

### SOP,CORS

Same-origin policy

CORS를 알려면 SOP가 뭔지 알아야 합니다.

SOP란 같은 Origin에만 요청을 보낼 수 있게 제한하는 보안 정책을 의미한다.

Origin 구성은

- URI Schema(http,https)
- Hostname(localhost,naver.com)
- Prot(80,8080)

이중에 하나라도 구성이 다르면 SOP 정책에 걸려서 요청을 보낼수 없다.

예시로 http://www.example.com/dir/page.html에 요청을 보낼 때

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/79d4a4bf-21d3-445a-a591-aa4a5fde2d61)

CORS는

서로 다른 Origin끼리 요청을 주고 받을 수 있게 정해둔 표준이라고 할 수 있다.

## 간단하게 동작 원리

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/fd69358d-5af4-4844-a944-cfe2a648510f)

1. 처음에 Get요청인지 Post요청인지 파악한다.
2. Content-Type과 Custom HTTP Header를 파악한다.
3. OPTIONS 요청을 통해 서버가 적절한 Access-Control를 가졌는지 확인한다.
4. 만약 적절한 Access-Control를 가졌으면 실제 XHR을 트리거한다.
5. 적절하지 못한 Access-Control을 가졌으면 Error를 발생시킨다.

이 과정에서 적절한 대처를 못했을때 발생하는것이 CORS 에러인것이다.

## 해결 방법

### Controller에 @CrossOrigin 추가하기

```java
@CrossOrigin(origins = "http://localhost:8080") // 추가
@RestController
public class MemberController {

  @GetMapping("/hello")
  public String hello() {
    return "안녕하세요?";
  }
}
```

이 어노테이션은 Spring에서 CORS를 적용할 수 있게 만든 어노테이션이다.

Origin 속성으로 도메인을 설정하면 해당 도메인은 다른 Origin을 가지고 있지만, 요청을 주고 받을 수 있게 된다.

### WebConfig에 CORS 설정하기

특정 도메인으로 오는 요청에는 모두 CORS를 적용하고 싶을때 WebConfig를 설정하면 된다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/**")
        .allowedOrigins("http://localhost:8080");
  }
}
```

WebMvcConfigurer를 상속 받아서 addCorsMappings를  Override하면 CORS 관련 설정이 가능합니다. 

이 코드는 http://localhost:8080에서 오는 요청은 어느 url이든 CORS를 적용 해주도록 설정해주는것이다.

이렇게하면 Controller에 @CrossOrigin을 안써도 된다.

### Proxy 만들기

서버에서 CORS 위반 한다고 서버에서 응답을 막는게 아니라 

1. 클라이언트 - 서버에서 호출
2. 서버에서 응답
3. 브라우저에서 CORS 정책을 위반하면 응답 파기

이런 과정으로 흘러간다. 

즉, 요청을 보내는 쪽에서 프록시 서버를 만들어 간접적으로 전달하면 응답을 받을 수 있다.

```java
@Controller
public class CorsController {

  @GetMapping("/api/proxy")
  @ResponseBody
  public String proxy() {
    String url = "http://localhost:1000/hello";

    RestTemplate restTemplate = new RestTemplate();
    return restTemplate.getForObject(url, String.class);
  }
}
```

클라이언트와 같은 포트에서 api를 만든 후에, RestTemplate의 getForObject(요청 url, 반환 클래스)를 이용해 서버에 요청을 보내고 응답을 리턴해준다.

이 방법은 클라이언트에서 프록시 서버를 필요할때 사용하는게 좋다.

# 참고

https://shinsunyoung.tistory.com/86

원리 https://evan-moon.github.io/2020/05/21/about-cors/

https://wonit.tistory.com/572
