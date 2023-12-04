---
title: shared_preferences
author: kymin
date: 2022-02-18 20:11
categories: [Mobile, Flutter]
tags: [flutter, dart]
---
# Shared Preference

### shared preferences란?

shared preferences는 플러터 어플리케이션이 key - value형태의 데이터를 디스크에 저장하기 위한 플러그인으로, IOS의 NSUserrDefaults와 안드로이드의 SharedPreferences의 기능을 플러터에서 사용할 수 있도록 만들어졌다.

무엇보다 데이터를 저장, 로드, 삭제하는 방법이 매우 간단하다.

> 데이터 저장을 위해 저장할 데이터를 key - value형태로 만들어야 하는데, 이 때 key는 항상 String이어야 한다.
>
> 지원되는 데이터의 자료형이 한정되어 있고 대용량의 데이터를 저장하기에 적합하지 않기 때문에 간단하고 중요도가 높지 않은 데이터를 기억하는 용도로 사용할 것을 권장하고 있다.

### shared preferences 사용

shared preference가 제공하는 기능은 데이터 저장하기, 데이터 읽기, 데이터 삭제하기로 나눌 수 있다.

> shared preference를 통해 저장, 삭제, 로딩이 가능한 자료형은 int, double, bool, String, List\<String\>뿐이다.

일단 shared preferences를 사용하기 위해 `pubspec.yaml` 파일에 의존성 추가를 해준다.

```yaml
dependencies:
  shared_preferences: <last version>
```

- **데이터 저장하기**

  key - value형태의 데이터를 저장하는 방법은 아래와 같다.

  ```dart
  // Obtain shared preferences.
  final prefs = await SharedPreferences.getInstance();
  
  await prefs.setInt('counter', 10);
  await prefs.setBool('repeat', true);
  await prefs.setDouble('decimal', 1.5);
  await prefs.setString('action', 'Start');
  await prefs.setStringList('items', <String>['Earth', 'Moon', 'Sun']);
  ```

  먼저 `getInstance()`를 통해 앱에 할당된 메모리에 접근하고 setter메소드로 데이터를 저장하는 것으로 보인다.

  > `getInstance()`는 호출시에 최초에 할당된 하나의 메모리를 계속해서 사용하기 때문에 변수 이름을 다르게 하거나 다른 파일, 클래스에서 `getInstance()`를 실행해도 데이터의 저장, 로딩, 삭제는 메모리 상에서 같은 곳에서 발생한다.



- **데이터 읽기**

  key - value형태의 데이터를 읽어오는 방법은 아래와 같다.

  ```dart
  final prefs = await SharedPreferences.getInstance();
  
  final counter = prefs.getInt('counter') ?? 0;
  final repeat = prefs.getBool('repeat') ?? true;
  final decimal = prefs.getDouble('decimal') ?? 0;
  final action = prefs.getString('action') ?? '';
  final items = prefs.getStringList('items') ?? [''];
  ```

  마찬가지로 먼저 `getInstance()`를 통해 앱에 할당된 메모리에 접근하고 getter메소드로 데이터를 불러오는 것으로 보인다.

  >각 자료형에 따른 getter함수의 호출결과 key에 해당하는 값이 저장되어있지 않다면 null을 리턴한다.
  >
  >따라서 ??(null 병합 연산자)를 통해 기본값을 설정하거나 불러온 데이터를 저장할 변수를 nullable로 선언해주어야 한다.



- **데이터 삭제하기**

  key - value형태의 데이터를 삭제하는 방법은 아래와 같다.

  ```dart
  final prefs = await SharedPreferences.getInstance();
  
  prefs.remove('counter');
  ```

  마찬가지로 먼저 `getInstance()`를 통해 앱에 할당된 메모리에 접근한 뒤에,  데이터를 삭제하는 것으로 보인다. 이 때 `remove()`의 parameter로 전달된 key에 해당하는 데이터가 존재하지 않는다면 아무 일도 일어나지 않는다.

**예시 코드**

[pub.dev : shared_preferences/example](https://pub.dev/packages/shared_preferences/example)

-----------------------

##### 참고자료 : 

[flutter-ko.dev](https://flutter-ko.dev/docs/cookbook/persistence/key-value)

[pub.dev : shared_preferences](https://nomad-programmer.tistory.com/263)

