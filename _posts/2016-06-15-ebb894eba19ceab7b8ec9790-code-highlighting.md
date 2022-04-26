---
layout: post
title: 블로그에 Code Highlighting 적용하기
date: '2016-06-15T00:31:55+09:00'
tags:
- blogging
- code
- SyntaxHighlighter
tumblr_url: https://appchemist.tumblr.com/post/145945378261/ebb894eba19ceab7b8ec9790-code-highlighting
---
Code Highlight를 적용하기 앞써 어떤 제품들이 있는지 확인해보자.

- Prism
- Rainbows
- Snippet
- Geshi
- Syntax Highlighter
- Google Code Prettify
- Highlight.js
- SHJS : Syntax Highlighting in JavaScript
- Quick Highlighter
- Ultraviolet
- Pygments : Python Syntax Highlighter
- Lighter for MooTools
- CodePress
- Beauty of Code
- Jush JavaScript Syntax Highlighter

위와 같이 다양한 제품을 사용할 수 있다. 해당 제품들의 간단한 설명은 아래의 글을 참고하기 바란다.
<blockquote data-secret="H7hT0YCuFk" class="wp-embedded-content"><a href="https://codegeekz.com/15-code-syntax-highlighters-to-prettify-your-code/">15 Code Syntax Highlighters To Prettify Your Code</a></blockquote>
<iframe class="wp-embedded-content" sandbox="allow-scripts" security="restricted" style="position: absolute; clip: rect(1px, 1px, 1px, 1px);" src="https://codegeekz.com/15-code-syntax-highlighters-to-prettify-your-code/embed/#?secret=H7hT0YCuFk" data-secret="H7hT0YCuFk" width="600" height="338" title="“15 Code Syntax Highlighters To Prettify Your Code” — Code Geekz" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>
이번 글은 Syntax Highlighter를 적용해본다.
설치는 간단하다.

아래와 같이 몇 줄만 추가하면 Code Highlight가 적용이 된다.


```js
<!— Syntax Highlighter를 적용하기 위해 추가가 필요한 부분 —>
<script type="text/javascript" src="js/shCore.js"></script>
<script type="text/javascript" src="css/shBrushJScript.js"></script>
<link href="css/shCore.css" rel="stylesheet" type="text/css" />
<link href="css/shThemeDefault.css" rel="stylesheet" type="text/css" />
<!— Syntax Highlighter를 적용하기 위해 추가가 필요한 부분 —>

<pre class="brush: js">
function foo()
{
}
</pre>
```

일단 위와 같은 코드를 적용하기 위해서 코드를 다운로드 받는다.

직접 운영중인 사이트라고 한다면, 해당 코드들을 서버에 올려서 링크를 걸면 된다.

Tumblr의 경우에는 코드들을 서버에 올릴수 있긴 하지만, 테마 변경 시, 모두 리셋이 되어 불편함이 있다.
나의 경우에는 Github에 Syntax Highlighter 코드를 모두 올려두고 사용했다.

Github에 코드를 올려두고 링크 주소를 아래와 같이 사용하면 된다.

<a href="https://github.com/%EC%95%84%EC%9D%B4%EB%94%94..">https://github.com/아이디..</a> 와 같은 형태의 주소를 <a href="https://cdn.rawgit.com/%EC%95%84%EC%9D%B4%EB%94%94/..">https://cdn.rawgit.com/아이디/..</a> 와 같은 형태의 주소로 사용하면 된다.
Syntax Highlighter는 다양한 Theme을 제공하는데, 다운로드 받은 코드 디렉토리의 styles 디렉토리에 들어 있다.

원하는 테마를 css/shThemeDefault.css 대신에 넣어주면 된다.