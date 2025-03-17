---
title: Branch와 Merge
author: kymin
date: 2022-01-05 19:31
categories: [Git]
tags: [git]
---
## Branch

브랜치(branch)는 어떤 작업이 이루어지고 있는 하나의 줄기라고 생각하면 된다.

깃과 깃허브를 이용한 협업 과정에서 여러 개발자들이 동시에 다양한 작업을 할 수 있게 만들어 주는 기능이다.

저장소를 처음 만들면, git에서 자동으로 'master' 또는 'main'라는 이름의 브랜치를 생성한다.

> 인종차별적 요소나 주종관계의 의미를 담고 있는 whitelist/blacklist, master/slave 등의 용어를 사용하지 않으려는 업계 전반적인 움직임에 따라 요즘에는 main브랜치가 기본적으로 생성된다.

### Branch 생성

기본적으로 브랜치를 생성하는 방법은 두 가지가 있다.

- 명령어로 branch 생성

  ```shell
  git branch [브랜치 이름]
  ```

  이 때 `git branch`에 아무런 옵션이나 브랜치 이름이 주어지지 않으면 브랜치 목록을 보여준다.

  `branch`명령어는 브랜치를 생성만 하고 HEAD를 해당 브랜치로 이동시키지 않는다.

  또한 새로운 브랜치는 마지막 커밋을 가리킨다.

- branch를 이동하는 명령어로 새로운 branch를 생성하며 이동

  ```shell
  git checkout -b [브랜치 이름]
  git switch -c [브랜치 이름]
  ```

  위의 두 명령어는 `git branch`와는 달리 브랜치를 이동하는 명령어지만 각각 `-b`옵션과 `-c`옵션을 통해 새로운 브랜치를 생성함과 동시에 이동을 하는 명령어이다.

### Branch 이동

브랜치를 이동하는 방법에도 두 가지가 있다.

- `checkout`

  ```shell
  git checkout [브랜치 이름]
  ```

  `git checkout`에 파일경로를 주면 해당 파일은 수정 전으로 돌아간다(add 명령어를 실행하지 않았을 경우에만).

  `git add`로 stage에 올라간 파일을 되돌리기 위해서는 `git reset`으로 처리해야 한다.

  즉, `git checkout`은 브랜치를 만드는 기능과 되돌리는 기능을 둘 다 수행한다.

  이를 분리한 명령어가 아래의 `git switch`와 `git restore`이다.

- `switch`

  `switch`는 `checkout`명령어의 기능을 `switch`와 `restore`로 분리하며 만들어진 명령어로 각각 브랜치의 이동과 복원을 담당한다.
  
  ```shell
  git switch [브랜치 이름]
  git restore [파일 이름]
  ```
  
  이 때 `git restore`은 `git checkout`과 달리 `--staged`옵션을 사용하면 `add`로 인해 stage에 올라간 파일까지 되돌릴 수 있다.

## Merge?

머지(Merge)는 서로 다른 브랜치의 내용 또는 원격 저장소와 로컬 저장소의 내용을 병합하는 것을 말한다.

### Branch 병합(Merge)

두 브랜치를 병합하기 위해 몇가지 과정이 필요하다.

```shell
git switch main
git merge [병합할 브랜치 이름]
```

1. 먼저 main브랜치 또는 병합을 진행할 브랜치로 이동한다.
2. `git merge`로 다른 브랜치를 현재 작업중인 브랜치에 병합한다.

위의 과정을 거치면 `git merge`에 주어진 브랜치는 main브랜치에 병합되고 사라진다.

## Rebase

리베이스(rebase)는 기능상으로만 본다면 브랜치를 병합한다는 점에서 머지와 같다고 볼 수 있지만 구조적으로 본다면 차이점이 존재한다.

리베이스는 해당 브랜치의 base를 옮긴다는 의미이다. 즉 main브랜치를 다른 브랜치로 병합 후 병합된 브랜치를 main으로 만든다.

### Branch 병합(Rebase)

rebase도 역시 몇가지 과정이 필요하다.

```shell
git switch [병합할 브랜치 이름]
git rebase main
```

1. 머지와는 다르게 병합시킬 브랜치로 이동한다.
2. `git rebase`로 병합시킬 브랜치의 base를 main브랜치의 base로 변경한다.



-----

##### 참고자료 : 

[https://git-scm.com](https://git-scm.com/book/ko/v2)

[https://www.inflearn.com/course](https://www.inflearn.com/course/%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%EC%98%A8-git/dashboard)

[https://subicura.com](https://subicura.com/git/guide/branch.html#git-switch-c-브랜치-생성)

