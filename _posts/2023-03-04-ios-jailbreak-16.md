---
title: "[iOS] iOS 16.3.1 Jailbreak (on MacOS)"
date: 2023-03-04 01:01:00 +0900
categories: [MOBILE, IOS]
tags: [mobile, ios, jailbreak]
description: ios jailbreak
---

## Why?
> 최근 맡은 프로젝트에서 iOS 애플리케이션을 분석할 일이 생겼다. APK의 경우 대충 시중에 널려있는 에뮬레이터 가져다가 (어차피 루팅 손쉽게 할 수 있으니깐 버튼 하나로) frida 설치하고 후킹 돌리면서 분석하면 되는데 iOS는 그렇게 간단한 문제가 아니더라. 대충 번개장터에서 아무 휴대폰이나 사서 다운그레이드 한 다음에 탈옥시키면 되는줄 알았다. 그런데 막상 중고로 산 핸드폰은 최신 업데이트 되어있었고, Sign 없는 버전은 설치 안되는 문제 등등 이리저리 치이다가 그냥 가장 최신인 iOS 16.3 에서 탈옥하는 방법을 택했다.

---

## 내 기기도 가능할까?

개발자에 의하면 `A8` 부터 `A11` 칩셋을 사용하는 모델까지 가능하다고 한다. 호환되는 기기 목록은 아래와 같다.
- iPhone 6s
- iPhone 6s Plus
- iPhone SE (2016)
- iPhone 7
- iPhone 7 Plus
- iPhone 8
- iPhone 8 Plus
- iPhone X
- iPad mini 4
- iPad Air 2
- iPad (5th generation)
- iPad (6th generation)
- iPad (7th generation)
- iPad Pro (9.7")
- iPad Pro (12.9") (1st generation)
- iPad Pro (10.5")
- iPad Pro (12.9") (2nd generation)
- iPod Touch (7th generation)

---

## 환경
![](/assets/img/posts/230304-01.png){: width="40%" height="40%"}

탈옥 대상 기기 스펙은 다음과 같다.
- 기기 : iPhone 8
- 모델 번호 : A1905(iPhone 8 GMS)
- 버전 : iOS 16.3.1

MacOS 스펙은 다음과 같다.
- 기기 : MacBook Air(M1, 2020)
- 모델 번호 : A2337
- 버전 : macOS Ventura 13.0

---

## 탈옥 과정

### 1. palera1n 다운로드
![](/assets/img/posts/230304-03.png)
[palera1n github repository](https://github.com/palera1n/palera1n-c/releases) 에서 **macos-universal** 바이너리를 다운로드 받는다.

### 2. 바이너리 권한 부여

다운받은 바이너리에 Execute Permission을 부여하도록 하자.

```
$ sudo mv ./palera1n-macos-universal /usr/local/bin/palera1n
$ ls -al /usr/local/bin/palera1n
-rw-r--r--@ 1 r99bbit  staff  15559360  3  4 00:24 /usr/local/bin/palera1n
$ sudo xattr -cr /usr/local/bin/palera1n
$ sudo chmod +x /usr/local/bin/palera1n
-rwxr-xr-x  1 r99bbit  staff  15559360  3  4 00:24 /usr/local/bin/palera1n
```

### 3. iPhone을 초기화하기
![](/assets/img/posts/230304-04.jpeg){: width="40%" height="40%"}

`A11` 칩셋이 탑재된 기기에서 `iOS 16.x` 를 사용중일 경우 탈옥을 위해서는 **암호를 설정한 적이 없어야 한다.** 만약 한번이라도 TouchID, passcode를 적용한 적이 있다면 iPhone에서 **설정 > 일반 > 전송 또는 iPhone 재설정 > 모든 콘텐츠 및 설정 지우기**를 통해 완전 초기화를 진행해줘야 한다. *(작업 진행 전에 iTunes 등으로 iPhone을 백업하도록 하자)*

iPhone 8 시리즈의 경우 A11 바이오닉 칩셋이 탑재되어 있기 때문에 상기 방법으로 초기화를 진행하였다.

### 4. iPhone을 MacBook에 연결하기

iPhone을 초기화 했다면 MacBook에 연결하도록 하자. 이 때 **USB A to Lightning을 사용해야 한다.** iPhone을 처음 샀을 때 주는 번들 케이블(USB C to Lightning Cable)을 사용할 경우 기기가 DFU 모드로 넘어가지 않는 이슈가 있기 때문이다. [checkra1n](https://twitter.com/checkra1n/status/1195452064958222337)도 이 이슈에 대해 언급했었다.

> 알다시피 MacBook에 USB A Type을 연결하려면 별도의 허브가 있어야 한다. 데이터 전송 과정에서 끊겨서 벽돌이 될 위험이나 이런걸 생각해보면 정품 허브를 사용하는 것이 안전해 보인다. 하지만 MacBook C Type 허브는 7-8만원 정도 한다. 아직 프로젝트 시작도 못했는데 돈이 꽤 깨졌다. 참 비싼 작업이다. (케이블 사고, 허브 사고, 기기 사고 ㅎㅎ...)

### 5. exploit

iPhone을 MacBook에 연결했다면 터미널을 열고 `palera1n` 을 실행하여 exploit을 수행하자. 과정은 아래와 같다.
1. `palera1n -v -c -f` 를 통해 fakefs를 생성
2. `palera1n -v -f` 를 통해 exploit을 수행

### 5.1. fakefs를 생성

- 우선 fakefs를 생성해야한다. 터미널에 `palera1n -v -c -f` 를 적어주자.

```
 - [03/04/23 03:06:19] <Info>: Waiting for devices
 - [03/04/23 03:06:19] <Verbose>: Normal mode device connected
 - [03/04/23 03:06:23] <Verbose>: Normal mode device disconnected
 - [03/04/23 03:06:29] <Verbose>: Recovery mode device 6869920866910254 connected
 - [03/04/23 03:06:30] <Info>: Press Enter when ready for DFU mode
```

- 그러면 기기를 찾은 뒤(포트에 연결 되어있어야 함) DFU 모드에 진입할 준비가 되면 Enter를 누르라고 나온다.


![](/assets/img/posts/230304-05.png){: width="40%" height="40%"}

- 여기까지 오면 연결된 iPhone이 자동으로 종료된 뒤에 위 그림처럼 리커버리 모드에 진입한다. 이제 Enter를 쳐주면 되는데, **DFU에 진입하기 위해 기기 버튼을 눌러줘야 하니** Get ready.. 라는 문구가 나올 때 준비를 해줘야 한다.

```
Hold volume down + side button (0)
Hold volume down button (3) 
 - [03/04/23 03:06:47] <Verbose>: DFU mode device 6869920866910254 connected
 - [03/04/23 03:06:47] <Info>: Device entered DFU mode successfully
 - [03/04/23 03:06:47] <Info>: About to execute checkra1n
```

- 개발자는 친절하게도 DFU 모드 진입을 준비할 시간을 3초 준다. 3, 2, 1 카운트에 맞춰 DFU 모드에 진입하면 되는데 iPhone 8의 경우 [전원 버튼 + 볼륨 다운] > [볼륨 다운] 을 DFU에 진입할 때 까지 눌러주면 된다. 자세한 내용은 터미널 Log에 찍혀 나온다. (혹은 DFU 진입 방법 검색)


![](/assets/img/posts/230304-06.jpeg){: width="60%" height="60%"}

![](/assets/img/posts/230304-07.jpeg){: width="60%" height="60%"}

- 이후 자동으로 Exploit을 위한 PongoOS로 진입하게 되고 가만히 냅두면 위와 같이 fakefs 생성이 진행된다. 5분정도 소요되며, 완료 후 자동으로 기기가 재부팅된다.


- full log는 아래와 같다.

```
# == palera1n-c ==
#
# Made by: Nick Chan, Tom , Mineek, Nebula, llsc12
#
# Thanks to: dora2ios, pythonplayer, tihmstar, nikias
# (libimobiledevice), checkra1n team (Siguza, axi0mx, littlelailo
# et al.), Procursus Team (Hayden Seay, Cameron Katri, Keto et.al)

 - [03/04/23 03:06:19] <Info>: Waiting for devices
 - [03/04/23 03:06:19] <Verbose>: Normal mode device connected
 - [03/04/23 03:06:19] <Info>: Telling device with udid 876f7ee96ef6832f9cfccfa39db4b0867f219188 to enter recovery mode immediately
 - [03/04/23 03:06:23] <Verbose>: Normal mode device disconnected
 - [03/04/23 03:06:29] <Verbose>: Recovery mode device 6869920866910254 connected
 - [03/04/23 03:06:30] <Info>: Press Enter when ready for DFU mode

Get ready (0)
Hold volume down + side button (2) - [03/04/23 03:06:38] <Verbose>: Recovery mode device disconnected
Hold volume down + side button (0)
Hold volume down button (3) - [03/04/23 03:06:47] <Verbose>: DFU mode device 6869920866910254 connected

 - [03/04/23 03:06:47] <Info>: Device entered DFU mode successfully
 - [03/04/23 03:06:47] <Info>: About to execute checkra1n
#
# Checkra1n 0.1337.1
#
# Proudly written in nano
# (c) 2019-2023 Kim Jong Cracks
#
#========  Made by  =======
# argp, axi0mx, danyl931, jaywalker, kirb, littlelailo, nitoTV
# never_released, nullpixel, pimskeks, qwertyoruiop, sbingner, siguza
#======== Thanks to =======
# haifisch, jndok, jonseals, xerub, lilstevie, psychotea, sferrini
# Cellebrite (ih8sn0w, cjori, ronyrus et al.)
#==========================

 - [03/04/23 03:06:48] <Verbose>: Starting thread for Apple TV 4K Advanced board
 - [03/04/23 03:06:48] <Info>: Waiting for DFU mode devices
 - [03/04/23 03:06:48] <Verbose>: DFU mode device found
 - [03/04/23 03:06:48] <Info>: Checking if device is ready
 - [03/04/23 03:06:48] <Verbose>: Attempting to perform checkm8 on 8015 11
 - [03/04/23 03:06:48] <Info>: Setting up the exploit
 - [03/04/23 03:06:48] <Verbose>: == checkm8 setup stage ==
 - [03/04/23 03:06:48] <Verbose>: Entered initial checkm8 state after 1 steps
 - [03/04/23 03:06:48] <Verbose>: Stalled input endpoint after 1 steps
 - [03/04/23 03:06:48] <Verbose>: DFU mode device disconnected
 - [03/04/23 03:06:48] <Verbose>: DFU mode device found
 - [03/04/23 03:06:48] <Verbose>: == checkm8 trigger stage ==
 - [03/04/23 03:06:51] <Info>: Checkmate!
 - [03/04/23 03:06:51] <Verbose>: Device should now reconnect in download mode
 - [03/04/23 03:06:51] <Verbose>: DFU mode device disconnected
 - [03/04/23 03:06:57] <Info>: Entered download mode
 - [03/04/23 03:06:57] <Verbose>: Download mode device found
 - [03/04/23 03:06:57] <Info>: Booting PongoOS...
 - [03/04/23 03:06:59] <Info>: Found PongoOS USB Device
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'fuse lock'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'sep auto'
 - [03/04/23 03:06:59] <Verbose>: Uploaded 115504 bytes to PongoOS
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'modload'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'kpf_flags 0x1'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'checkra1n_flags 0x0'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'palera1n_flags 0x5'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'rootfs'
 - [03/04/23 03:06:59] <Verbose>: Uploaded 524288 bytes to PongoOS
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'ramdisk'
 - [03/04/23 03:06:59] <Verbose>: Uploaded 5821922 bytes to PongoOS
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'overlay'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'xargs  wdt=-1 rootdev=md0'
 - [03/04/23 03:06:59] <Verbose>: Executing PongoOS command: 'bootx'
 - [03/04/23 03:06:59] <Info>: Booting Kernel...
```

### 5.2. 탈옥을 진행
- 여기까지 잘 따라왔으면 iPhone이 별 다른 문제 없이 바탕화면을 보여주고 있을 것이다.
- 이번엔 터미널에 `palera1n -v -f` 를 적어주자. 아까와 다른점은 fakefs를 생성해주는 옵션인 `-c` 가 빠졌다.
- 리커버리 진입, DFU 진입 등 과정은 *5.1.* 과 동일하게 진행하면 된다.

### 6. 기기에서 palera1n 앱 실행 및 설치 작업
![](/assets/img/posts/230304-08.PNG){: width="40%" height="40%"}
- 정상적으로 설치 되었다면 홈 화면에 위와 같이 "palera1n" 애플리케이션이 설치 되어있을 것이다.

![](/assets/img/posts/230304-09.PNG){: width="40%" height="40%"}
- "palera1n" 을 실행하고 "install"

![](/assets/img/posts/230304-10.PNG){: width="40%" height="40%"}
- "Respring" 을 클릭하여 탈옥을 마무리

### 7. 축하합니다! 탈옥 성공~!!
![](/assets/img/posts/230304-11.PNG){: width="40%" height="40%"}
- 탈옥이 정상적으로 완료되었다.
- 홈 화면에 생성된 "Sileo" 애플리케이션을 통해 다양한 Tweak을 설치해볼 수 있다.

### 8. 주의할 점 한가지
- 앞서 A11 칩셋 이후 모델은 TouchID나 passcode가 한번이라도 설정 되었다면 안된다고 언급했었다.
- 이는 탈옥 이후에도 마찬가지다. 암호설정 절대절대 금지이다

---

## Trouble Shooting

### 1. DFU 진입이 안된다.
- 이는 앞서 언급한 케이블 문제일 가능성이 크며 C to Lightning 즉, 기기 구매시 번들로 오는 케이블을 이용했을 때 DFU에 진입하지 못할 수 있다. 이는 해당 레포지토리 README 에서도 언급한 바 있다.
> The BootROM will only enter DFU if it detects USB voltage, which boils down to checking whether a certain pin is asserted from the Tristar chip. The Tristar does this based on the cable's accessory ID, and apparently USB-A and USB-C cables have different accessory IDs, and the one of the USB-C cables makes the Tristar not assert the USB voltage pin.

### 2. PongoOS 진입 후 특정 로그만 계속 뜨고 재부팅이 안된다.
![](/assets/img/posts/230304-12.png){: width="40%" height="40%"}
- 위와 같은 문구가 계속 뜨면서 재부팅이 안된다면 다음과 같은 두개의 문제를 의심해볼 수 있다.

![](/assets/img/posts/230304-13.png){: width="60%" height="60%"}
- 첫째, 사용하고 있는 케이블이 문제 일 수 있다. 실제로 정품 케이블로 바꾸자 귀신같이 해결되었다.

![](/assets/img/posts/230304-14.png){: width="60%" height="60%"}
- 둘째, fakefs 생성 과정에서 문제가 있을 수 있다. 나의 경우 create fakefs 할 때 모종의 이유로 `-c` 옵션이 아닌, `-B` 옵션을 사용 했었는데(bindfs) 이는 iOS 16에서는 더이상 제대로 동작하지 않는 기능이기 때문에 벽돌이 됐었다.

### 3. 이외의 문제들..
- README에서 확인해볼 수 있는 "탈옥이 실패할 수 있는 케이스" 들을 살펴보면
  - AMD CPU에서는 USB 컨트롤러 문제(아마도) 때문에 익스에 실패할 수 있으니 INTEL이나 MACOS에서 시도해라
  - 해당 익스는 `A8` 에서 `A11` 칩셋까지 가능함
  - (앞서 기술한) USB C 타입 케이블 사용

### 4. 탈옥에 실패한다면?
- 탈옥에 실패하면 `palera1n --force-revert -f` 를 실행하여 fakefs 제거한 뒤 다시 시도
- DFU 등에서 핸드폰이 멈추면 강제 재부팅(볼륨 업 > 볼륨 다운 > 전원 길게 누르기) 해보기

---

## Reference
- [참고 링크](https://www.iclarified.com/88666/how-to-jailbreak-iphone-using-palera1n-ios-1631-mac)
- [palera1n](https://github.com/palera1n/palera1n-c)

