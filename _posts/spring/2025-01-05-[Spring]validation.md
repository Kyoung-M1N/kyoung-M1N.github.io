---
title: validation
author: kymin
date: 2025-01-05 12:54
categories: [Spring]
tags: [spring, web, java]


---

## 데이터 검증(Validation)

서버로 들어오는 요청에 포함된 데이터는 여러가지 이유로 정해진 형식이 있을 수 있다. 예를 들어 비밀번호의 경우 복잡성을 높이기 위해 영문과 숫자를 포함하여 7자 이상으로 입력해야 하는 등의 조건이 있을 수 있다.

보통 1차적으로 프론트엔드에서 사용자의 입력값을 검증한 뒤에 검증이 끝난 값을 요청에 포함하여 서버로 전달한다.

하지만 프론트엔드에서 적절한 검증이 이루어지지 않거나 웹페이지를 통하지 않고 서버에 요청을 보내는 방식으로 정해진 형식과 다른 데이터가 저장되는 것을 방지하기 위해 백엔드에서도 전달된 데이터의 유효성을 검사하는 과정이 필요하다.

스프링이 제공하는 Validation 라이브러리는 이미 구현된 어노테이션을 추가하거나 정규식을 통해 원하는 검증 조건을 만들어 데이터의 검증 과정을 편리하게 구현할 수 있도록 한다.



### 데이터 검증 조건

서버로 전달되는 데이터가 가질 수 있는 조건에 따라 해당 필드에 어노테이션을 추가하여 데이터에 조건을 지정할 수 있다.

주로 사용되는 어노테이션은 아래와 같다.

- 문자열
  - `@NotNull` : null을 허용하지 않음
  - `@NotBlank` : null과 빈 문자열을 허용하지 않음
  - `@NotEmpty` : null, 빈 문자열과 공백만으로 이루어진 문자열을 허용하지 않음
  - `@Null` : null만 허용
  - `@Size(min = , max = )` : 문자열의 최소 길이와 최대 길이 지정
  - `@Email` : 이메일 형식의 문자열만 허용
  - `@Pattern(regex = )` : 정규식에 맞는 문자열만 허용
- 숫자
  - `@Positive` : 양수만 허용
  - `@PositiveOrZero` : 양수와 0만 허용
  - `@Negative` : 음수만 허용
  - `@NegativeOrZero` : 음수와 0만 허용
  - `@Min()` : 최솟값 지정
  - `@Max()` : 최댓값 지정
- 날짜
  - `@Future` : 현재보다 미래의 날짜만 허용
  - `@FutureOrPresent` : 현재 또는 미래의 날짜만 허용
  - `@past` : 현재보다 과거의 날짜만 허용
  - `@PastOrPresent` : 현재 또는 과거의 날짜만 허용

위의 어노테이션 이외에도 boolean타입의 값이 true인지 검증하는 `@AssertTrue`등 다양한 조건의 어노테이션들이 존재한다.

모든 어노테이션들에는 `message`파라미터를 전달할 수 있으며 이를 통해 예외 처리 시 전달할 메시지를 정의할 수 있다.



### 검증할 데이터 지정

실제 요청에 포함된 데이터를 검증하기 위해서는 검증할 데이터를 정의 하는 필드에 어노테이션을 추가하고, 데이터를 검증할 지점에서 `@Valid` 어노테이션을 추가한다.

- `@RequestBody`

  ```java
  public class RequestDto {
    @Eamil
    private String userId;
    
    @NotEmpty(message = "닉네임은 1자 이상이어야 합니다.")
    private String nickname;
    
    // 대소문자 구분없이 영문과 숫자 포함 7자 이상
    @Pattern(regexp = "^(?=.*[a-zA-Z])(?=.*\\d)[a-zA-Z\\d]{7,}$")
    private String password;
    
    @Positive
    private int age;
    
    @Past
    private Date birthday;
  }
  ```

- `@RequestParam`

  ```java
  public interface Controller {
    @GetMapping(value = "/search")
    ResponseEntity<?> search(@RequestParam(name="keyword")
                             @NotEmpty String keyword);
  }
  ```

- `@PathVariable`

  ```java
  public interface Controller {
    @GetMapping(value = "/page/{number}")
    ResponseEntity<?> search(@PathVariable
                             @Positive int number);
  }
  ```

  

## 검증과 예외 처리

데이터에 대한 유효성 검증을 실행하는 방법으로는 `@Vaild`와 `@Validated`가 있다.

### Valid

`@Vaild`는 자바 표준 스펙으로 유효성을 검증하는 과정이 `ArgumentResolver`를 통해 진행되므로 컨트롤러의 Argument로 전달되는 값만 검증할 수 있다.

유효성 검증에 실패하면 `MethodArgumentNotValidException`이 발생하게 된다.

> `ArgumentResolver`는 사용자의 요청을 전달받으면 컨트롤러가 필요한 객체를 생성하고 전달받은 데이터와 바인딩을 해주는 역할을 한다.
>
> 대표적으로  `@RequestParam`, `@RequestBody`, `@PathVariable`등이 `ArgumentResolver`로 동작한다.

```java
public interface Controller {
  @PostMapping(value = "/signup")
  ResponseEntity<?> signup(@RequestBody @Valid RequestDto requestDto);
  
  @GetMapping(value = "/search")
  ResponseEntity<?> search(@RequestParam(name="keyword")
                           @NotEmpty @Valid String keyword);
  
  @GetMapping(value = "/page/{number}")
  ResponseEntity<?> search(@PathVariable
                           @Positive @Valid int number);
}
```

`@Valid`만 사용하여 유효성 검사를 진행할 때에는 컨트롤러에서 검증을 진행할 파라미터에 어노테이션을 추가한다.

### Validated

`@Validated`는 스프링 프레임워크가 제공하는 기능으로 AOP를 통해 메서드의 요청을 인터셉트하여 스프링 빈의 유효성 검증을 진행한다.

`@Valid`만 사용하는 방식과 달리 AOP를 통해 스프링 빈의 유효성을 검증하는 방식이므로 컨트롤러 이외에서도 유효성 검사를 할 수 있다.

유효성 검증에 실패하면 `ConstraintViolationException`이 발생하게 된다.

```java
@Validated
public interface UserService extends UserDetailsService{
    void createUser(@Valid SignupDto signupDto);
}
```

검증이 발생하는 클래스에 `@Validated`어노테이션을 추가하여 해당 클래스의 `MethodValidationInterceptor`를 등록하고 실제 검증이 진행되는 메서드의 파라미터에서 `@Valid`를 추가하여 유효성을 검사한다.



-----

##### 참고자료 :

[https://adjh54.tistory.com](https://adjh54.tistory.com/77)

[https://goldenrabbit.co.kr](https://goldenrabbit.co.kr/2024/04/02/spring-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EA%B0%92-%EA%B2%80%EC%A6%9Dvalidation-%EA%B0%80%EC%9D%B4%EB%93%9C/)

[https://mangkyu.tistory.com](https://mangkyu.tistory.com/174)