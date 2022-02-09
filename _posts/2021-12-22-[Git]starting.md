---
layout: post
title: "[Git & GitHub]Git 시작"
date: 2021-12-22 19:31
categories: Git
---
# Git 설치

Git을 설치하기 위해 [git-scm.com/downloads](https://git-scm.com/downloads)로 이동하여 각 운영체제에 맞는 것을 설치한다.

또는 쉘에서 각 운영체제에 맞는 명령어를 통해 설치가 가능하다.([git - book/download](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%84%A4%EC%B9%98))

> Git은 GUI를 지원하기도 하지만 주로 CLI에서 사용하기 때문에 기본적으로 쉘을 다루는 방법을 숙지해야 한다.



# 원격 Repository 생성

[Git Hub](https://github.com/)에서 회원가입 후에 Your repositories 페이지의 New버튼을 눌러 원격 저장소를 생성한다.

> 옵션으로 공개, 비공개 설정을 할 수도 있고, 라이센스 파일이나 README.md파일을 자동으로 생성시킬 수 있다.



# Git 기초 명령어

- `git init`

  현재 디렉토리에 로컬 깃 저장소를 생성한다. 이 때 해당 디렉토리에는 .git디렉토리가 생성된다.

- `git clone [원격 저장소 주소]`

  원격 저장소의 내용을 복제하여 새로운 로컬 저장소를 생성된다.

- `git status`

  현재 디렉토리의 git 상태를 출력한다.

- `git add [파일 경로] ` 

  stage area로 업로드

- `git commit`

  로컬 저장소에 저장

- `git push`

  원격 저장소에 업로드

  

- `git pull`

  원격 저장소의 내용을 로컬저장소로 불러온다. 기존의 로컬저장소에 변경된 내용이 병합됨

- `git log`

  `--all --decorate --oneline --graph`
  
  

![files](/public/img/git-structure.JPG)









-----

##### 참고자료 : 

[git-scm.com](https://git-scm.com/book/ko/v2)

[inflearn - Git](https://www.inflearn.com/course/%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%EC%98%A8-git/dashboard)

[AWS CodeCommit](https://docs.aws.amazon.com/ko_kr/codecommit/latest/userguide/how-to-basic-git.html)

