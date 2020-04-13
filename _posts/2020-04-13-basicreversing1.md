---

layout: post

current: post

cover: 'assets/images/basic_reversing/IMG_000.png'

navigation: True

title: Chapter 1. 리버싱 기초(1)

date: 2020-04-13 17:34:00

tags: Posts

class: post-template

subclass: 'post tag-windows'

author: Choonsik

---




### 1. Ollydbg



##### 참조되는 문자열 직접 수정하기!




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





































