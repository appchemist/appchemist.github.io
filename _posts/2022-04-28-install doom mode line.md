---
layout: post
title: Doom mode line 설정
categories: [Emacs]
tags: [emacs, theme]
---
# Table of Contents

1.  [Doom mode line 설치](#org8986342)
2.  [all-the-icons 설치](#org6cd2749)
3.  [Doom themes 설치](#orgea5f56c)


<a id="org8986342"></a>

# Doom mode line 설치

Doom mode line github 주소 : <https://github.com/seagle0128/doom-modeline>  
package manager를 통해서 doom-modeline 설치  

init.el에 다음과 같이 설정  

    (require 'doom-modeline)
    (doom-modeline-mode 1) 


<a id="org6cd2749"></a>

# all-the-icons 설치

Doom mode line은 all-the-icons에 포함된 폰트를 사용하므로 해당 폰트도 설치해 주자  

all-the-icons github 주소 : <https://github.com/domtronn/all-the-icons.el#installation>  

    M-x all-the-icons-install-fonts


<a id="orgea5f56c"></a>

# Doom themes 설치

그리고, Doom mode line 개발자가 강력 추천하는 doom-themes도 설치해주자(선택 사항)  
doom-themes github 주소 : <https://github.com/hlissner/emacs-doom-themes>  
해당 테마는 package manager로 doom-themes로 검색해서 설치하면 된다.  

