# 期末報告(C4理解)

## 申明
程式碼來源自[陳忠誠老師改編的程式](https://gitlab.com/ccc109/sp/-/blob/master/C1-c4/c4.c)

* C4簡介
    * C4 是 Robert Swierczek 寫的一個小型 C 語言編譯器，全部 527 行的原始碼。
    * 傳統的C語言編譯器，最終都把C語言源碼編譯成可執行文件，也就是二進位的機器碼。
    而C4則是把C語言源碼先編譯成其專門設計的字節碼（bytecode），然後直接在虛擬機中解釋執行。
    * C4 Compiler 不僅是個 C self-host compiler，它也啟發了許多人去撰寫自己的 Compiler，甚至是個了解 Compiler 很好的入門磚。
    * C4 編譯完成後，會產生一種《堆疊機機器碼》放在記憶體內，然後 虛擬機 會立刻執行該機器碼。其最特別的功能在於他能自我編譯。
* C4特色
    * Self-Host Compiler.(自舉-自己編譯自己)
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
