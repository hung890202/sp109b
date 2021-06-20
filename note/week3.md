# week3

* lexer sum.c
```
#include "sum.h"

int main() {
  int t = sum(10);
  printf("sum(10)=%d\n", t);
}
token=#
token=include
token="sum.h"
token=int
token=main
token=(
token=)
token={
token=int
token=t
token==
token=sum
token=(
token=10
token=)
token=;
token=printf
token=(
token="sum(10)=%d\n"
token=,
token=t
token=)
token=;
token=}
0:#
1:include
2:"sum.h"
3:int
4:main
5:(
6:)
7:{
8:int
9:t
10:=
11:sum
12:(
13:10
14:)
15:;
16:printf
17:(
18:"sum(10)=%d\n"
19:,
20:t
21:)
22:;
23:}
```

* 習題IF
```
void IF(){
  int ifBegin = nextLabel();
  int ifMid = nextLabel();
  int ifEnd = nextLabel();
  emit("(L%d)\n", ifBegin);
  skip("if");
  skip("(");
  int e = E();
  emit("if not T%d goto L%d\n", e, ifMid);
  skip(")");
  STMT();
  emit("goto L%d\n", ifEnd);
  emit("(L%d)\n", ifMid);
  if(isNext("else")){
    skip("else");
    STMT();
    emit("(L%d)\n",ifEnd);
  }
}
```
* 執行結果
```
========== lex ==============
token=if
token=(
token=a
token=>
token=3
token=)
token={
token=t
token==
token=1
token=;
token=}
token=else
token=if
token=(
token=a
token=<=
token=0
token=)
token={
token=t
token==
token=2
token=;
token=}
token=else
token={
token=t
token==
token=3
token=;
token=}
========== dump ==============
0:if
1:(
2:a
3:>
4:3
5:)
6:{
7:t
8:=
9:1
10:;
11:}
12:else
13:if
14:(
15:a
16:<=
17:0
18:)
19:{
20:t
21:=
22:2
23:;
24:}
25:else
26:{
27:t
28:=
29:3
30:;
31:}
============ parse =============
(L0)
t0 = a
t1 = 3
t2 = t0 > t1
if not T2 goto L1
t3 = 1
t = t3
goto L2
(L1)
(L3)
t4 = a
t5 = 0
t6 = t4 <= t5
if not T6 goto L4
t7 = 2
t = t7
goto L5
(L4)
t8 = 3
t = t8
(L5)
(L2)
```

* 習題FOR
```
void FOR(){
  int forBegin = nextLabel();
  int forEnd = nextLabel();
  emit("(L%d)\n", forBegin);
  skip("for");
  skip("(");
  ASSIGN();
  int e = E();
  emit("if not T%d goto L%d\n", e, forEnd);
  skip(";");
  int e1 = E();
  skip(")");
  STMT();
  emit("goto L%d\n", forBegin);
  emit("L%d", forEnd);
}
```
* 執行結果
```
========== lex ==============
token=for
token=(
token=i
token==
token=0
token=;
token=i
token=<
token=10
token=;
token=i
token=++
token=)
token={
token=a
token==
token=a
token=+
token=1
token=;
token=}
========== dump ==============
0:for
1:(
2:i
3:=
4:0
5:;
6:i
7:<
8:10
9:;
10:i
11:++
12:)
13:{
14:a
15:=
16:a
17:+
18:1
19:;
20:}
============ parse =============
(L0)
t0 = 0
i = t0
t1 = i
t2 = 10
t3 = t1 < t2
if not T3 goto L1
i = i + 1
t4 = i
t5 = a
t6 = 1
t7 = t5 + t6
a = t7
goto L0
(L1)
```