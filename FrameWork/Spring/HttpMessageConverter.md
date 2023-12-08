## HttpMessageConverter이란

웹 브라우저 같은 클라이언트에서 보여지는 HTML 컨텐츠 렌더링(Rendering)되는 방식은 크게 두 가지입니다.

하나는 서버에서 데이터를 포함한 HTML을 만들어서 이 자체를 한번에 클라이언트쪽으로 주는 방식인 서버 사이드 렌더링이고,(SSR)

하나는 서버가 요청을 받으면 클라이언트에 HTML과 JS를 보낸다. 클라이언트를 그걸 받아 렌더링을 하는게 클라이언트 사이드 렌더링이다.(CSR)

HttpMessageConverter는 CSR을 위해 필요한 데이터를 직렬화,역직렬화 해야할때 사용합니다.

예제 코드

```java
@PostMapping
    public ResponseEntity postMember(@Valid @RequestBody MemberDto.Post requestBody) {
        Member member = mapper.memberPostToMember(requestBody);
        member.setStamp(new Stamp());

        Member createdMember = memberService.createMember(member);
        URI location = UriCreator.createUri(MEMBER_DEFAULT_URL, createdMember.getMemberId());

        return ResponseEntity.created(location).build();
    }
```

이런 상황에서 객체를 JSON 형태로 바꿔주기 위해 메세지 컨버터를 사용하게 됩니다.

## 직렬화와 역직렬화

데이터 직렬화는 메모리를 디스크에 저장하거나 네트워크 통신이 가능한 상태로 변환하는 것.

데이터 역직렬화는 직렬화와 반대로 디스크에 저장한 데이터를 읽거나, 네트워크 통신으로 받은 데이터를 메모리에 쓸 수 있도록 변환하는것이다.

이런 과정이 필요한 이유는

우리가 사용하는 데이터 메모리 구조는 크게 2가지로 나뉜다.

- 값 형식 데이터 :  int,float, char 등 값 형식 데이터. 스택에 메모리가 쌓이고 직접 접근 가능.
- 참조 형식 데이터 : 객체 같은 참조 형식 변수를 선언하면 힙에 메모리가 할당되고, 스택은 이 힙 메모리를 참조하는 구조로 되어 있다.

이중에 값 형식 데이터가 디스크에 저장하거나 통신 할때 쓰인다.

<aside>
❓ 왜 참조 형식 데이터를 사용할수 없을까?
예를 들면. 객체 A를 만들고 주소값이 0x22라고 가정하고, 이값을 파일에 포함해서 저장했을때 이후 프로그램을 종료하고 다시 실행해서 주소값 0x22을 가져 와도 기존 A 객체의 데이터를 가져올수 없다. 
프로그램이 종료되면 기존에 할당된 메모리는 해제되도 없어지기 때문입니다.
네트워크 통신도 각 PC마다 사용하고 있는 메모리 공간 주소는 전혀 다르다. 그러므로 내가 다른 PC로 전송한 A객체 데이터는 의미가 없다. 데이터 받은 PC의 0x22 주소는 다른값이 존재하기 때문에.

</aside>

그래서 직렬화는 왜 하는가?

직렬화를 하게 되면 각 주소값이 가지는 데이터를 전부 끌어 모아서 값 형식 데이터로 변환 해준다. 직렬화가 된 데이터는 언어에 따라서 text, binary 등의 형태가 되는데,
이런 형태가 됐을때 저장하거나 통신할 때 파싱이 가능한 유의미한 데이터가 된다.
즉, 하는 이유는 사용하고 있는 데이터를 파일 저장 혹은 데이터 통신에서 파싱할 수 있는 유의미한 데이터를 만들기 위함이다.

쉽게 말하면 객체를 저장, 전송 할수 있는 형태로 변환.

역직렬화는 데이터를 받아서 객체로 변환.

## 데이터 직렬화의 종류

1. CSV, XML, JSON 직렬화
    - 사람이 읽을 수 있는 형태
    - 저장 공간의 효율성이 떨어지고, 파싱하는 시간이 오래 걸림.
    - 데이터의 양이 적을 때 주로 사용
    - 요즘은 JSON 형태로 많이 직렬화함.
    - 모든 시스템에서 사용 가능.
2. Binary 직렬화
    - 사람이 읽을수 없는 형태.
    - 저장 공간을 효율적으로 사용할 수 있고, 파싱하는 시간이 빠름
    - 데이터 양이 많을때 주로 사용함.
    - 모든 시스템에서 사용 가능
    - 예제 : 프로토콜 버퍼,apache Avro 등
3. Java 직렬화
    - Java 시스템간의 데이터 교환이 필요할 때 사용.

여기서 보통 Java 직렬화가 많이 나오지만 스프링 이기 때문에 생략 하겠습니다.

## HttpMessageConverter

## 정의

## 동작과정

@ResponseBody를 사용할때 동작 과정 

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/a4ac5f26-d4d8-429c-961d-7392b416f7d3)

동작하는 과정은

@ResponseBody를 사용하게 되면 ViewResolver 대신에 HttpMessageConverter가 동작하게 된다.

String 같은 기본 문자는 StringHttpMessageConverter를 사용하고, 객체는 MappingJackson2HttpMessageConverter 사용한다.

이외에도 각자 맞는 컨버터들이 등록되어 있어서 이것들을 사용하게 된다.

응답은 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보를 조합해서 컨버터가 선택된다.

@ResponseBody의 경우는 이 위에 컨버터를 사용한다고 했지만, 정확하게 말하면

HTTP 요청은 `@**RequestBody`, HttpEntity(RequestEntity)**

HTTP 응답은 `@ResponseBody`, HttpEntity(ResponseEntity)

```java
public interface HttpMessageConverter<T> {
//주석 다 지움

	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	List<MediaType> getSupportedMediaTypes();

	default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
		return (canRead(clazz, null) || canWrite(clazz, null) ?
				getSupportedMediaTypes() : Collections.emptyList());
	}

	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```

canRead,canWrite를 통해 읽기,쓰기 여부를 확인한다. 컨버터는 요청과 응답 두 경우 모두 사용하기 때문이다. 

그이후엔 read, write 메소드를 통해 읽고 쓰기 동작을 실행한다.

그리고 클래스와 MediaType을 체크 해서 어떤 메세지 컨버터를 사용할지 결정하는데, 우선순위 대로 메세지 컨버터 사용 검토하면서, 조건에 만족하지 못하면 다음으로 넘어간다.

대표적인 컨버터는 이렇게 있는데 (우선순위대로 나열)

- ByteArrayHttpMessageConverter
- StringHttpMessageConverter
- MappingJackson2HttpMessageConverter

**ByteArrayHttpMessageConverter**

byte[] 데이터를 처리하고. 클래스 타입은 byte[] 이며 MediaType은 */*이다.

**StringHttpMessageConverter**

String 문자로 데이터 처리하고, 클래스 타입은 String, MediaType은 */*이다.

**MappingJackson2HttpMessageConverter**

json으로 데이터 처리하고, 클래스 타입은 객체 또는 HashMap이며, MediaType은 application/json 관련이다.

<aside>
❓ 참고로 */*는 모든 미디어 타입을 말합니다.

</aside>

## 적용

그러면 이 메세지 컨버터는 어디서 적용이 될까?

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/c6fc1442-80d2-4e41-b068-5793108c1893)

DispatcherServlet은 이런 형태로 동작을한다.

@RequestMapping 어노테이션이 핸들러 매핑과 핸들러 어댑터에 대한 정보를 주게 되는데 컨버터는 이 핸들러 어댑터를 수행하는 과정에서 이루어진다.

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/ce55fbb1-e727-4695-bc74-7f7da22f48e4)

어댑터에서 이 핸들러를 실행할때 ArgumentResolver를 이용해 파라미터를 넘겨준다.

전에 ArgumentResolver를 정리하면서 했던 것의 핵심은 @Pathvariable, @RequestBody 등이 이걸 통해 데이터를 넘겨주었던것이었는데,

그 과정에선 애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdaptor`는 **ArgumentResolver**를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. **ArgumentResolver**가 필요한 파리미터의 값을 반환해주면 핸들러(컨트롤러)를 호출하면서 값을 넘긴다.

컨버터는 `HandlerMethodArgumentResolver`와 `HandlerMethodReturnValueHandler`의 과정에서 이용을 하는 것이었다.

이런 과정이 흐르게 된다.

`HandlerMethodArgumentResolver` : 줄여서**ArgumentResolver** 이라고 부르고,

`HandlerMethodReturnValueHandler` : 줄여서 **ReturnValueHandle** 이라고 부른다.

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/bbdf7217-ecd3-45db-afe8-dfb614feb7f9)

# 참고

https://steady-coding.tistory.com/576

https://code-lab1.tistory.com/289

****HttpMessageConverter****

https://bepoz-study-diary.tistory.com/374

[https://velog.io/@woply/spring-메시지-바디의-데이터-처리를-담당하는-HttpMessageConverter](https://velog.io/@woply/spring-%EB%A9%94%EC%8B%9C%EC%A7%80-%EB%B0%94%EB%94%94%EC%9D%98-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%B2%98%EB%A6%AC%EB%A5%BC-%EB%8B%B4%EB%8B%B9%ED%95%98%EB%8A%94-HttpMessageConverter)
