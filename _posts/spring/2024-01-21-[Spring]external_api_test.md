---
title: 스프링 외부 API 테스트
author: kymin
date: 2024-01-21 17:34
categories: [Spring]
tags: [spring, web, java, api]

---

## 외부 API의 테스트 코드

외부에서 제공하는 API를 사용하여 개발한 기능도 다른 기능들과 마찬가지로 테스트코드를 작성해야한다.

webClient를 이용하여 외부API를 사용하는 기능을 만든 경우에 webTestClient의 인스턴스를 통해 테스트를 진행하면 외부API를 사용하는 로직의 동작을 테스트할 수 있지만 그 과정에서 실제 외부 API를 호출하게 된다.

이 때, 외부 API는 호출량이 제한되어있는 경우가 있고, 만약 외부 API가 잘 동작하지 않는다면 테스트를 할 수 없다는 문제가 있다.

## Mock?

mock은 한글로 '모조품'이라는 의미로 테스트할 때 필요한 실제 객체와 동일한 모의 객체를 만들어 테스트의 효용성을 높이기 위해 사용한다.

외부 API를 이용하는 로직을 테스트하기 위해서는 Mockito를 이용한 방법과 Mock Server를 이용하는 방법이 있다.

Mockito를 이용하는 방법은 Mock객체를 선언하고 외부 API를 호출하는 로직의 결과로 반환되는 객체를 Mock객체로 설정하여 테스트를 진행한다.

Mock Server를 이용하는 방법은 외부 서버를 Mocking하여 

외부 api나 라이브러리를 사용하는 경우 해당 로직이 작동하는지 테스트하기 위해 mock객체를 생성하여 로직에 대한 테스트를 진행한다.

또한 특정 로직에 대한 반환값을 임의로 설정하여 반환되는 결과값과 상관없이 로직의 작동여부를 테스트할 수 있다.



## Mockito

mockito는 mock객체를 생성하고 검증할 수 있는 기능을 제공하는 프레임워크이다.

### 의존성 추가

현재는 Spring Boot 프로젝트를 생성할 때, 이미 testImplementation된 `‘org.springframework.boot:spring-boot-starter-test’`라이브러리에 mockito가 포함되어있기 때문에 별도의 의존성을 추가하지 않아도 된다.

### 테스트코드 작성

```java
public class MockTest {
  @Test
  public void mockTest() {
    // given
    ApiService apiService = new ApiService();
    Person mockObject = mock(Person.class);	// mock객체 생성
    when(apiService.getPerson()).thenReturn(mockObject);	// stubbing : 가짜 반환값 설정
    
    // when
    Person person = mockObject.getPerson();
    String result = person.getName();
    
    // then
    assertThat(result).isEqualTo("name");
  }
}
```

`ApiService`의 `getPerson()`이 외부 API를 호출하여 얻은 응답을 `Person`으로 반환한다면, 위의 코드에서는 Mock 객체로 `mockObject`를 선언하고 `getPerson()`의 결과로 Mock 객체인 `mockObject`를 반환하도록 한다.



## MockWebServer

MockWebServer는 실제 외부 API서버를 대신하여 HTTP Request를 받아서 Response를 반환하는 기능을 제공한다.

### 의존성 추가

아래와 같이 스프링 프로젝트에 포함된 `build.gradle`의 `dependencies`에 MockWebServer에 대한 의존성을 추가한다.

```java
dependencies {
	...
	implementation 'com.squareup.okhttp3:mockwebserver'
	...
}
```

### 테스트코드 작성

```java
public class MockServerTest {
    private MockWebServer mockWebServer;
    private WebClient webClient;
    private final String response = "{\n" +
            "\"category_group_code\": \"FD6\",\n" +
            "\"category_group_name\": \"음식점\",\n" +
            "\"x\": \"127.301488273626\",\n" +
            "\"y\": \"37.6675046684288\"\n" +
            "}";

    @BeforeEach
    void serverStart() throws IOException {
        this.mockWebServer = new MockWebServer();	// mock서버 생성 후 실행
        // mockWebServer의 인스턴스를 생성할 때 mock서버가 자동으로 실행되므로 mockWebServer.start()를 생략
    }

    @AfterEach
    void shutdown() throws IOException {
        if (mockWebServer != null) {
            this.mockWebServer.shutdown();
        }
    }
    
    @Test
    public void getRestaurantsTest() {
        // given
        mockWebServer.enqueue(new MockResponse()
                .setResponseCode(200)
                .setBody(response)
                .addHeader("Content-Type", "application/json")); // 응답 생성
        webClient = WebClient
                .builder()
                .baseUrl(this.mockWebServer.url("/").toString())
                .build();	// webClient의 baseUrl을 mock서버로 설정
        // when
        Map<String, String> result = this.webClient.get().retrieve()
                .bodyToMono(Map.class)
                .block();
        // then
        assertThat(result.get("category_group_code")).isEqualTo("FD6");
    }
}

```

`@BeforeEach`와 `@AfterEach`를 통해 각 테스트의 시작과 끝에 mock서버를 실행시키고 중지시킨다. 이 때 mockWebServer의 인스턴스 선언 과정에서 mock서버의 실행이 자동으로 시작되므로 `mockWebServer.start()`를 생략한다.

`mockWebServer.enqueue()`를 통해 mock서버의 응답queue에 mock서버 호출에 대한 응답을 삽입한다.

`webClient`의 baseUrl을 mock서버로 설정 후 테스트를 진행한다.

---

##### 참고자료 :

[https://velog.io/@ejung803](https://velog.io/@ejung803/Mock-%EC%9D%B4%EB%9E%80)

[https://velog.io/@kyle](https://velog.io/@kyle/%EC%99%B8%EB%B6%80-API%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%A0-%EA%B2%83%EC%9D%B8%EA%B0%80)

[https://lahezy.tistory.com/108](https://lahezy.tistory.com/108)

[https://tecoble.techcourse.co.kr/post](https://tecoble.techcourse.co.kr/post/2020-09-30-mocking-server/)

[https://www.devkuma.com](https://www.devkuma.com/docs/mock-web-server/)