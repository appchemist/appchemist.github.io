---
layout: post
title: ld linker (링킹의 기본 이해)
date: '2013-03-31T08:35:17+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118749334556/ld-linker-eba781ed82b9ec9d98-eab8b0ebb3b8
---
ld는 리눅스 시스템에서 사용하는 링커이다. gcc는 collect2를 호출해 링킹 과정을 수행하는데, collect2는 내부적으로 진짜 링커인 ld를 호출해 링킹 과정을 수행한다. 
링킹 과정이란?

컴파일과 링킹에서 마지막 과정으로 조각한 오브젝트 파일들을 하나의 바이너리 이미지로 합치는 과정이다.

링킹 과정은 결합과 재배치 딱 두마디로 요약할 수 있다. 
링킹 과정 절차

결합 과정은 ELF 포맷으로 되어 있는 각 오브젝트를 섹션 종류별로 하나의 오브젝트로 합치는 과정

main.c


```cpp
#include &lt;stdio.h&gt;

void func1();
void func2();

int var1 = 0x111111;
int var2;
int var3 = 0;

int main() {
        static int var4 = 0x222222;
        static int var5;
        int var6;

        printf("This is main() function!n");
        func1();
        func2();

        return 0;
}
```

funcs.c


```cpp
#include &lt;stdio.h&gt;

extern int var1, var2;
int var8 = 0x333333;
const int var9 = 0x12345678;
int var10;

void func1() {
        printf("This is func1() function!n");
        printf("var1 = 0x%X, var2=0x%Xn", var1, var2);
}

void func2() {
        printf("This is func2() function!n");
}
```

위의 소스 파일들을 가지고 아래의 명령을 내리면 다음과 같이 test 실행 파일이 생성된다.


```bash
$ gcc -o test main.c funcs.c -v -save-temps
Using built-in specs.
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-libgcj-multifile --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --disable-plugin --with-java-home=/usr/lib/jvm/java-1.4.2-gcj-1.4.2.0/jre --with-cpu=generic --host=x86_64-redhat-linux
Thread model: posix
gcc version 4.1.2 20080704 (Red Hat 4.1.2-52)
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/cc1 -E -quiet -v main.c -mtune=generic -fpch-preprocess -o main.i
ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../x86_64-redhat-linux/include"
#include "..." search starts here:
#include &lt;...&gt; search starts here:
 /usr/local/include
 /usr/lib/gcc/x86_64-redhat-linux/4.1.2/include
 /usr/include
End of search list.
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/cc1 -fpreprocessed main.i -quiet -dumpbase main.c -mtune=generic -auxbase main -version -o main.s
GNU C version 4.1.2 20080704 (Red Hat 4.1.2-52) (x86_64-redhat-linux)
        compiled by GNU C version 4.1.2 20080704 (Red Hat 4.1.2-52).
GGC heuristics: --param ggc-min-expand=64 --param ggc-min-heapsize=63690
Compiler executable checksum: 0fb434bacb069a61dfb7d474a8bae350
 as -V -Qy -o main.o main.s
GNU assembler version 2.17.50.0.6-20.el5 (x86_64-redhat-linux) using BFD version 2.17.50.0.6-20.el5 20061020
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/cc1 -E -quiet -v funcs.c -mtune=generic -fpch-preprocess -o funcs.i
ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../x86_64-redhat-linux/include"
#include "..." search starts here:
#include &lt;...&gt; search starts here:
 /usr/local/include
 /usr/lib/gcc/x86_64-redhat-linux/4.1.2/include
 /usr/include
End of search list.
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/cc1 -fpreprocessed funcs.i -quiet -dumpbase funcs.c -mtune=generic -auxbase funcs -version -o funcs.s
GNU C version 4.1.2 20080704 (Red Hat 4.1.2-52) (x86_64-redhat-linux)
        compiled by GNU C version 4.1.2 20080704 (Red Hat 4.1.2-52).
GGC heuristics: --param ggc-min-expand=64 --param ggc-min-heapsize=63690
Compiler executable checksum: 0fb434bacb069a61dfb7d474a8bae350
 as -V -Qy -o funcs.o funcs.s
GNU assembler version 2.17.50.0.6-20.el5 (x86_64-redhat-linux) using BFD version 2.17.50.0.6-20.el5 20061020
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/collect2 --eh-frame-hdr -m elf_x86_64 --hash-style=gnu -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o test /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crt1.o /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/4.1.2/crtbegin.o -L/usr/lib/gcc/x86_64-redhat-linux/4.1.2 -L/usr/lib/gcc/x86_64-redhat-linux/4.1.2 -L/usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 main.o funcs.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-redhat-linux/4.1.2/crtend.o /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crtn.o
```

위의 내용을 확인해보면 실제로 호출된 collect2는 실제 링커인 ld를 호출하는데, collect2가 받은 옵션들을 ld에게 그대로 넘겨 링킹한다. 따라서 위에서 collect2만 ld로 변경하면 그대로 링킹됨을 확인할 수 있다.

그리고 main.c와 funcs.c를 컴파일해 test 실행 파일을 만드는데, 다음 그림과 같이 실제로는 많은 라이브러리와 오브젝트들이 함께 링킹됨을 확인할 수 있다.
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2013/03/linking-%EA%B7%B8%EB%A6%BC.png"><img src="http://i2.wp.com/appchemist.net/wp-content/uploads/2013/03/linking-%C3%AA%C2%B7%C2%B8%C3%AB%C2%A6%C2%BC.png?resize=720%2C598" alt="linking 그림" class="aligncenter size-full wp-image-331" data-recalc-dims="1"/></a>

여기에서 어셈블된 *.o의 오브젝트 파일은 ELF 바이너리 포맷으로 되어있으며, ELF 바이너리 포맷은 .text 섹션, .data 섹션, bss 섹션, rodata 섹션 등으로 이루어 져 있다.

결합 과정에서 각 오브젝트 파일의 각 세션이 종류별로 합쳐저 하나의 ELF 실행 파일을 구상한다. 이렇게 합쳐지는 순서는 별도의 링커 스크립트를 사용하지 않았다면 ld 명령에서 인자로 넣은 오브젝트 파일의 순서를 따른다.

링킹 과정에서 합쳐지는 섹션들이 있고, 통합되거나 없어지는 섹션도 있다.
두 번째로 재배치 과정이 일어난다. 재배치 과정은 결합 과정에서 합쳐진 각 센셕을 실제 코드에 맞게 조정하는 과정이라고 볼 수 있다.

재배치 과정은 메모리에 바이너리 이미지가 로드될 위치를 시작으로 결합 과정이 끝난 바이너리에 각 심볼이 가지게 될 실제 주소를 구하고, 해당 심볼을 참조하는 부분에 대해 구한 주소를 설정하는 과정이다.

심볼이란 주소를 가지는 모든 것을 말하는데, 이는 함수일수도 있고 변수일수도 있다.

C소스에서 함수와 전역변수 등은 컴파일 과정이 끝난 어셈블리 코드에서 함수와 변수를 참조하기 위한 레이블(label)이란 것을 가진다.

레이블은 함수명 또는 변수명 뒤에 콜란(:)이 붙은 형태인데, 해당 함수를 호출하거나 해당 변수를 load/store할 때는 이런 레이블을 참조한다. 이후 레이블은 모두 주소로 변경되고 레이블을 참조하는 부분 역시 레이블의 주소로 변경된다.

레이블은 main.c 파일을 gcc -S main.c 명령으로 컴파일하면 다음과 같이 어셈블리 파일이 나오는데, 여기에서 확인할 수 있다.


```cpp
        .file   "main.c"
.globl var1
        .data
        .align 4
        .type   var1, @object
        .size   var1, 4
var1:
        .long   1118481
.globl var3
        .bss
        .align 4
        .type   var3, @object
        .size   var3, 4
var3:
        .zero   4
        .local  var5.2131
        .comm   var5.2131,4,4
        .data
        .align 4
        .type   var4.2130, @object
        .size   var4.2130, 4
var4.2130:
        .long   2236962
        .section        .rodata
.LC0:
        .string "This is main() function!"
        .text
.globl main
        .type   main, @function
main:
.LFB2:
        pushq   %rbp
.LCFI0:
        movq    %rsp, %rbp
...
```

레이블과 같이 c 소스상에서 주소를 표현하는 모든 것을 심볼이라고 한다. 여기에서 심볼에 포함되는 것에는 레이블만 있는 것은 아니다.

레이블은 어셈블리상의 주소표현이고, 링커 스크립트에 의해 생성되는 심볼도 있다.
재배치 과정까지 끝아면 실행파일이 준비가 된다. ./test 명령을 내리면 bash 셸에서 exec() 시스템 콜을 사용해 test 실행 파일을 메모리에 로드하고, test 프로그램은 ld에 의해 정해진 주소에 로드되어 수행된다.