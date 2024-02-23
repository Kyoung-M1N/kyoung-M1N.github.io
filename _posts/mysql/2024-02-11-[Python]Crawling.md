---
title: 데이터 크롤링과 DB생성
author: kymin
date: 2024-02-11 21:11
categories: [Database]
tags: [crawling, database, python]

---

## crawling

### 크롤링(crawling)이란?

크롤링은 웹 페이지를 그대로 가져와서 데이터를 추출해 내는 행위를 말하며 크롤링을 하는 소프트웨어는 크롤러라고 한다.

크롤링이라는 용어는 스크래핑(Scraping)과 자주 혼동되지만 크롤링은 불특정 웹 사이트들을 링크를 통해 탐색하 웹 페이지나 링크를 다운로드하고, 스크래핑은 웹 뿐만 아니라 일반 문서등의 다양한 소스들에게 특정 데이터를 추출한다는 점에서 차이가 있다.

크롤링은 주로 원하는 데이터를 얻기 위한 목적으로 웹 상에 떠도는 정보들을 탐색하고 수집하는 데에 사용된다.

크롤링을 통해 얻은 데이터들은 데이터베이스 등에 저장되어 각자 목적에 맞게 사용된다.

### 법적인 문제

무분별한 크롤링은 서버에 너무 많은 요청을 하게 되고, 이에 따라 서비스에 장애를 일으킬 수도 있다.

또한 법적으로 접근권한이 없으면 데이터에 접근, 데이터를 사용하여서는 안된다.

> "정보통신망법 제48조 정보통신망 침해행위 금지"
> 누구든지 정당한 접근권한 없이 또는 허용된 접근권한을 넘어 정보통신망에 침입하여서는 아니된다.

따라서 항상 모든 웹사이트의 모든 웹페이지들을 크롤링할 수 있는 것은 아니다.

이에 대해 크롤링을 허용하는지에 대한 정보를 `robots.txt`파일에서 확인한 뒤에 크롤링을 진행해야 한다.

```
// https://www.ftc.go.kr/robots.txt(공정거래위원회 홈페이지)
User-agent: *	// 모든 사용자에 대해 크롤링 허용
Allow : /	// 모든 하위 페이지에 대해 크롤링 허용
```

```
// https://www.daangn.com/robots.txt(당근마켓 홈페이지)
User-agent: *
Disallow: /ad/*
Disallow: /admin
Disallow: /wv/*
Allow: /wv/faqs
Allow: /wv/faqs/*
Allow: /wv/feedbacks/new
Disallow: /kr/job-posts/about

Sitemap:https://www.daangn.com/sitemap-index.xml
```

위의 예시에서 볼 수 있듯이 공정거래위원회는 모든 사용자가 모든 웹페이지에 대해 크롤링하는 것을 허용하지만 당근마켓은 일부는 허용하고 일부는 금지하는 것을 알 수 있다.

## 크롤링으로 DB생성

많은 사람들이 크롤링을 할 때에 파이썬의 `beautifulsoup`나 `selenium`라이브러리를 사용한다.

여러페이지를 이동하 데이터를 추출하는 데에는  `beautifulsoup`가 `selenium`보다 조금 더 적합하기 때문에  `beautifulsoup`를 사용하였다.

### 요청

`requests`라이브러리에서 제공하는 http method를 이용하여 데이터를 가져올 웹페이지에 GET요청을 통해 텍스트 형태의 html파일을 응답으로 받아온다.

```python
request = requests.get("url")
```

### Html 파싱(parsing)

GET 요청의 결과로 받아온 텍스트 형태의 html파일을 `BeautifulSoup`객체로 변환한다.

```python
parse = BeautifulSoup(request.text, "html.parser")
```

원하는 데이터를 html 태그와 속성 또는 CSS선택자를 매개로 하여 파싱한다.

이 때 html태그와 클래스 등에 대한 정보는 실제 웹사이트에서 개발자도구를 이용하여 파악해야 한다.

- `find()`

  `find()`는 태그, 속성과 속성값을 활용하여 html 문서의 내용을 추출할 수 있다.

  `find()`는 특정 태그의 내용 중 하나만을 추출할 때 사용되고 특정 태그에 포함되는 여러 개의 내용을 리스트로 가져오고 싶다면 `find_all()`을 사용한다.

  html의 id나 class등의 속성과 속성값 추가적으로 활용하여 값을 추출할 수도 있다.

  ```python
  # p태그의 내용을 찾아 추출
  parse.find('p')
  # class속성의 title의 내용을 찾아 추출
  parse.find(attrs = {'class':'title'})
  # p태그의 내용 중 h1태그의 내용을 찾아 추출
  parse.find('p').find('h1')
  # a태그의 내용 중 class속성의 title에 대한 모든 내용을 찾아 추출
  parse.find_all("a", {"class":"title"})
  ```

- `select()`

  `select()`는 css의 태그 선택자, 클래스 선택자, 아이디 선택자를 통해 html문서의 내용을 추출한다.

  `select()`는 특정 태그에 포함되는 여러 개의 내용을 리스트로 가져오시 위해 사용되며 특정 태그의 내용 중 하나만을 추출할 때는 `select_one()`을 사용한다.

  ```python
  # p태그의 내용을 모두 찾아 추출
  parse.select('p')
  # p태그의 내용을 찾아 추출
  parse.select_one('p')
  # title속성에 대한 내용을 찾아 추출
  parse.select_one('.title')
  ```

### 데이터 추출

문자열 슬라이싱과 조건문 등을 통해 원하는 데이터를 추출한다.

### DB저장

- DB실행

  ```shell
  mysql.server start
  ```

  위의 명령어를 통해 DB를 실행한다.

- DB에 연결

  ```python
  conn = MySQLdb.connect(
      user = "DB에 접근할 계정",
      host = "DB에 접근할 위치",
      passwd = "DB에 접근할 계정의 비밀번호",
      db = "접근할 DB 이름"
  )
  ```

  위의 코드는 `mysql -u<계정> -h<접근 위치> -p <DB이름>`명령어와 같은 역할을 수행한다.

- cursor 생성

  ```python
  cursor = conn.cursor()
  ```

  DB에 쿼리문 수행을 명령할 cursor를 생성한다.

- 쿼리문 실행

  ```python
  cursor.execute(f"쿼리문")
  ```

  cursor를 통해 DB에 원하는 쿼리문 실행을 전달한다.

- commit

  ```python
  conn.commit()
  ```

  변경사항을 DB에 적용하기 위해 commit명령 수행한다.

- DB와 연결 종료

  ```python
  conn.close()
  ```

  DB에 대한 작업이 끝나면 DB와의 연결을 종료한다.

### 전체 코드

```python
from bs4 import BeautifulSoup
import requests
import MySQLdb

conn = MySQLdb.connect(
    user = "DB에 접근할 계정",
    host = "DB에 접근할 위치",
    passwd = "DB에 접근할 계정의 비밀번호",
    db = "접근할 DB 이름"
)
cursor = conn.cursor()

request = requests.get("url")
parse = BeautifulSoup(request.text, "html.parser")

parsingResult = parse.find_all("<html태그>", {"class":"title"}).get_text()

for item in parsingResult:
    cursor.execute(f"insert into <테이블 이름>(<column 이름>) values(\"{item}\")")
conn.commit()
conn.close()
```



---

##### 참고자료 :

[https://hyun-am-coding.tistory.com/entry](https://hyun-am-coding.tistory.com/entry/%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0-DB%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)

[https://blog.hectodata.co.kr](https://blog.hectodata.co.kr/crawling_vs_scraping/)

[https://modulabs.co.kr/blog/crawling-tips](https://modulabs.co.kr/blog/crawling-tips/)

[https://wikidocs.net/157601](https://wikidocs.net/157601)