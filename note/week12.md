# week12

## fork1.c
```
#include <stdio.h> 
#include <sys/types.h> 
#include <unistd.h>

int main() { 
    fork(); // 一個行程分叉成父子兩個行程
    fork(); // 兩個行程又分別分叉出兩對父子，所以變成四個行程。
    printf("%-5d : Hello world!\n", getpid());
}

``` 
* result
```
12372 : Hello world!                                             //父行程
12371 : Hello world!                                             //子行程
12373 : Hello world!                                             //父行程
./fork1                                                           
12374 : Hello world!                                             //子行程
```
## fork2.c
```
#include <stdio.h> 
#include <sys/types.h> 
#include <unistd.h>

int main() { 
    printf("%-5d : enter\n", getpid());
    fork(); // 一個行程分叉成父子兩個行程
    printf("%-5d : after 1st fork\n", getpid());
    fork(); // 兩個行程又分別分叉出兩對父子，所以變成四個行程。
    printf("%-5d : Hello world!\n", getpid());
}
```
* result
```
./fork2
12393 : enter
12393 : after 1st fork
12393 : Hello world!
12395 : Hello world!
12394 : after 1st fork
12394 : Hello world!
12396 : Hello world!
```
## 02-child fork2.c
```
#include <stdio.h> 
#include <sys/types.h> 
#include <unistd.h>

int main() { 
    fork();  // 一個行程分叉成父子兩個行程
    if (fork()==0) { // 兩個行程又分別分叉出兩對父子，所以變成四個行程。
      printf("%-5d: I am child!\n", getpid());
    } else {
      printf("%-5d: I am parent!\n", getpid());
    }
}
```
* ressult
```
./fork2
12402: I am parent!
12404: I am child!
12403: I am parent!
12405: I am child!
```
## execvp1.c >>>(ls -l)
```
#include <stdio.h>
#include <unistd.h>

int main() {
  char *arg[] = {"ls", "-l", NULL };
  printf("execvp():before\n");
  execvp(arg[0], arg);
  printf("execvp():after\n");
}
```
* result
```
drwxrwxr-x 2 tony tony  4096 Mar 29 10:19 backup
-rwxrwxr-x 1 tony tony 16792 Jun 16 21:10 execvp1
-rw-rw-r-- 1 tony tony   176 Mar 29 10:19 execvp1.c
```
## system1.c
```
#include <stdio.h>
#include <stdlib.h>

int main() {
  system("ls -l");
  printf("main end!\n");
}
```
* result
    
```
-rw-rw-r-- 1 tony tony   260 Jun 10 11:48 mysystem0.c
-rw-rw-r-- 1 tony tony   332 Jun 10 11:48 mysystem1.c
-rwxrwxr-x 1 tony tony 16744 Jun 16 21:35 system1
-rw-rw-r-- 1 tony tony    99 Mar 29 10:19 system1.c
main end!
```
## mysystem1.c
```
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h> 
#include <sys/wait.h>

int mysystem(char *arg[]) {
  if (fork()==0) {                                      //父行程繼續執行main
    execvp(arg[0], arg); // child : exec("ls -l")       //子行程被替換
  }
  int status;                                           
  wait(&status);                                        //等待子行程跑完
  return status;
}

int main() {
  char *arg[] = {"ls", "-l", NULL };
  mysystem(arg);
  printf("main end!\n");
}

```
## zombie.c
```
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
int main () {
  pid_t child_pid;
  /* Create a child process. */
  child_pid = fork ();
  if (child_pid > 0) {
    /* This is the parent process. Sleep for a minute. */
    sleep (60);
  } else {
    /* This is the child process. Exit immediately. */
    exit (0);
  }
  return 0;
}
```
* result
```
    PID TTY          TIME CMD
  10092 pts/0    00:00:00 bash
  12455 pts/0    00:00:00 zombie
  12456 pts/0    00:00:00 zombie <defunct>
  12457 pts/0    00:00:00 ps
```
## echo.c
```
#include <stdio.h>
#include <unistd.h>
#define SMAX 128

int main() {
  char line[SMAX];
  int n = read(0, line, SMAX); // 從 0 (標準輸入 stdin:鍵盤) 讀入一行字 line
  line[n] = '\0';              // 設定字串結尾
  write(1, line, n);           // 將 line 輸出到 1 (標準輸出 stdout)
  write(2, line, n);           // 將 line 輸出到 2 (標準錯誤 stderr)
}

```
* result
```
tony Hi // 讀檔點
tony Hi
tony Hi
```
## fecho.c
```
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#define SMAX 128

int main() {
  close(0);                      // 關閉標準輸入 stdin
  close(1);                      // 關閉標準輸出 stdout
  int a = open("a.txt", O_RDWR);         // 此時 open，會找沒被使用的最小檔案代號 0
  int b = open("b.txt", O_CREAT|O_RDWR, 0644); // 此時 open，會找沒被使用的最小檔案代號 1
  char line[SMAX];
  gets(line);                    // 從 0 (a.txt) 讀入一行字 line
  puts(line);                    // 輸出 line 到 1 (b.txt)
  printf("a=%d, b=%d\n", a, b);
}

```
* result
```
tony@foo1:~/sp/08-posix/04-fs/02-fecho$ ./fecho
>>>>>>>>>>>>>>>>>>>>>a.txt結果>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
tony@foo1:~/sp/08-posix/04-fs/02-fecho$ cat a.txt
Hello, File descriptor !
>>>>>>>>>>>>>>>>>>>>>b.txt結果>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
tony@foo1:~/sp/08-posix/04-fs/02-fecho$ cat b.txt
Hello, File descriptor !
a=0, b=1
```

#### myshell.c
```
#include "../myshell.h"

int main(int argc, char *argv[]) {
  char path[SMAX], cmd[SMAX];
  getcwd(path, SMAX-1); // 取得初始路徑
  while (1) { // 不斷等待使用者輸入命令並執行之
    printf("myshell:%s $ ", path); // 顯示提示訊息
    fgets(cmd, SMAX-1, stdin);     // 等待使用者輸入命令
    system(cmd);                   // 執行命令
  }
}

```

* result
```
tony@foo1:~/sp/08-posix/05-myshell/v1$ ./myshell
myshell:/home/tony/sp/08-posix/05-myshell/v1 $ ls
README.md  myshell  myshell.c
```