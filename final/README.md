# 期末報告(C4理解)

## 申明
程式碼來源自[陳忠誠老師改編的程式](https://gitlab.com/ccc109/sp/-/blob/master/C1-c4/c4.c)\
相關資料[license](https://github.com/hung890202/sp109b/blob/main/LICENSE.md)

## C4虛擬機
* C4簡介
    * C4 是 Robert Swierczek 寫的一個小型 C 語言編譯器，全部 527 行的原始碼。
    * 傳統的C語言編譯器，最終都把C語言源碼編譯成可執行文件，也就是二進位的機器碼。
    而C4則是把C語言源碼先編譯成其專門設計的字節碼（bytecode），然後直接在虛擬機中解釋執行。
    * C4 Compiler 不僅是個 C self-host compiler，它也啟發了許多人去撰寫自己的 Compiler，甚至是個了解 Compiler 很好的入門磚。
    * C4 編譯完成後，會產生一種《堆疊機機器碼》放在記憶體內，而虛擬機會立刻執行該機器碼。
* C4特色
    * Self-Host Compiler.
    ```
    gcc -m32 -o c4 c4.c  (you may need the -m32 option on 64bit machines)
    ./c4 test/hello.c
    ./c4 -s test/hello.c

    ./c4 c4.c test/hello.c
    ./c4 c4.c c4.c test/hello.c
    ```
    * Local Variables Declaration 必須在 Function 裡面的最開頭位置。
    * single-pass compiler(僅使每個編譯單元的各個部分通過一次，然後立即將每個部分轉換為其最終機器代碼的編譯器)
    * 最後生成虛擬機的代碼。

* C4不支援的語法：
    * 設定初值 int a=3; int arr = {1,2,3}
    * 巨集 #define LEN 100
    * 宣告陣列 int a[10];
    * 結構 struct
* C4忽略的語法
    ```
    #include <stdio.h>
    ```
    >但是預設已經引入一些函式庫例如： printf 。

* 高階指令 (透過呼叫 C 語言函數完成) :

| 指令    意義                        |         實作     | 
| ----                               | ----            | 
|OPEN: open                           |   a = open((char *)sp[1], *sp);|
|READ: read                           |  a = read(sp[2], (char *)sp[1], *sp);|
|CLOS: close                          |   a = close(*sp);|
|PRTF: printf                         |  t = sp + pc[1]; a = printf((char *)t[-1], t[-2], t[-3], t[-4], t[-5], t[-6]);|
|MALC: malloc                         | a = (int)malloc(*sp);|
|FREE: free                           |free((void *)*sp);|
|MSET: memset                         |a = (int)memset((char *)sp[2], sp[1], *sp);|
|MCMP: memcmp                         |a = memcmp((char *)sp[2], (char *)sp[1], *sp);|

>累積器 a 堆疊暫存器 sp 框架暫存器 bp ，所有 +-*/ 等運算幾乎都是對 a 進行的，而 bp 則可被用來存取區域變數。透過這些指令， c4 可以直接將這些工作轉交 C 語言函式庫來處理。
* Virtual Machine (c4)
    * sp：stack frame。
    * a：register 暫存器。
    * bp：概念跟 rbp(32 bits)/ebp(64 bits) 相同。
    * pc：program counter，指向當前指令。

* 機器指令：

|指令   |意義                                                |    實作|
|----                   |----                                     |----    |
LEA : load local address |載入區域變數                         | a = (int)(bp + *pc++); |
IMM : load global address or immediate |載入全域變數或立即值    | a = *pc++;|
JMP : jump               |躍躍指令                              |pc = (int *)*pc;|
JSR : jump to subroutine |跳到副程式                           | *--sp = (int)(pc + 1); pc = (int *)*pc;|
BZ  : branch if zero     |if (a==0) goto m[pc]                | pc = a ? pc + 1 : (int *)*pc;|
BNZ : branch if not zero |if (a!=0) goto m[pc]                | pc = a ? (int *)*pc : pc + 1;|
ENT : enter subroutine   |進入副程式                           | *--sp = (int)bp; bp = sp; sp = sp - *pc++;|
ADJ : stack adjust       |調整堆疊                             | sp = sp + *pc++;|
LEV : leave subroutine   |離開副程式                           | sp = bp; bp = (int *)*sp++; pc = (int *)*sp++;|
LI  : load int           |載入整數                             | a = *(int *)a;|
LC  : load char          |載入字元                             | a = *(char *)a;|
SI  : store int          |儲存整數                             | *(int *)*sp++ = a;|
SC  : store char         |儲存字元                             | a = *(char *)*sp++ = a;|
PSH : push                |推入堆疊                             | *--sp = a;|
OR  : a = a OR pop        |pop 代表從堆疊中取出一個元素          | a = *sp++ |  a;|
XOR : a = a XOR pop       |                                    | a = *sp++ ^  a;|
AND : a = a AND pop       |                                    | a = *sp++ &  a;|
EQ : a = a EQ pop         |                                    | a = *sp++ == a;|
NE : a = a NE pop         |                                    | a = *sp++ != a;|
LT : a = a LT pop         |                                    | a = *sp++ <  a;|
GT : a = a GT pop         |                                    | a = *sp++ >  a;|
LE : a = a LE pop         |                                    | a = *sp++ <= a;|
GE : a = a GE pop         |                                    | a = *sp++ >= a;|
SHL : a = a SHL pop       |                                    | a = *sp++ << a;|
SHR : a = a SHR pop       |                                    | a = *sp++ >> a;|
ADD : a = a ADD pop       |                                    | a = *sp++ +  a;|
SUB : a = a SUB pop       |                                    | a = *sp++ -  a;|
MUL : a = a MUL pop       |                                    | a = *sp++ *  a;|
DIV : a = a DIV pop       |                                    | a = *sp++ /  a;|
MOD : a = a MOD pop       |                                    | a = *sp++ %  a;|
EXIT : 終止離開           |                                     |  return *sp;|

## 執行C4
* hello.c
```
$ ./c4 test/hello.c
hello, world
exit(0) cycle = 9
```
* 加`-d`執行 :
```
$./c4 -d test/hello.c       
1> ENT  0
2> IMM  6946904
3> PSH
4> PRTF
hello, world
5> ADJ  1
6> IMM  0
7> LEV
8> PSH
9> EXIT
exit(0) cycle = 9
```
* 加`-s`執行 :
```
$ ./c4 -s test/hello.c       
1: #include <stdio.h>
2:
3: int main()
4: {
5:   printf("hello, world\n");
    ENT  0
    IMM  8454232
    PSH
    PRTF
    ADJ  1
6:   return 0;
    IMM  0
    LEV
7: }
    LEV
```

## 參考資料
* [老師的c4.c](https://gitlab.com/ccc109/sp/-/tree/master/C1-c4)
* [原創程式碼](https://github.com/rswier/c4)
* [C in four function (c4) Compiler](https://hackmd.io/@srhuang/Bkk2eY5ES)
* [C4：4個函數，528行代碼實現可自舉的C語言編譯器l](https://kknews.cc/zh-tw/code/zrkmqga.html)
* [有哪些关于c4 - C in four function 编译器的文章？](https://www.zhihu.com/question/28249756)