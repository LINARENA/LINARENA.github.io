---
layout: post
current: post
cover: 'assets/images/windows/dll/0x00_1.png'
navigation: True
title: DLL Hijacking & Ghidra
date: 2019-03-17 10:18:00
tags: Posts
class: post-template
subclass: 'post tag-windows'
author: Joel
---

# DLL Hijacking & Ghidra

안녕하세요. Joel입니다.

방구석에 누워있다 갑자기 DLL Hijacking 포스팅을 올리라는 일거리를 받아서 글을 쓰게 되었습니다. 허허..

##### Overview

DLL Hijacking은 PE 프로그램이 의도하지 않은 external library(dll)을 실행하도록 만드는 것 입니다. 
이 공격은 DLL Search Order 때문에 취약한 PE 프로그램이 공격자가 작성한 DLL을 로드하고 실행하게 됩니다.
지금부터, 취약한 프로그램 분석과 Hijacking할 dll을 찾는 과정 등을 하나씩 살펴보도록 하겠습니다.

먼저, 오늘 포스팅에서 사용한 Tool과 환경구축 내용입니다.

 - Ghidra (jdk 11 이상, 64bit 필수)
 - PuTTY 0.65
 - Sysinternals suite
 - Visual Studio 2017

##### Find the dlls which are loaded by target application.

###### Procexp

PuTTY 0.65를 실행한 뒤, Sysinternals의 Procexp.exe를 통해 어떤 dll들을 사용하는지 확인해보겠습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_2.png" | relative_url }}' alt='[그림 1-1]'>
</figure>

Procexp.exe에서 [View]-[Lower Pane View]-[DLLs] 설정을 하면 PuTTY 프로그램이 로드한 dll 목록들을 확인할 수 있습니다.
그런데 이것만 가지고는 어떤 dll을 hijack할 수 있는지 알 수가 없습니다. 결과만 보이기 때문이죠.

그래서 이번에는 Sysinternals의 Procmon을 실행시켜서 dll이 로드되는 과정을 살펴보겠습니다.

###### Procmon

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_3.png" | relative_url }}' alt='[그림 1-2]'>
</figure>

Procmon을 실행한 후 필터를 위와 같이 설정합니다. 그럼 아래와 같이 PuTTY가 dll을 로드하는 과정에서 발생한 내용들을 볼 수가 있습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_4.png" | relative_url }}' alt='[그림 1-3]'>
</figure>

위 그림과 Procexp의 내용을 종합해보면 PuTTY는 WINMM.dll을 자신이 있는 디렉터리에서 먼저 찾아서 로드하려고 한다는 것을 알 수 있습니다.

하지만 해당 디렉터리에는 WINMM.dll이 없기에 결과적으로는 C:\Windows\SysWOW64\WINMM.dll을 로드하고 있습니다.

만약 해당 디렉터리에 이 WINMM.dll이 존재한다면 어떻게 될까요? ^^

아 물론, 여기서 해당 디렉터리에 low priv user가 Write Access가 가능하다는 전제하에 진행하는 것입니다.

만약, 해당 디렉터리가 C:\, C:\Program Files 등 Admin 권한이 필요한 경우에는 UAC 팝업이 뜨겠죠?

UAC 팝업은 쉽게 우회가 가능하나, 이 글의 범위를 넘어서므로 바탕화면 특정 디렉터리에서 진행합니다.

계속 갑니닷.

###### Find a Target Function with Ghidra

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_5.png" | relative_url }}' alt='[그림 1-4]'>
</figure>

Target Function을 Ghidra를 이용해서 찾아보겠습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_1.png" | relative_url }}' alt='[그림 1-5]'>
</figure>

[Symbol Tree] - [Imports] - [WINMM.dll]을 따라가면 PlaySoundA라는 함수를 찾을 수 있습니다. 쉽죠??

그럼 이제 우리가 해야할 일은 Hijacking에 사용할 dll을 만드는 것 입니다.
 
###### Make a DLL

	
	#include "stdafx.h"

	BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {

		switch (ul_reason_for_call) {
		case DLL_PROCESS_ATTACH:
			WinExec("calc", SW_NORMAL);
		case DLL_THREAD_ATTACH:
		case DLL_THREAD_DETACH:
		case DLL_PROCESS_DETACH:
			break;

		}
		return true;
	}

	extern "C" __declspec(dllexport) void PlaySoundA() {
		WinExec("calc", SW_NORMAL);
	}
	
위와 같이 PuTTY가 실행될 때 계산기를 실행하도록 새로운 dll을 생성했습니다.

###### Execute Target Program

아래와 같이, 같은 디렉터리에 방금 만든 dll을 위치시킨 후, 실행해보겠습니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_6.png" | relative_url }}' alt='[그림 1-6]'>
</figure>

<figure>
  <img data-action="zoom" src='{{ "/assets/images/windows/dll/0x00_7.png" | relative_url }}' alt='[그림 1-7]'>
</figure>

계산기가 잘 실행되네요. ^^

지금까지 DLL Hijacking에 대해서 알아봤습니다. 뿅!