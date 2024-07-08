---
title: 커밋 메시지 컨벤션
author: kymin
date: 2023-11-06 14:53
categories: [Git]
tags: [git]
---

## 커밋 메시지 컨벤션?

Git에서 커밋을 할 때 작성하게 되는 커밋 메시지에 대한 규칙

Git을 사용하는 가장 큰 목적 중 하나가 버전관리인 만큼 프로젝트에 대한 히스토리를 관리할 때에 커밋을 이해하기 쉽도록 가독성을 높이고 협업에 더욱 유용하도록 만들어졌다.

## 구조

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

### 제목(subject)

- Type(tag)를 통해 해당 커밋이 어떤 성격을 가지는지 표현
- 제목을 통해 커밋에 대해 간략하게 설명
- 제목은 50자 이내로 작성하며 마침표를 붙이지 않음
- 제목은 명령문 형태로 작성

>**Type(tag)종류**
>
>- `feat` : 새로운 기능 추가
>- `fix` : 버그 수정
>- `docs` : 문서 수정
>- `style` : 코드 포맷팅, 세미콜론 누락 , 함수 이름 변경, 파일 이름 변경 주석 삭제 등
>- `refactor` : 코드 리펙토링(버그 수정이나 기능 추가가 아닌 코드 변경)
>- `test` : 테스트 코드 추가 및 기존 테스트 코드 수정
>- `chore` : 유지 보수 작업, 빌드 업무 수정, 패키지 매니저 수정 등의 설정 변경

### 본문(body)

- 제목만으로 커밋에 대한 설명이 충분하다면 본문은 생략 가능
- 커밋에 대한 이유나 부연설명에 대해 작성
- 어떻게 작업했는지 보다는 어떤 것을 작업했는지, 왜 작업했는지 작성
- 제목과 구별하기 위해 공백 한 줄 밑에 작성

### 꼬리말(footer)

- 꼬리말 역시 생략 가능
- Issue tracker의 아이디를 기입하는데에 주로 사용

>**꼬리말에 들어갈 issue tracker 종류**
>
>- `Fixes` : 이슈 수정중 (아직 해결되지 않은 경우)
>- `Resolves` : 이슈를 해결했을 때 사용
>- `Ref` : 참고할 이슈가 있을 때 사용
>- `Related to` : 해당 커밋에 관련된 이슈번호 (아직 해결되지 않은 경우)

## 예시

```
docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```

```
feat(directive): ng:disabled, ng:checked, ng:multiple, ng:readonly, ng:selected

New directives for proper binding these attributes in older browsers (IE).
Added coresponding description, live examples and e2e tests.

Closes #351
```

```
Feat: "Add login API"        // 타입: 제목

로그인 API 개발               // 본문

Resolves: #123              // 꼬리말 => 이슈 123을 해결했으며,
Ref: #456                               이슈 456 를 참고해야하며,
Related to: #48, #45           현재 커밋에서 아직 이슈 48 과 45 가 해결되지 않았다.
```



-----

##### 참고자료 : 

[gist.github.com/stephenparish](https://gist.github.com/stephenparish/9941e89d80e2bc58a153#format-of-the-commit-message)

[velog.io/@msung99](https://velog.io/@msung99/Git-Commit-Message-Convension)
