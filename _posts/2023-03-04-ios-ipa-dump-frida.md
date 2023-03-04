---
title: "[iOS] iOS-Encrypted 된 IPA 복호화하기"
date: 2023-03-04 01:03:00 +0900
categories: [MOBILE, IOS]
tags: [mobile, ios, firda]
description: ios frida settings
---

## Why?
> 최근 맡은 프로젝트에서 iOS 애플리케이션을 분석할 일이 생겼다. 점점 하다보니 왜 애플이 자기네들 제품/솔루션에 어마어마한 바운티를 걸었는지 알 것 같다. 그냥 간단한 애플리케이션 추출 과정도 번거롭고 번거롭다... iOS 애플리케이션 패키지인 IPA는 안드로이드 APK와 다르게 바로 추출한 파일을 Decompile 했을 때 분석을 할 수 없게 Instruction 들을 뭉개놓았다. 이를 해체하는 작업을 해보자.

---

## 그냥 추출하면?
![](/assets/img/posts/230304-03-01.png)
![](/assets/img/posts/230304-03-02.png)
- 아쉽게도 탈옥 없이, 덤프 없이, 복호화 없이는 위와 같이 다 뭉개진 바이너리를 던져주는 것을 알 수 있다.
- 애초에 IDA에서 "이건 iOS Encrypted 되어있는데 갠춘?" 이라고 물어본다.
- frida를 이용해서 복호화 된 IPA를 덤프해보자

---

## 덤프하기

### 0. 사전 작업
- frida-ios-dump를 이용하기 위해서는 [탈옥](/posts/ios-jailbreak-16/)과 [frida 설치](/posts/ios-setting-frida)가 선행되어야 한다.

### 1. frida-ios-dump 및 Dependency 설치
```
$ git clone https://github.com/AloneMonkey/frida-ios-dump.git
```
- Git에서 `frida-ios-dump`를 내려받도록 하자.
  
```
$ cd frida-ios-dump/
$ pip3 install -r requirements.txt
```
- 이후 해당 repo를 이용하기 위해 설치해야 되는 python 패키지를 다운로드 받자.

### 2. dump.py 수정

```python
User = 'root'
Password = 'alpine'
Host = 'localhost'
Port = 2222
KeyFileName = None
```
- `dump.py` 를 열어보면 위와 같은 코드가 있을 것이다.
- 이는 iPhone의 SSH 정보를 적는 것이며, 알맞게 수정하도록 하자. (openssh-server가 설치되어 있어야 한다.)

### 3. 덤프할 Package Name 확인
![](/assets/img/posts/230304-03-03.png)
- `frida-ps -Uai` 를 통해 덤프할 패키지의 이름을 확인하도록 하자.

### 4. 덤프하기
![](/assets/img/posts/230304-03-04.png)
- `python3 dump.py [패키지 명]` 을 통해 덤프를 진행해주자.
- 나의 경우 대상 애플리케이션이 탈옥을 탐지하면 Splash 이미지만 띄워주고 바로 튕기는 솔루션이 걸려있었기 때문에 몇 가지 꼼수를 이용하여 IPA가 메모리에 올라올 때 까지 기다리게 만들었다.

![](/assets/img/posts/230304-03-05.png)
- 인내심을 가지고 조금 기다려보면 IPA를 현재 디렉토리에 만들어준다.

---

## 결과는?
![](/assets/img/posts/230304-03-06.png)
- 아까와 다르게 아주 깔끔하게 Decompile 되는 것을 확인할 수 있다.