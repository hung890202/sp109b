## For
* for 1.0

<pre><code>int F() {
  int f;
  if (isNext("(")) { // '(' E ')'
    next(); // (
    f = E();
    next(); // )
  }
  else { // Number | Id
    f = nextTemp();
    char *item = next();
    if(isdigit(*item)){
      emit("t%d = %s\n", f, item);
    }
    else{
      if(isNext("++")){
        next();
        emit("%s = %s + 1\n", item, item);
      }
      else if(isNext("--")){
        next();
        emit("%s = %s - 1\n", item, item);
      }
      emit("t%d = %s\n", f, item);
    }
  }
  return f;
}
</code></pre>


<pre><code>void FOR(){
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
</code></pre>

* for.c
<pre><code>for(i=0; i<10; i++){
    a = a + 1;
}
</code></pre>

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

* for 2.0

<pre><code>// #define emit printf
int isTempIr = 0;
char tempIr[100000], *tempIrp = tempIr;
#define emit(...) ({ \
  if (isTempIr){  \
    sprintf(tempIrp, __VA_ARGS__);  \
    tempIrp += strlen(tempIrp);\
  }\
  else { \
    printf(__VA_ARGS__); \
  }\
})
</code></pre>

<pre><code>void FOR(){
  int forBegin = nextLabel();
  int forEnd = nextLabel();
  emit("(L%d)\n", forBegin);
  skip("for");
  skip("(");
  ASSIGN();
  int e = E();
  emit("if not T%d goto L%d\n", e, forEnd);
  skip(";");
  isTempIr = 1;
  int e1 = E(); 
  isTempIr = 0;
  char e1str[10000];
  strcpy(e1str, tempIr); //複製tempIr給e1str
  skip(")");
  STMT();
  emit("%s\n", e1str); //印出e1str(i++)
  emit("goto L%d\n", forBegin);
  emit("(L%d)", forEnd);
}
</code></pre>

* for.c
<pre><code>for(i=0; i<10; i++){
    a = a + 1;
}
</code></pre>

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
t5 = a
t6 = 1
t7 = t5 + t6
a = t7
i = i + 1
t4 = i

goto L0
(L1)
```