---
layout: post
current: post
cover: 'assets/images/linux/jobs/CVE-2019-5736(docker eunC)/5736-3.png'
navigation: True
title: CVE-2019-5736 (runC Container 취약점)
date: 2019-03-24 06:18:00
tags: Posts
class: post-template
subclass: 'post tag-linux'
author: Sulla
---

## CVE-2019-5736 (runC Container 취약점)

안녕하세요. Sulla임돠!

2월 11일 **docker**관련 런타임인 **runC** 관련 취약점 **CVE-2019-5736** 공개 되었습니다. 
공개된지 1주일도 안지나 **POC**가 공개되고 있으며, 영향력도 큰 취약점으로 많은 주목을 받고있습니다.

그래서 이번 포스팅에서는 해당 취약점에 대하여 리뷰해 보겠습니다.

### **본 포스팅에서는 환경 구축/버전 등의 정보는 제외하겠습니다.**

먼저 **docker**가 뭔지...**runC**가 뭔지에 대해서 알아야겠죠??언제나 그렇듯 최대한 간단하고 필요한 내용만 쉽게 알아보겠습니다.(**귀차니즘** 아닙니다.....)

우선 **docker**란 **Linux** 기반의 **container runtime 오픈소스 플랫폼** 입니다. 쉽게 말해 **Container 관리 플랫폼**이며, **VM**(Virtual Machine)과 비슷한 역할을 수행하지만 쉬운 배포 및 높은 확장성 등의 **접근성**과 메모리 점유등의 **성능**의 차이를 보입니다.("**가상화**"가 목표이긴 합니다.) 

**runC**는 **container**의 **생성/실행** 등을 위한 **CLI** 도구입니다. 즉, **container**의 **조작**을 위해 존재 한다고 생각하면 될 것 같습니다.

그럼 이번 포스팅의 주인공인 **CVE-2019-5736**에 대해서 알아보겠습니다.

# CVE-2019-5736

해당 취약점은 위에서 알아본 **runC**에서 발생하는 취약점입니다. ****컨테이너 내부에서 **루트 권한**으로 악의적인 프로세스를 실행할 경우 **runC 버그**를 이용하여 컨테이터를 실행하는 **호스트에 대한 루트 권한**을 탈취하는 취약점입니다. 따라서, 서버에 대한 접근 및 해당 서버의 다른 컨테이너 또한 접근이 가능합니다. 

쉽게 말해 **악성 컨테이너**를 통해 **호스트 서버의 루트 권한 탈취**가 가능하게 됩니다.

해당 취약점의 트리거 방법은 다음중 하나 입니다.

- **컨테이너 내부의 악성 프로세스 실행(루트로 실행되는 )**
- **악성 docker 이미지를 실행**

해당 취약점을 트리거 하기 위해서는 위에서도 언급했 듯 **컨테이너 내부**에 **루트 권한**이 있어야 합니다. 

화면 구성은 위쪽은 **Host Server(이하 HS라 하겠습니다.)**, 아래쪽은 **Container** 입니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/CVE-2019-5736(docker eunC)/5736-1.png" | relative_url }}' alt='[그림 5736-1]'>
</figure>

첫 번째 방법 부터 보도록 하죠. 진행 자체는 굉장히 간단하게 진행됩니다.(물론 저는 삽질 했습니다......Aㅏ.....) 먼저 **HS**쪽에 해당 파일이 없음을 확인 합니다. 후에는 컨테이너에서 악성 파일을 실행하면 준비가 끝납니다. 함정 처럼 **/bin/sh**를 **#!/proc/self/exe** 바꿔놓고 기다리는 단계입니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/CVE-2019-5736(docker eunC)/5736-2.png" | relative_url }}' alt='[그림 5736-2]'>
</figure>

그리고 호스트가 컨테이너쪽으로  **docker exec** 명령을 통해 **/bin/sh**의 실행을 명하게 되면 위에서 설치해뒀던 함정이 발동하게 됩니다. 함정 카드..... **(/bin/sh를 쓸것이라 추측하고 세팅 하는겁니다.)** 
다음으로 **runC**의 **PID**를 찾고 **핸들링(O_PATH ,O_WRONLY 플래그 사용)**을 위한 여러 과정을 거치고 최종적으로 호스트의 **runC** 바이너리 파일을 **악성 runC** 로 변조되며 악성 파일에 미리 정의 되있던 행위를 **호스트**쪽에서 **루트 권한**으로 수행하게 됩니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/CVE-2019-5736(docker eunC)/5736-3.png" | relative_url }}' alt='[그림 5736-3]'>
</figure>

처음에는 없던 shadow 파일이 root 권한으로 생성된 것을 확인할 수 있습니다. 해당파일을 열람시 실제 shadow 파일임을 확인 가능합니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/CVE-2019-5736(docker eunC)/5736-4.png" | relative_url }}' alt='[그림 5736-4]'>
</figure>

위 과정을 거치면서 **HS**쪽의 runC 파일이 변조됩니다. 해쉬값을 비교하면 쉽게 확인 가능하며 위쪽 해쉬값이 원본이면 아래쪽은 변조된 runC 바이너리의 해쉬값입니다.

두 번째 방법의 경우는 첫 번쨰와 원리는 같으며 차이점은 악성 docker 이미지를 생성해서 배포하고 피해자는 해당 이미지를 다운받아 실행하게 되면 위에서와 같은 일련의 동작들을 수행하게 됩니다.

보신것처럼 **Container** 쪽에서 **HS**의 **root** 권한으로 악의적인 행위가 가능한 취약점입니다.

# 대응 방법

취약점의 동작 조건중 하나는 **Container**에의 **root**권한이 있어야 합니다. 또한 **출처를 알 수 없는 이미지**를 무분별하게 사용시에도 공격 시나리오가 생길것입니다. 
전부는 아니지만 대부분의 Cloud Container system이 **CVE-2019-5736** 취약할 것 입니다. 

1. **Container root 권한 제한**
2. **신뢰할 수 없는 이미지 파일 사용 자제**
3. **runC 등 최신 버전 유지**
4. .....

해당 취약점에 영향을 받는 여러 업체들(Red Hat, runC 관리자, google, Amazon, Docker, debian, ubuntn 등등)은 취약 runC 버전에 대해서 업데이트 한 이미지를 배포중입니다.

영어 해석에 재능이 부족하여 틀린 내용이 있다면 알려주시기 바랍니다 :)

감사합니다. 

## References 참고 문헌

- https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html
- https://brauner.github.io/2019/02/12/privileged-containers.html
- https://www.helpnetsecurity.com/2019/02/12/runc-container-escape-flaw/
- https://kubernetes.io/blog/2019/02/11/runc-and-cve-2019-5736/
- https://vulmon.com/exploitdetails?qidtp=EDB&qid=46369
- https://github.com/lxc/lxc/commit/6400238d08cdf1ca20d49bafb85f4e224348bf9d
- https://github.com/rancher/runc-cve
- https://github.com/Frichetten/CVE-2019-5736-PoC
- https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html