---
layout: post
title: "[Flutter]provider 사용법"
date: 2022-02-04 21:11
categories: Flutter
---
# provider?

provider는 플러터 애플리케이션을 구성하는 위젯들의 상태(데이터)를 관리하기 위한 개발 패턴이자 패키지이다. 

구글에서도 공식적으로 상태관리를 위해 provider를 사용할 것을 권장하기도 하였다.

> 대규모 프로젝트에는 provider 패턴보다 BloC 패턴을 더 권장한다.

#### provider를 사용하는 이유

- **관심사의 분리**

  관심사는 코드가 하는 역할이나 수행하는 일을 의미한다. flutter는 선언형 UI로 코드를 작성하다보면 각 기능들을 담당하는 코드와 UI를 생성하는 코드가 섞여 복잡해질 수 있다. 따라서 이를 막기 위해 provider를 사용하여 UI와 데이터를 분리하여 관리할 수 있다.

- **원활한 데이터 공유**

  flutter에서 여러 페이지가 데이터를 공유하기 위해서는 각 페이지에서 해당 데이터를 저장할 변수를 선언하고, 페이지가 호출될 때마다 계속해서 변수의 상태를 갱신해야 한다는 번거로움이 있다. 이 과정은 코드상으로도 매우 복잡해서 알아보기도 어렵다. provider를 이용하면 데이터를 따로 관리하는 클래스에서 데이터의 상태를 관리하게 되며, 각 페이지들은 호출될 때마다 갱신할 필요없이 provider가 전달하는 데이터를 가져다 쓰기만 하면 된다.

- **간결한 코드**

  provider는 상태관리를 위한 다른 패턴들 보다 더 적은 코드로 클래스들을 분리해내고 상태관리를 수행할 수 있다. 이에 따라 소스 코드를 작성할 때 더 간편해질 뿐만 아니라 가독성을 높여주고 유지 보수를 원활하게 할 수 있도록 만들어 준다.



# provider 사용







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







```dart
class ValueWidget extends StatelessWidget {
  const ValueWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Consumer<Value>(
        builder: (context, value, child) {
          return Text(value.num.toString());
        },
      ),
    );
  }
}
```



전체 예시 코드 : [https://github.com/Kyoung-M1N/flutter_provider_study](https://github.com/Kyoung-M1N/flutter_provider_study)



대충 전체적인 구조를 보면 아래 그림과 같다.

![files](/public/img/flutter-provider.JPG)





-----------------------

##### 참고자료 : 

[pub.dev : provider](https://pub.dev/packages/provider)

[https://nomad-programmer.tistory.com/263](https://nomad-programmer.tistory.com/263)

