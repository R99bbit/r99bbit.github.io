---
title: "[iOS] iPhone에 Frida 환경 세팅하기"
date: 2023-03-04 01:02:00 +0900
categories: [MOBILE, IOS]
tags: [mobile, ios, firda]
description: ios frida settings
---

## Why?
> 최근 맡은 프로젝트에서 iOS 애플리케이션을 분석할 일이 생겼다. 우열곡절 iOS 탈옥을 완료했고, 이제 frida를 세팅해보려고 한다.

---

## 분석 환경 구성하기

### 0. iPhone 탈옥하기
- 아무래도 frida를 사용하려면 탈옥, 루팅이 필요할 것이다. 바로 [이전 글](/posts/ios-jailbreak-16/)에서 iOS 탈옥에 대해 다뤘으니 선행하도록 하자.
  
### 1. MacBook에 frida-tools 설치하기

![](/assets/img/posts/230304-02-01.png)
- 터미널을 열고 `pip3 install frida-tools` 으로 frida-tools를 설치한다. 이는 `frida-ps`, `frida-trace` 등 유용한 도구들을 포함한다.

### 2. iPhone에 frida 설치하기

- iPhone이 탈옥되어 있다면 `Cydia`, `Sileo` 등 탈옥된 기기의 패키지 관리 애플리케이션이 있을 것이다.

![](/assets/img/posts/230304-02-02.re.png){: width="40%" height="40%"}

- `Sileo` 의 경우 [하단 네비게이션] > [소스] > [우측 상단 더하기] 에서 소스추가를 통해 frida의 repository인 `https://build.frida.re/` 를 추가해줘야 한다.

![](/assets/img/posts/230304-02-03.re.png){: width="40%" height="40%"}

- 이제 [하단 내비게이션] > [검색] > ['Frida' 검색] 을 통해 Frida를 기기에 설치한다.

### 3. 기기를 MacBook에 연결하고 frida attach
- `frida-ps -U` 로 usb 연결된 장치 프로세스 목록을 출력해보자.
![](/assets/img/posts/230304-02-04.png)
