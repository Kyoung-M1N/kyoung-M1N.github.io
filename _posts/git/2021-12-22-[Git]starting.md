---
title: 기초 명령어와 구조
author: kymin
date: 2021-12-22 19:31
categories: [Git]
tags: [git, github]
---
## Git 설치

Git을 설치하기 위해 [git-scm.com/downloads](https://git-scm.com/downloads)로 이동하여 각 운영체제에 맞는 것을 설치한다.

또는 쉘에서 각 운영체제에 맞는 명령어를 통해 설치가 가능하다.([git - book/download](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%84%A4%EC%B9%98))

## 원격 Repository 생성

[Git Hub](https://github.com/)에서 회원가입 후에 Your repositories 페이지의 New버튼을 눌러 원격 저장소를 생성한다.

> 옵션으로 공개, 비공개 설정을 할 수도 있고, 라이센스 파일이나 README.md파일을 자동으로 생성시킬 수 있다.

## Git 기초 명령어

- `git init`

  현재 디렉토리에 로컬 깃 저장소를 생성한다. 이 때 해당 디렉토리에는 .git디렉토리가 생성된다.

- `git config --[범위] [이름] [값]`

  git 저장소에 대한 설정을 수정할 수 있는 명령어로 설정을 적용시킬 범위를 옵션을 통해 정할 수 있다.

- `git clone [원격 저장소 주소] [로컬 디렉토리 경로]`

  명령어에 입력된 주소에 해당하는 원격 저장소의 복사본을 만들어 현재 디렉토리 로컬 깃 저장소를 생성한다. 이 때 생성된 로컬 저장소는 원격 저장소와 자동으로 연결된다.

- `git status`

  현재 디렉토리에 있는 파일들의 상태를 확인한다. 작업이 이루어지는 디렉토리와 로컬 깃 저장소를 비교하여 파일의 수정여부나 커밋상태가 표시된다.

- `git add [파일 경로] ` 

  로컬 깃 저장소에 포함된 파일들의 현재 상태를 추적하여 인덱스에 저장한다. 이 때 `-A`옵션이나 파일경로로 `.`을 입력하면 현재 디렉토리의 모든 파일에 대해 `add`명령을 실행할 수 있다. 구조적으로 보자면 현재 깃 저장소의 스냅샷을 stage area로 업로드하는 과정이다.

- `git commit`

  현재 인덱스에 추가된 변경사항들을 로컬 저장소의 변경이력에 추가한다. 주로 `-m`옵션을 통해 해당 커밋에 대한 메시지를 남긴다.

- `git push`

  원격 저장소에 로컬 저장소의 변경 이력을 업로드한다.  `-f`옵션을 통해 conflict상태에서 강제로 `push`를 실행할 수 있다. `push`를 통해 업로드된 파일은 github에서 열람 가능하다.

- `git pull`

  원격 저장소의 내용을 로컬저장소로 불러온다. 기존의 로컬저장소의 내용이 변경된 원격 저장소의 내용과 병합된다. `--ff`옵션을 통해 conflict상태에서 강제로 `pull`을 실행할 수 있다.

- `git log`

  현재 깃 저장소의 변경 이력들을 모두 표시한다. `--all --decorate --oneline --graph`옵션을 통해 시각적으로 예쁜(?)그래프로 로그를 확인할 수 있다.

  

>`push`에서의 `-f`옵션과 `pull`에서의 `--ff`옵션은 변경 전 문서의 내용이나 데이터가 손상, 유실될 위험이 있다.

> 자세한 내용은 `git`명령어 또는 `-h`옵션을 통해 확인 가능하다.



<!-- 시각적으로 명령어의 동작과 깃 저장소의 구조를 그려보면 아래와 같다.(개인적인 이해를 바탕으로 그린 거라 정확하지는 않을 수도...) -->

<!-- ![files](/public/img/git-structure.JPG) -->





-----

##### 참고자료 : 

[git-scm.com](https://git-scm.com/book/ko/v2)

[inflearn - Git](https://www.inflearn.com/course/%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%EC%98%A8-git/dashboard)

[AWS CodeCommit](https://docs.aws.amazon.com/ko_kr/codecommit/latest/userguide/how-to-basic-git.html)

[subicura.com](https://subicura.com/git/guide/basic.html#git-init-저장소-만들기)

