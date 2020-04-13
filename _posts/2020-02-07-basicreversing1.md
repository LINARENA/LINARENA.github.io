---
layout: post
current: post
cover: 'assets/images/basic_reversing/IMG_000.png'
navigation: True
title: Chapter 1. 리버싱 기초(1)
date: 2020-04-12 20:34:00
tags: Posts
class: post-template
subclass: 'post tag-windows'
author: Choonsik
---

### 리버싱 이란?



안녕하세요. 린아레나 춘식입니다. **리버싱**은 프로그램 흐름을 분석하여 흐름을 변경하거나 보완하는 기술입니다. 프로그램 개발 및 보안 연구 분야에서 꼭 필요한 영역이며, 심도있는 분석을 위해 꼭 익혀야 할 **오래되고 전통이 깊은** 기술이라 할 수 있겠습니다! 리버싱은 대게 사람이 알아보기 쉬운 코드(~~코드 조차 프로그래밍 언어를 모르면 힘들지만..또릐릐~~)가 아니라 컴퓨터가 해석하기 좋게 번역(?)된 코드를 보고 분석합니다. 그리고 리버싱 도구(~~혹은 디버깅 도구~~)도 필수적으로 필요하구요. 많은 사람들이 **IDA** 라는 강력한 도구를 사용하고, IDA 보다는 가볍지만 그렇다고 절대 무시할 수 없는 또 다른 강력한 도구인 **Ollydbg**를 사용합니다. 이번 포스팅은 정말 기본적인 내용을 다뤄볼 건데요. 윈도우 실행 프로그램을 직접 만들어보고, 그 프로그램을 Ollydbg를 이용하여 흐름을 재밌게 바꿔보도록 하겠습니다. 시작은 항상 어렵습니다만 **조금씩 천천히** 이 깊고 넓은 분야를 모험해보도록 합시다!
<br>

### 1. 시작!


윈도우 32bit 환경에서 프로그램을 작성하였습니다. Ollydbg는 64bit를 지원하지만 시중에 널리 알려진 문제들이 32bit 기반으로 작성되어 있어, 32bit에서 작성하였습니다. 모든 부분은 Step 으로 구성되어 있으며, 정말 하나씩 따라 해보셔도 되고 복습을 하시는 분들이라면 천천히 머릿 속에 Step을 따라 이해해보시면 좋겠습니다.

++파일이 필요하신 분은 ssw@linarena.com 으로 메일 주시면 보내드리겠습니다.++

<br><br>
**Step 1.** 윈도우 실행 프로그램은 C++ 기반으로 작성되었습니다. C++ 개발 환경에 최적화된 **DEV C++** 라는 프로그램을 이용해서 프로젝트를 새로 생성해보겠습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_001.png" | relative_url }}' alt='[그림 1-1]'>
  <figcaption><center>[그림 1-1] DEV C++ 프로젝트 생성(1)</center></figcaption>
</figure><br>

**Step 2.** 프로젝트를 생성할 때 프로그램 타입은 Basic > Console Application 을 선택합니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_002.png" | relative_url }}' alt='[그림 1-2]'>
  <figcaption><center>[그림 1-2] DEV C++ 프로젝트 생성(2)</center></figcaption>
</figure><br>

**Step 3.** 프로젝트를 생성하면 가장 기본 뼈대가 되는 코드를 개발 환경에서 알아서 작성해줍니다. 이제 이 부분을 우리가 원하는 대로 수정해봅시다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_003.png" | relative_url }}' alt='[그림 1-3]'>
  <figcaption><center>[그림 1-3] DEV C++ 프로젝트 생성 결과</center></figcaption>
</figure><br>

**Step 4.** 프로그램 실행 시 "Hello Reversing" 이라는 문자열이 설정된 조그만한 알림창을 띄워볼까요? 그 코드는 아래와 같습니다. 코드는 이미지로 첨부했으니 직접 한 번 따라쳐보아요!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_004.png" | relative_url }}' alt='[그림 1-4]'>
  <figcaption><center>[그림 1-4] linarena_reversing1.cpp 예제 코드</center></figcaption>
</figure><br>

**Step 5.** 이제 우리는 C++ 언어(사람이 알아보기 좋은 언어)로 작성된 프로그램을 번역(컴파일)하는 과정을 거쳐야 하는데, 그 과정에서 문법적으로 문제가 없는지 확인할 수 있습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_005.png" | relative_url }}' alt='[그림 1-5]'>
  <figcaption><center>[그림 1-5] DEV C++ 컴파일 방법</center></figcaption>
</figure><br>

**Step 6.** 프로그램 문법 오류 등의 문제가 발생하지 않는 다면 Error 부분이 0으로 표시됩니다. 정상적으로 컴파일 되는 걸 보니 아무래도 잘 완성한 것 같네요!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_006.png" | relative_url }}' alt='[그림 1-6]'>
  <figcaption><center>[그림 1-6] DEV C++ 컴파일 결과</center></figcaption>
</figure><br>

**Step 7.** 이제 이 녀석을 exe 확장자를 가진 실행 프로그램으로 **추출** 하기 위해서 상단메뉴의 Execute > Run 을 선택합니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_007.png" | relative_url }}' alt='[그림 1-7]'>
  <figcaption><center>[그림 1-7] 실행 프로그램 추출</center></figcaption>
</figure><br>

**Step 8.** 생성된 프로그램을 실행하니 알림창(이하로는 메시지 박스라고 표현하겠습니다..!)이 잘뜨네요! 예!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_008.png" | relative_url }}' alt='[그림 1-8]'>
  <figcaption><center>[그림 8] linarena_reversing1.exe. 실행 결과</center></figcaption>
</figure><br>

### 2. Ollydbg

##### 참조되는 문자열 직접 수정하기!
<br>

**Step 1.** Ollydbg 프로그램을 실행하고 상단 메뉴 File > Open 으로 생성한 실행 프로그램을 엽니다. Ollydbg 고녀석 참 복잡하게 생겼습니다! 하나 하나 다 책보듯이 외우지 말고 만화 속 등장인물 들 이름 외우듯 만날 때마다 이름을 소개하도록 하겠습니다. 지금은 프로그램이 실행될 때 우리가 C++로 작성했던 함수가 어디서 호출되는지 찾아보도록 하겠습니다. 가장 왼쪽 상단이 코드 윈도우 입니다. 코드 윈도우에 마우스 포인터를 놓고우클릭 > Search for > All intermodular calls 를 선택하면 창이 하나 새로 열립니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_009.png" | relative_url }}' alt='[그림 2-1]'>
  <figcaption><center>[그림 2-1] 프로그램 실행 간 호출되는 함수 목록 찾기(1)</center></figcaption>
</figure><br>

**Step 2.** 새로 열린 창에서 MessageBoxA 라는 함수가 호출되는 부분이 보이네요. 이부분을 더블 클릭해서 따라가 봅시다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_010.png" | relative_url }}' alt='[그림 2-2]'>
  <figcaption><center>[그림 2-2] 프로그램 실행 간 호출되는 함수 목록 찾기(2)</center></figcaption>
</figure><br>

**Step 3.** CALL.. 뭐라고 적힌 거 같은데 이 부분이 호출하는 부분처럼 보이네요! MessageBoxA 가 보이는 위치를 클릭하고 F2 키를 이용하여 브레이크 포인트를 설정합시다. 브레이크 포인트를 설정하면 프로그램이 잘 실행되다가 이 설정된 위치에서 딱 멈춥니다. Ctrl + F2 단축키로 프로그램을 재실행해봅시다.(~~뭔가 뜨면 그냥 확인!~~) 브레이크 포인트가 설정된 상태에서 계속 됩니다. 더 나가셨다면 다시 Ctrl + F2!!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_011.png" | relative_url }}' alt='[그림 2-3]'>
  <figcaption><center>[그림 2-3] MessageBoxA 함수 브레이크 포인트 설정</center></figcaption>
</figure><br>

**Step 4.** 이제 시선을 돌려 우측 하단의 창을 봅시다. 이 녀석의 이름은 스택윈도우입니다. 스택 윈도우를 봅시다. Ollydbg 안에서 우리가 바꾸려고 하는 문자열을 처음 만났네요.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_012.png" | relative_url }}' alt='[그림 2-4]'>
  <figcaption><center>[그림 2-4] 스택 윈도우에 표시된 메시지 박스 내 문자열</center></figcaption>
</figure><br>

**Step 5.** 이제 이 녀석 위에 마우스 포인터를 올리고 우클릭 > Follow in Dump를 선택합시다!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_013.png" | relative_url }}' alt='[그림 2-5]'>
  <figcaption><center>[그림 2-5] 문자열이 저장된 스택 Dump </center></figcaption>
</figure><br>

**Step 6.** Follow in Dump를 클릭하면 좌측 하단의 창에 내용이 바뀝니다. 이 녀석의 이름은 덤프 윈도우입니다. 덤프 윈도우에 우리가 변경하려고 하는 문자열이 표시됐습니다. 바꾸려고 하는 문자열을 드래그 하고 Ctrl + E 단축키를 눌러봅시다!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_014.png" | relative_url }}' alt='[그림 2-6]'>
  <figcaption><center>[그림 2-6] 덤프 윈도우에 표시된 문자열 </center></figcaption>
</figure><br>

**Step 7.** 새로운 창이 나타났습니다. ASCII 라는 부분에 새로운 문자열을 입력해볼까요? 저는 Hello i2sec!!!! 으로 해보겠습니다. 아, 여기서 중요한 점은 글자수는 맞춰야 합니다. 프로그램은 지금 참조하는 문자열 부분만큼 공간을 채워놓았고 우린 그 부분의 내용물만 쏙 빼서 바꾸는 것이기 때문에 원래 문자열보다 길거나 짧으면 에러가 발생합니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_015.png" | relative_url }}' alt='[그림 2-7]'>
  <figcaption><center>[그림 2-7] 문자열 수정 </center></figcaption>
</figure><br>

**Step 8.** OK를 선택하고 밖으로 빠져나오면 바뀐 부분이 빨간색으로 표시됩니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_016.png" | relative_url }}' alt='[그림 2-8]'>
  <figcaption><center>[그림 2-8] 프로그램 내 변경된 내용 표시 </center></figcaption>
</figure><br>

**Step 9.** 이제 다시 F9 단축키를 이용하여 브레이크 포인트 이후 코드를 쭉! 실행하도록 하면 메시지 박스에 변경된 내용이 표시되는 것을 확인할 수 있었습니다~ 프로그램 흐름! 변경! 성공! BAAAM!

<figure>
  <img data-action="zoom" src='{{ "/assets/images/basic_reversing/IMG_017.png" | relative_url }}' alt='[그림 2-9]'>
  <figcaption><center>[그림 2-9]  </center></figcaption>
</figure><br>


다음 시간에는 참조되는 문자열을 직접 수정하지 않고, 참조될 문자열의 주소를 변경하여 다른 문자열이 메시지 박스에 표시되도록 해보겠습니다. 감사합니다.


















