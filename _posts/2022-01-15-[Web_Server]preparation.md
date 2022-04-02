---
layout: post
title: "[Web_Server] 준비"
date: 2022-01-15 19:20
category: Web_Server
---

# 라즈베리파이에 우분투서버 설치

라즈베리파이에 우분투서버를 설치하기 위해 부팅 이미지를 생성해야한다.

SD카드에 부팅 이미지를 생성하기 위해 https://www.raspberrypi.com/software/에서 Raspberry Pi Imager를 설치한다.

설치가 완료되면 SD카드가 두 개의 파티션으로 나누어지는데, 그 중에서system-boot라는 드라이브로 들어가서 network-config라는 이름의 파일의 내용을 아래와 같이 수정한다.

```yaml
# This file contains a netplan-compatible configuration which cloud-init
# will apply on first-boot. Please refer to the cloud-init documentation and
# the netplan reference for full details:
#
# https://cloudinit.readthedocs.io/
# https://netplan.io/reference
#
# Some additional examples are commented out below
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
wifis:
  wlan0:
    dhcp4: true
    optional: true
    access-points:
      "라즈베리파이가 사용할 와이파이 이름":
        password: "와이파이 비밀번호"
#      myworkwifi:
#        password: "correct battery horse staple"
#      workssid:
#        auth:
#          key-management: eap
#          method: peap
#          identity: "me@example.com"
#          password: "passw0rd"
#          ca-certificate: /etc/my_ca.pem
```

위의 내용을 저장하고 SD카드를 라즈베리파이에 삽입한 뒤에 전원을 연결하면 라즈베리파이가 와이파이에 자동으로 연결된다.





# 우분투 ssh 원격접속 설정

우분투가 설치된 라즈베리파이에 접속하기 위해 공유기 관리페이지로 접속해서 우분투가 현재 할당받은 내부ip를 확인한다.

> 공유기 관리페이지는 공유기 제조사마다 다르기 때문에, 구글링을 통해 자신의 공유기에 대한 관리페이지 접속 방법을 알아봐야 한다.

내 컴퓨터에서 아래의 명령어를 통해 라즈베리파이의 우분투계정으로 원격접속을 시도한다.

```shell
ssh ubuntu@[내부ip]
```

이 때 라즈베리파이의 우분투계정과 비밀번호의 초기값은 둘 다 ubuntu이다.

로그인 후에 바로 시키는대로 비밀번호를 바꿔준다.

> 대부분 우분투에는 Open SSH Server가 기본적으로 설치되어 있지만 만약 설치되지 않았다면 아래의 명령어를 통해 Open SSH Server를 설치해야 한다.
>
> ```shell
> sudo apt-get update
> sudo apt-get install openssh-server
> ```
>
> 만약 ssh가 실행되지 않는다면 아래의 명령어를 통해 ssh를 실행시켜야 한다.
>
> ```shell
> sudo service ssh enable
> sudo service ssh start
> ```





# ssh 포트 설정

아래의 명령어를 통해 sshd_config파일에서 #Port 22의 주석처리를 제거하여 22번 포트를 열어준다.

```bash
sudo vi /etc/ssh/sshd_config
```

> 보안상의 문제로 22번 포트를 사용하고 싶지 않다면 본인이 원하는 포트의 번호로 수정하여 다른 포트를 사용할 수 있다.



- 포트포워딩

  - 포트?

    포트는 

- 모뎀 브릿지설정

