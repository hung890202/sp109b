# week13

## io1.c
```
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#define SMAX 128

int main() {
  int a = open("a.txt", O_RDWR); //傳回a的檔案描述值
  int b = open("b.txt", O_CREAT|O_RDWR, 0644);//傳回a的檔案描述值
  char text[SMAX];
  int n = read(a, text, SMAX); //讀檔內文字並存入text
  printf("n=%d\n", n);
  write(b, text, n);  //寫入b.txt
  printf("a=%d, b=%d\n", a, b);
}

```
* result
```
tony@ubuntu:~/sp/08-posix/04-fs/00-basic$ ./io1                 
n=10
a=3, b=4
tony@ubuntu:~/sp/08-posix/04-fs/00-basic$ cat b.txt             
hello!
hi
```
## stderr2.c
```
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
  int fdb = open("log.txt", O_CREAT|O_RDWR, 0644);  // 打開檔案 log.txt 並取得代號 fdb
  dup2(fdb, 2); // 複製 fdb 檔案描述子到 2 (stderr)
  fprintf(stderr, "Warning: xxx\n");
  fprintf(stderr, "Error: yyy\n");
}

```
* result
```
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ ./stderr2
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ ls
README.md  error.txt  log.txt  output.txt  stderr1.c  stderr2  stderr2.c
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ error.text
error.text: command not found
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ cat error.text
cat: error.text: No such file or directory
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ cat error.txt
Warning: xxx
Error: yyy
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$ cat log.txt
Warning: xxx
Error: yyy
tony@ubuntu:~/sp/08-posix/04-fs/04-stderr$
```

## blocking.c
```
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(int argc, char* argv[])
{
    int fd = open("/dev/tty",O_RDONLY); //Open the standard I / O file, which is blocked.
    if(fd == -1){
        perror("open /dev/tty");
        exit(1);
    }
    int ret = 0;
    char buf[1024] = {0};
    while(1){
        ret = read(fd, buf, sizeof(buf));
        if(ret == -1){
            perror("read");
            exit(1);
        }
        else if(ret == 0)
            printf("buf is null\n");
        else if(ret > 0)
            printf("buf is %s\n",buf);
        printf("test\n");
        sleep(1);
    }
    close(fd);

    return 0;
}
```
* result
```
tony@ubuntu:~/sp/08-posix/07-nonblocking$ gcc blocking1.c -o blocking1
tony@ubuntu:~/sp/08-posix/07-nonblocking$ ./blocking1
sda                                                 
buf is sda

test
```
* blocking1.c
```
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(int argc, char* argv[])
{
    int fd = open("/dev/tty",O_RDONLY); //Open the standard I / O file, which is blocked.
    if(fd == -1){
        perror("open /dev/tty");
        exit(1);
    }
    int ret = 0;
    char buf[1024] = {0};
    while(1){
        ret = read(fd, buf, sizeof(buf));
        if(ret == -1){
            perror("read");
            exit(1);
        }
        else if(ret == 0)
            printf("buf is null\n");
        else if(ret > 0)
            printf("buf is %s\n",buf);
        printf("test\n");
        sleep(1);
    }
    close(fd);

    return 0;
}
```
* result
```
tony@ubuntu:~/sp/08-posix/07-nonblocking$ ./blocking1
ffff
buf is ffff

test
qqqq
buf is qqqq

test
```
* nonblocking1.c
```
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(int argc, char* argv[])
{
    int fd = open("/dev/tty", O_RDONLY | O_NONBLOCK); // O ﹣ Nonblock set file input and output to non blocking
    if(fd == -1){
        perror("open /dev/tty");
        exit(1);
    }
    int ret = 0;
    char buf[1024] = {0};
    while(1){
        ret = read(fd, buf, sizeof(buf));
        if(ret == -1){
            perror("read /dev/tty"); // fputs(stderr, "read /dev/tty")
            printf("no input,buf is null\n");
        }
        else {
            printf("ret = %d, buf is %s\n",ret, buf);
        }
        sleep(1);
    }
    close(fd);

    return 0;
}
```
* result
```
tony@ubuntu:~/sp/08-posix/07-nonblocking$ ./nonblocking1
read /dev/tty: Resource temporarily unavailable
no input,buf is null
read /dev/tty: Resource temporarily unavailable
no input,buf is null
aaa
ret = 4, buf is aaa

aaaa
ret = 5, buf is aaaa
```
