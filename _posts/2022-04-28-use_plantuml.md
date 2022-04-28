---
layout: post
title: Emacs에서 PlantUml을 사용하자
categories: [Emacs, org-mode]
tags: [emacs, plantuml, uml]
---
# Table of Contents

1.  [PlantUml 다운로드](#org3e9bd13)
2.  [graphviz 설치](#orgf4c1a0c)
3.  [emacs 설정](#org8896522)
4.  [사용법](#orgbc8f02a)

emacs의 org-mode에서 uml을 작성하고 생성할 수 있는 plantuml을 emacs에 설정해보겠다.  
맥에서 세팅중이라 맥을 기준으로 작성하겠다.  


<a id="org3e9bd13"></a>

# PlantUml 다운로드

PlanUml 사이트 다운로드 : <https://plantuml.com/ko/download>  
해당 사이트에서 PlanUml을 적당한 위치에 다운로드 한다.  


<a id="orgf4c1a0c"></a>

# graphviz 설치

PlantUml이 Class Diagram을 생성할때 graphviz를 필요로 하므로 미리 설치한다.  

    brew install graphviz


<a id="org8896522"></a>

# emacs 설정

    ;; 위에서 다운받은 plantuml.jar 위치를 지정해준다.
    (setq org-plantuml-jar-path
          (expand-file-name "~/.emacs.d/custom/package/plantuml.jar"))
    ;; plantuml 문법으로 uml 작성 후, C-c C-c를 누르면 이미지가 생성된다
    (add-hook 'org-babel-after-execute-hook
              (lambda ()
                (when org-inline-image-overlays
                  (org-redisplay-inline-images))))


<a id="orgbc8f02a"></a>

# 사용법

    #+BEGIN_SRC plantuml :file 파일명.png
    ClassA -> ClassB
    #+END_SRC

이제 위의 코드 블록에서 C-c C-c를 누르면 결과 Uml이 출력이 된다.  

