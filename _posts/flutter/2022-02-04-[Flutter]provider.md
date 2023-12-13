---
title: 플러터 상태관리 provider
author: kymin
date: 2022-02-04 21:11
categories: [Mobile, Flutter]
tags: [flutter, dart]
---
## provider란?

provider는 플러터 애플리케이션을 구성하는 위젯들의 상태(데이터)를 관리하기 위한 개발 패턴이자 패키지이다. 

구글에서도 공식적으로 상태관리를 위해 provider를 사용할 것을 권장하기도 하였다.(대규모 프로젝트에는 provider 패턴보다 BloC 패턴을 더 권장한다.)

## provider를 사용하는 이유

- 관심사의 분리

  관심사는 코드가 하는 역할이나 수행하는 일을 의미한다. flutter는 선언형 UI로 코드를 작성하다보면 각 기능들을 담당하는 코드와 UI를 생성하는 코드가 섞여 복잡해질 수 있다. 따라서 이를 막기 위해 provider를 사용하여 UI와 데이터를 분리하여 관리할 수 있다.

- 원활한 데이터 공유

  flutter에서 여러 페이지가 데이터를 공유하기 위해서는 각 페이지에서 해당 데이터를 저장할 변수를 선언하고, 페이지가 호출될 때마다 계속해서 변수의 상태를 갱신해야 한다는 번거로움이 있다.

  이 과정은 코드상으로도 매우 복잡해서 알아보기도 어렵다. provider를 이용하면 데이터를 따로 관리하는 클래스에서 데이터의 상태를 관리하게 되며, 각 페이지들은 호출될 때마다 갱신할 필요없이 provider가 전달하는 데이터를 가져다 쓰기만 하면 된다.

- 간결한 코드

  provider는 상태관리를 위한 다른 패턴들 보다 더 적은 코드로 클래스들을 분리해내고 상태관리를 수행할 수 있다. 이에 따라 소스 코드를 작성할 때 더 간편해질 뿐만 아니라 가독성을 높여주고 유지 보수를 원활하게 할 수 있도록 만들어 준다.

## provider 사용

provider는 데이터를 생상하는 부분과 소비하는 부분으로 나뉜다고 한다.

> 생산한다는 개념이 잘 와닿지 않는다면 provider를 직역한 의미인 ''제공한다'' 라는의미로 받아들여도 좋을 것 같다.

일단 provider를 사용하기 위해 `pubspec.yaml` 파일에 의존성 추가를 해준다.

```yaml
dependencies:
  provider: <last version>
```

### **데이터 생산**

데이터를 생산한다는 말은 provider가 관리할 데이터를 선언하거나 전달하는 것을 의미한다.

가장 기본적인 방법으로는 `provider<T>.value()`를 이용하여 데이터를 생성하는 방법이 있다.

```dart
Provider<int>.value(
      value: 5,
      child: Container(),
)
```

하지만 pub.dev의 공식 provider문서에서는 `.value()`를 이용한 데이터 생산을 권장하지 않는다.

보편적으로는 `ChangeNotifier`를 상속받은 클래스 내부에 변수와 변수를 조작하는 함수를 생성하여 데이터를 생산 관리한다.

이 때 값의 변화를 인식하여 프레임워크에 전달하고 UI를 새로 갱신하기 위해 상태변경을 수행하는 함수의 내부에서 상태변경 후에 꼭  `notifyListeners()`를 실행해주어야 한다.

> `ChangeNotifier`를 상속받은 클래스는 `notifyListeners()`를 통해 값의 변화를 인식하여 프레임워크에 전달하고, `ChangeNotifierProvider`는 `notifyListeners()`를 통해 전달받은 값을 반영하여 UI를 다시 빌드한다.

```dart
import 'package:flutter/material.dart';

class Value extends ChangeNotifier {
  int _num = 0;
  int get num => _num;

  void add() {
    _num++;
    notifyListeners();
  }

  void remove() {
    _num--;
    notifyListeners();
  }
}
```

`ChangeNotifier`를 상속받은 클래스에서 발생한 데이터의 변화를 전달받고, UI에 반영시키기 위해 앱을 구성하는 프로젝트의 최상위 위젯인 `MyApp`을 `ChangeNotifierProvider`로 감싸준다.

이 때 `MultiProvider`를 통해 여러 개의 provider에서 일어나는 변화를 전달받을 수 있다.

> `ChangeNotifier`를 상속받은 클래스에서 `ChangeNotifierProvider`로 데이터 변화를 전달하는 것은 `notifyListeners()`를 통해 이루어진다.

```dart
void main() {
  runApp(MultiProvider(
    providers: [
    ChangeNotifierProvider(
      create: (context) => Value()),
      ],
      child: const MyApp(),
    ),
  );
}
```



### **데이터 소비**

데이터를 소비한다는 말은 provider의 데이터 값을 변경하거나 화면에 보여주는 것을 의미한다.

provider에서 관리하는 값에 변화를 주기 위해서는 `context.watch<T>()`를 통해 값을 보여주거나 `context.read<T>()`로 값을 변경할 수 있다.

보통은 아래와 같이 `Provider.of(context)`로 `ChangeNotifier`를 상속받은 클래스의 데이터를 사용하거나 변경할 수 있다.

> `context.watch<T>()`는 `Provider.of<T>(context)`와 동일한 기능을 수행하고, `context.read<T>()`는 `Provider.of<T>(context, listen: false)`와 동일한 기능을 한다.

```dart
class Home extends StatelessWidget {
  const Home({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Value _value = Provider.of<Value>(context, listen: false);
    return Scaffold(
      appBar: AppBar(
        title: const Text('provider study'),
      ),
      body: Center(
        child: Column(mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const ValueWidget(),
            const SizedBox(
              height: 50,
            ),
            TextButton(
                onPressed: () => Navigator.push(context,
                    MaterialPageRoute(builder: (context) => const Next())),
                child: const Text('Go to next'))
          ],
        ),
      ),
      floatingActionButton: SizedBox(
        width: 100,
        child: Row(
          children: [
            IconButton(
              icon: const Icon(Icons.add),
              onPressed: () {
                _value.add();
              },
            ),
            IconButton(
              icon: const Icon(Icons.remove),
              onPressed: () {
                _value.remove();
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

또한 `Consumer`를 통해서도 `ChangeNotifier`를 상속받은 클래스의 데이터를 사용하거나 변경할 수 있다.

보통 provider에 의한 데이터 생산과 소비가 한 곳에서 이루어지는 상황에서 사용하며, 이러한 상황은 `ChangeNotifierProvider`의 child에 `build`를 통해 생성된 위젯이 아닌 직접 생성된 위젯이 존재하여  `context.watch<T>()`와 `context.read<T>()`가 `ChangeNotifierProvider`의 자식들에게서 context를 탐색할 수 없게 되었을 때 발생한다.

> 따라서 보통은 `ChangeNotifier`를 상속받은 클래스에서 관리하는 데이터를 앱의 하위클래스들이 쉽게 접근할 수 있도록 `MaterialApp`클래스나  `MaterialApp`을 리턴하는 최상위 클래스를 `ChangeNotifierProvider`로 감싸준다.

```dart
class ValueWidget extends StatelessWidget {
  const ValueWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Value>(
        create: (_) => Value(),
        child: Center(
      	child: Consumer<Value>(
        	builder: (context, value, child) => Text(
                value.num.toString(),
            ),
      	),
      ),
    );
  }
}
```

**전체 예시 코드** 

[https://github.com/Kyoung-M1N/flutter_provider_study](https://github.com/Kyoung-M1N/flutter_provider_study)



대충 전체적인 구조를 보면 아래 그림과 같다.

![files](/public/img/flutter-provider.JPG)





-----------------------

##### 참고자료 : 

[pub.dev : provider](https://pub.dev/packages/provider)

[https://nomad-programmer.tistory.com/263](https://nomad-programmer.tistory.com/263)

