## 1. Ruby 설치
https://rubyinstaller.org/downloads/에서 자신의 OS Bit에 맞는 Ruby+Devkit을 설치합니다.
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_1.png)
&nbsp;
jekyll을 실행하기 위해서는 반드시 Devkit이 필요하기 때문에 with로 하시는 게 좋습니다.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_2.png)


설치 후 Ruby –v로 잘 설치 되었는지 확인하세요. 명령어가 동작하지 않을 시, 환경변수에 Ruby를 추가해주세요.


## 2. Jekyll & bundler 설치

&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_3.png)
&nbsp;
gem 명령어를 이용해서 jekyll과 bundler를 설치해줍니다.
&nbsp;
## 3. github branch clone
&nbsp;
github source branch의 blog_tmp에는 지금까지 블로그에 사용된 소스코드가 들어있습니다. clone하여 소스코드를 받아주세요.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_4.png)
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_5.png)

## 4. md 및 이미지 추가
&nbsp;
본인이 만든 md는 _posts 디렉터리에 업로드, 이미지는 assets/images에 추가해주시면 됩니다.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_6.png)
&nbsp;
#5. md 작성 시 필요한 사항들
&nbsp;
### 1) _data/authors.yml 작성
&nbsp;
_data/authors.yml 파일을 열어 다음과 같이 작성자 프로필 설정을 해주세요.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_7.png)
&nbsp;
### 2) md 설정 값 작성
&nbsp;
본인이 작성한 md 헤더에 다음과 같이 값을 넣어주세요.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_8.png)
&nbsp;
- cover : 블로그 포스팅 시, 글 도입부에 표시될 이미지
- title : 글 제목
- date : 글 작성일. (반드시 글 포스팅 하는 날짜를 기입해주세요. 날짜 순서로 포스팅됩니다.)
- author : 작성자 이름. authors.yml에 등록한 본인 이름을 적어주세요.
&nbsp;

## 6. Jekyll 동작 확인
&nbsp;
bundle exec jekyll serve 명령어를 통해 jekyll을 serve합니다.
&nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_9.png)
&nbsp;
http://127.0.0.1:4000/으로 접근하여 로컬에서 홈페이지가 잘 동작하는지 확인합니다.
* 빌드만 할 경우는 bundle exec jekyll build 명령어를 사용하면 됩니다.
* &nbsp;
![home page](https://raw.githubusercontent.com/LINARENA/LINARENA.github.io/source/assets/images/guide_10.png)
&nbsp;
## 7. 빌드된 파일들 github에 push
&nbsp;
빌드 또는 serve를 하면 상위 디렉터리에 Lin-Posts/가 생성됩니다. 해당 디렉터리에 있는 파일들을 master branch에 push하시면 블로그 업로드 완료가 됩니다.
&nbsp;
## 8. 소스코드 최신으로 유지하기
&nbsp;
빌드에 사용한 소스코드를 “반드시” Source Branch의 blog_tmp 하위에 업로드 해주시기 바랍니다.