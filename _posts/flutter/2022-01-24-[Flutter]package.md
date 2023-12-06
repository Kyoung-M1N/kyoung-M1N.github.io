---
title: 플러터 패키지 만들기
author: kymin
date: 2022-01-24 21:11
categories: [Mobile, Flutter]
tags: [flutter, dart]
---
## 패키지 유형

플러터의 패키지는 다트 패키지와 플러그인 패키지로 나누어진다.

- 다트 패키지

  단순하게 Dart 파일만 제공하는 패키지로 Dart언어만을 이용하여 작성되기 때문에, 플랫폼별로 별도의 코드를 작성하지 않아도 되는 경우에 사용된다.  dart:math와 같은 기본 라이브러리들이 이에 해당한다.

- 플러그인 패키지

  플러터 앱에 특정 기능을 제공하기 위해 플랫폼별 구현과 결합된 Dart 코드로 작성한 API를 포함하는 특수 Dart 패키지이다. 같은 기능이라도 구현이나 동작 과정에서 플랫폼 별로 차이가 있기 때문에, 해당 플랫폼의 네이티브 언어들(Swift, Kotlin 등)로도 작성가능한 부분이 존재한다.



## 패키지 프로젝트 생성

패키지를 생성하기위해 아래의 명령어로 프로젝트를 생성한다.

```shell
flutter create --org [org이름] --template=[템플릿 종류] --platforms=[지원할 플랫폼들] -a [안드로이드 언어] -i [ios언어] [패키지 이름]
```

-  `--org` : 플러터 프로젝트에 명시할 organization의 이름이다.
- `--template` : 템플릿의 종류를 결정하는 옵션으로 기본값은 app이다.
- `--platforms` : 지원할 플랫폼에 관한 옵션으로 --template이 app이나 plugin일 때만 작동하고, 기본적으로 모든 플랫폼을 지원하도록 설정되어있다.
- `-a` : 안드로이드의 언어로 Java를 사용할 것인지 Kotlin을 사용할 것인지를 결정하는 옵션으로 기본값은 Kotlin이다.
- `-i` : ios의 언어로 ObjectiveC를 사용할 것인지 Swift을 사용할 것인지를 결정하는 옵션으로 기본값은 Swift이다.

>패키지를 생성하는 명령어를 실행하면 `flutter pub get `도 함께 실행되기  때문에 인터넷이 연결되지 않은 상태에서 프로젝트를 생성하기 위해서는  `--offline`옵션도 추가해주어야 한다.
>
>다트 패키지를 생성할 때에는 템플릿의 종류를 `package`로, 플러그인 패키지를 생성할 때에는 템플릿의 종류를 `plugin`으로 설정해주면 된다.
>
>더 자세한 내용은 `flutter create --help`명령어를 통해 알아보자.

예시

```shell
flutter create --org kr.co.shiban --template=plugin --platforms=android,ios,web cotten_candy_ui
```



## 패키지 소스코드 작성

위의 예시 명령어를 입력한 결과로 아래와 같은 파일들이 생성되었다.

![files](/public/img/flutter-screenshot1.png)

전체적인 구조를 살펴보면 일반적인 app프로젝트를 생성했을 때 만들어지는 구조가 example폴더 안에 똑같이 들어가있는 것을 알 수 있다.

패키지를 통해 제공하고자 하는 기능에 대한 소스코드는 lib디렉토리 내에서 작성하면 되고, 패키지에 대한 예시코드는 example디렉토리 내에서 자신이 작성한 플러그인을 import한 뒤에 작성하면 된다.

패키지에 대한 소스코드와 각종 설정, 문서들의 작성이 완료되면 깃허브에 레포지토리를 생성하고 push한다.

>pubspec.yaml에서 패키지의 버전, description, homepage에 대한 값들을 설정해주어야한다. 이 때 description은 60자 이상으로 쓰는 것을 추천(pub point 권장사항)
>
>CHANGELOG.md에 버전별 변경사항과 추가사항들을 잘 작성해준다.
>
>README.md에 패키지에 대한 전반적인 정보를 잘 작성해준다.
>
>LICENSE문서는 직접 작성해도 되지만 깃허브 레포지토리 생성 시 설정을 통해 만들어지는 것을 사용하는게 편하다.
>
>더 자세한 내용은 맨 아래의 결과물 링크를 참조



## Pub.dev에 패키지 배포

>pub.dev에서 패키지들을 검색하다보면 PUB POINTS라는 것을 볼 수 있는데 이는 해당 패키지의 전반적인 품질정도를 의미한다.
>
>PUB POINTS가 매겨지는 기준은 맨 아래의 참고사항 링크에서 확인 가능하다.
>
>최종 배포 전에 PUB POINTS의 기준에 따라 코드나 문서를 수정하는 것을 추천한다.

위의 과정을 모두 끝냈다면 깃의 원격저장소에 올라가있는 내용을 pull로 로컬저장소에 가져온다.

터미널에서 패키지 프로젝트가 있는 경로로 이동하고 아래의 명령어를 통해 배포기준을 충족시키는지 확인해본다.

```shell
flutter pub publish --dry-run
```

위의 명령어의 결과로 `Package has 0 warnings.`라는 메시지가 뜬다면 이제 배포만 하면 된다.

아래의 명령어를 통해 pub.dev에 직접 만든 패키지를 배포한다.

```shell
flutter pub publish
```

명령어를 실행하면 pub.dev에서 로그인을 진행하라고 하는데 로그인을 마치면 패키지가 심사에 들어간다.

심사는 대략 2시간 정도 소요되는 듯하며 심사가 끝나면 PUB POINTS가 매겨지고 직접 만든 패키지가 다른 패키지들과 함께 pub.dev에 올라와 있는 것을 볼 수 있다.

아래의 링크는 위의 과정에 따라 실제로 패키지를 만들어서 배포해본 결과물이다.

[pub.dev : cotton_candy_ui](https://pub.dev/packages/cotton_candy_ui)



-----------------------

##### 참고자료 : 

[https://docs.flutter.dev/development/packages-and-plugins/developing-packages](https://docs.flutter.dev/development/packages-and-plugins/developing-packages)

[https://pub.dev/help/scoring](https://pub.dev/help/scoring)

