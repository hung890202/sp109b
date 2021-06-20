# week1
* push
```
master >>> main
```
* basic
```
gcc sum.c
gcc sum.c -o sum
```

* link
```
多個檔案>>>執行檔
gcc main.c sum.c -o run

產生組合語言>>>執行檔
gcc -S main.c -o main.s
gcc -S sum.c -o sum.s
gcc main.s sum.s -o run

產生目的檔>>>執行檔
gcc -c sum.c -o sum.o
gcc -c main.c -o main.o
gcc main.o sum.o -o run
```

* toolchain
```
編譯成組合語言
gcc -S sum.c -o sum.s

組合語言組譯產生目的檔
gcc -c sum.s -o sum.o

將目的檔與函式庫連結後產生執行檔
gcc sum.o -o sum
```

* cpp
```
gcc >>> g++
```

* make
```
makeExe
    CC := gcc
    CFLAGS = -std=c99 -O0
    TARGET = run

    all: $(TARGET)

    $(TARGET): sum.c main.c
	    $(CC) $(CFLAGS) $^ -o $@

    clean:
	    rm -f *.o *.exe $(TARGET)

makeLib
    CC := gcc
    AR := ar
    CFLAGS = -std=c99 -O0
    TARGET = run
    LIB = libstat.a

    all: $(TARGET)

    $(TARGET): $(LIB) main.o
	    $(CC) $(CFLAGS) $^ -L ./ -lstat -o $@

    $(LIB): sum.o
	    $(AR) -r $@ $^

    %.o: %.c
	    $(CC) $(CFLAGS) -c $< -o $@

    clean:
	    rm -f *.o *.a *.exe $(TARGET)
```