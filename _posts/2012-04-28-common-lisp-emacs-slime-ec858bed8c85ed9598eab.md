---
layout: post
title: Common Lisp, Emacs, slime 셋팅하기
date: '2012-04-28T12:54:24+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201006901/common-lisp-emacs-slime-ec858bed8c85ed9598eab
---
Common Lisp 개발을 위해서 에디터를 Emacs를 사용해보자.
처음 사용해보는 에디터 Emacs는 기존 에디터들과 다르게 CL개발을 위해서 환경을 제공해준다.

1. Emacs
윈도우 버전 : <a href="http://www.gnu.org/software/emacs/windows/Getting-Emacs.html#Getting-Emacs" target="_blank" class="broken_link">http://www.gnu.org/software/emacs/windows/Getting-Emacs.html#Getting-Emacs</a>
맥 버전 : Carbon Emacs: <a href="http://homepage.mac.com/zenitani/emacs-e.html" target="_blank" rel="nofollow">http://homepage.mac.com/zenitani/emacs-e.html</a>
Aquamacs Emacs: <a href="http://aquamacs.org/" target="_blank" rel="nofollow">http://aquamacs.org/</a>

2. CL 구현
SBCL : <a href="http://sbcl.sourceforge.net/platform-table.html" target="_blank" rel="nofollow">http://sbcl.sourceforge.net/platform-table.html</a>
맥에서는 MacPorts 라는 패키지 관리자를 이용하자, 다음 명령을 터미널에서 수행.
sudo port install sbcl

맥에서 패키지 관리자를 이용하여 설치하면 기본 설치 위치는
/usr/local 밑에 설치가 된다.

3. slime
slime : <a href="http://common-lisp.net/project/slime/" target="_blank" rel="nofollow">http://common-lisp.net/project/slime/</a>
맥 : sudo port install slime

맥에서 패키지 관리자를 이용해 설치하면 기본 설치 위치는
/opt/local/share/emacs/site-lisp/ 밑에 설치가 된다

4. 설정
Emacs 실행할때 홈디렉토리에서 .emacs 파일을 로딩하여 초기 설정을 한다.
윈도우는 c;, 리눅스 계열은 자신의 계정 홈디렉토리 .emacs라는 파일을 참조한다.


```lisp
(add-to-list 'load-path "/opt/local/share/emacs/site-lisp/slime/") 
(setq inferior-lisp-program "/usr/local/bin/sbcl") 
(require 'slime) 
(slime-setup '(slime-fancy slime-fuzzy slime-c-p-c)) 
(setq slime-net-coding-system 'utf-8-unix) 
```

정상적으로 Emacs 가 실행되었다면, Alt+x 혹은 ESC+x 키를 누르면 나오는 프롬프트창에 slime 이라고 입력하고 엔터키를 누르면 몇 몇 파일들이 주욱 로딩되며 아래와 slime listener 창이 뜰 것 이다.

이제 CL-USER> 라는 프롬프트를 볼 수 있을것이다.
그러면 성공한 것이다.