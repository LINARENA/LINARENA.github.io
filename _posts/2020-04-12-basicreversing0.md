---

layout: post

current: post

cover: 'assets/images/reversing_intro/IMG_003.png'

navigation: True

title: Chapter 0. Intro

date: 2020-04-12 20:34:00

tags: Posts

class: post-template

subclass: 'post tag-windows'

author: Choonsik

---



### 들어가며







안녕하세요. 린아레나 춘식입니다. **리버싱**은 프로그램 흐름을 분석하여 흐름을 변경하거나 보완하는 기술입니다. 프로그램 개발 및 보안 연구 분야에서 꼭 필요한 영역이며, 심도있는 분석을 위해 꼭 익혀야 할 **오래되고 전통이 깊은** 기술이라 할 수 있겠습니다! 리버싱은 대게 사람이 알아보기 쉬운 코드(~~코드 조차 프로그래밍 언어를 모르면 힘들지만..또릐릐~~)가 아니라 컴퓨터가 해석하기 좋게 번역(?)된 코드를 보고 분석합니다. 그리고 리버싱 도구(~~혹은 디버깅 도구~~)도 필수적으로 필요하구요. 많은 사람들이 **IDA** 라는 강력한 도구를 사용하고, IDA 보다는 가볍지만 그렇다고 절대 무시할 수 없는 또 다른 강력한 도구인 **Ollydbg**를 사용합니다. 이번 포스팅은 정말 기본적인 내용을 다뤄볼 건데요. 윈도우 실행 프로그램을 직접 만들어보고, 그 프로그램을 Ollydbg를 이용하여 흐름을 재밌게 바꿔보도록 하겠습니다. 시작은 항상 어렵습니다만 **조금씩 천천히** 이 깊고 넓은 분야를 모험해보도록 합시다!

<br>



### Intro



본 편으로 들어가기 전에 위에서 말한 프로그램 흐름은 뭔지, 그 흐름을 바꾼다는 건 어떤 건지 간단하게 알아보겠습니다.

<br>



아래 그림을 볼까요? 프로그램을 실행하면 A라는 함수를 실행하고 어떤 조건식 맞는지 판단하고 0또는 1을 반환하는 아주 아주 간단한 프로그램입니다.



<figure>

  <img data-action="zoom" src='{{ "/assets/images/reversing_intro/IMG_001.png" | relative_url }}' alt='[그림 1]'>

  <figcaption><center>[그림 1] 프로그램 A의 흐름</center></figcaption>

</figure><br>



이런 흐름으로 프로그램이 동작하도록 하기 위해서는 "프로그래밍 언어" 라는 것을 이용해서 "코딩"을 해야하죠. 그렇다면 컴퓨터는 그 언어를 어떻게 해석할까요? 이런 개념에 대해서 조금 더 깊게 공부하려면 운영체제 이론이나 컴퓨터 구조 같은 것들을 잘 알아야 하지만 우선 갈 길이 멀기 때문에 그 부분은 따로 공부하시면 좋겠습니다. :) 컴퓨터는 **컴파일**이라는 과정을 거쳐 이 코드를 자신이 이해하기 쉬운 형태로 변환합니다. 아래 그림은 A 프로그램의 흐름을 조금 더 컴퓨터가 이해하기 쉬운 형태로 변환한 결과물입니다.



<figure>

  <img data-action="zoom" src='{{ "/assets/images/reversing_intro/IMG_002.png" | relative_url }}' alt='[그림 2]'>

  <figcaption><center>[그림 2] 컴퓨터가 이해하기 쉬운 형태의 프로그램 흐름</center></figcaption>

</figure><br>



우리는 다양한 방법으로 이런 흐름들을 파악하고 흐름을 바꾸거나, 저장된 데이터를 가져오는 등의 행위를 **직접** 해볼 생각입니다. 흐름을 바꾼다는 것은 아래 그림과 같이 조건이 맞으면 특정 위치로 이동되게끔 작성된 프로그램을 조건이 맞지 않으면 특정 위치로 이동하게끔 변경하는 등의 행위로 공부를 시작하겠습니다.



<figure>

  <img data-action="zoom" src='{{ "/assets/images/reversing_intro/IMG_002.png" | relative_url }}' alt='[그림 3]'>

  <figcaption><center>[그림 3] 프로그램 흐름 조작</center></figcaption>

</figure><br>



다음 챕터에서는 C++ 이라는 언어로 윈도우 실행프로그램을 만들어보겠습니다. 감사합니다.





























