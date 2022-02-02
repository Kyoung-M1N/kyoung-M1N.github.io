---
layout: post
title: "[Git & GitHub]Git 준비"
date: 2021-12-19 19:31
categories: Git
---
# Git & Git Hub

### Git?

Git은 리누스 토르발스(리눅스 만든 사람)가 만든 분산형 버전 관리 시스템(VCS : Version Control System)이다.

##### 특징

- 스냅샷(Snap Shot) 기반 버전 관리 시스템

  스냅샷은 특정 시점에서 파일, 폴더 또는 워크스페이스의 상태를 의미한다. Git에서는 새로운 버전을 기록하기 위한 명령인 커밋을 하면 스냅샷이 저장된다.

  기존의 다른 분산형 버전 관리 시스템은 프로젝트를 구성하는 각 파일들의 변화를 개별적으로  시간에 따라 관리(**델타 기반 버전관리 시스템**)하는 반면, Git은 시간순으로 프로젝트 전체의 스냅샷을 저장한다.이 때 성능을 위해 달라지지 않은 파일은 새로 저장하지 않고 이전 파일의 링크를 저장한다.

- 거의 모든 명령이 로컬에서 실행

  Git Hub나 Git Lab 같은 원격 저장소에 저장하지 않는 이상 Git에 의해 저장되는 프로젝트의 모든 히스토리들이 로컬 디스크에 존재하고, Git의 거의 모든 명령들이 로컬 파일과 데이터만 사용하기 때문에 오프라인 작업이 가능할 뿐만 아니라 네트워크의 상태와 상관없이 모든 명령이 순식간에 실행된다.

> 이외에도 더 많은 특징들과 장단점들이 존재한다.



### Git Hub?

Git Hub은 가장 큰 Git 저장소이며 전세계의 수많은 개발자들이 오픈 소스 프로젝트를 호스팅하고 협업을 진행한다.

##### 특징

- 협업

  각각의 repository에 대해 여러명의 협력자(Collaborator)를 등록하여 공동작업으로 프로젝트를 진행할 수 있다.

- GitHub Pages

  Git Hub에서 제공하는 웹호스팅 서비스로 Jekyll을 사용하거나 HTML/CSS/JS를 이용한 웹페이지를 repository에 호스팅하여 개인 홈페이지로 활용 가능하다.

- 

> 마찬가지로 이외에도 더 많은 특징들과 장단점들이 존재한다.



# Git 설치

깃을 설치하기 위해 [git-scm.com/downloads](https://git-scm.com/downloads)로 이동하여 각 운영체제에 맞는 것을 설치한다.

또는 쉘에서 각 운영체제에 맞는 명령어를 통해 설치가 가능하다.([git - book/download](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%84%A4%EC%B9%98))



-----

##### 참고자료 : 

[git-scm.com](https://git-scm.com/book/ko/v2)

[inflearn - Git](https://www.inflearn.com/course/%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%EC%98%A8-git/dashboard)

[pages.github.com](https://pages.github.com/)

