---
layout: post
title: org-mode에서 한글로 export 하기
categories: [Emacs, org-mode]
tags: [emacs, org-mode, ko, export]
---

# Table of Contents

1.  [tex 설치](#orgd7609b7)
    1.  [Ubuntu](#org244ba9a)
    2.  [Mac](#orgb7ed3d6)
2.  [Emacs 설정](#org1902336)

기본 org-mode를 사용해서 글을 작성 후, pdf 포멧으로 생성할 경우 에러를 만나게 된다.

즉, 기본 설정 org-mode에서는 한글로 작성한 글을 pdf 포멧으로 바로 생성이 안 된다.
해당 글에서는 org-mode에서 export를 통해서 한글이 존재하는 글을 pdf 포멧으로 생성하는 방법을 정리하고자 한다.


<a id="orgd7609b7"></a>

# tex 설치


<a id="org244ba9a"></a>

## Ubuntu

    apt-get install texlive-xetex
    apt-get install ko.tex


<a id="orgb7ed3d6"></a>

## Mac

    brew cask install mactex


<a id="org1902336"></a>

# Emacs 설정

init.el 파일 이나 설정 파일에 다음과 같이 추가

    (require 'ox-latex)
    (setenv "PATH"
          (concat
           ;; latex 위치를 못 찾을 경우 해당 경로를 지정
           ;; mac의 경우 못 찾아서 설정을 해줬음
           ;; 아래의 위치는 나의 mactex 설치 위치
           "/Library/TeX/texbin/" ":"
           (getenv "PATH")))

위와 같이 tex 설치 및 설정이 완료가 됐다면, 이제 org-mode에서 글을 작성 후, export pdf를 수행하면 된다.

단, 각 글의 해더 부분에 아래와 같이 작성해줘야 한다.

    #+LATEX_HEADER: \usepackage{kotex}

이제 org-mode에서 pdf 출력 준비가 완료 됐다.
한글 문서를 작성하고 Org Export 기능(C-c C-e)을 사용해서 pdf를 출력해보자

