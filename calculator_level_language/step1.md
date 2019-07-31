# 第1步：創造能編譯1個整數的語言

想像一下最簡單的C語言的子集合。不知道讀者想到的是什麼樣的語言？是只有`main`函式的語言嗎？還是只有只有一個式子所構成的語言？

如果仔細想想，所謂最簡單的子集合，應該是由1個整數所構成的語言。也是在這一步我們要實作的語言。

在這一步所做出的編譯器，會從輸入讀入1個整數，然後輸出將其作為結束碼的組合語言。也就是輸入是像42這樣的整數，讀到之後編譯器會輸出下面這樣的組合語言：

```text
.intel_syntax noprefix
.global main

main:
        mov rax, 42
        ret
```

`.intel_syntax noprefix`代表的是在眾多組合語言的寫法中，指定本書所使用的 Intel 語法的組合語言指令。這次製作的編譯器在第一行請務必加上這行。其他行的指令的說明請參照前一章的介紹。

這時讀者可能會想說：「這樣的程式才不算編譯器」，說實話筆者也這樣認為。但是這個程式可以對應只有1個整數的語言，並且根據該數值輸出對應的程式，照定義來說它有充分的資格能作為一個編譯器。而這樣一個簡單的程式，只要一路改寫很快就能做很複雜的事情了，但首先得先完成這一步。

這一步，從整體開發流程來看，其實非常重要。因為以後的開發是以這一步開發的成果為骨幹進行的。這一步在編譯器本身之外，還會寫 Makefile、製作單元測試（unit test）、設定git repository。我們一個一個來看該怎麼完成這些作業。

此外，本書所做的編譯器的名字叫做 9cc，cc 是 C compiler 的簡稱。9這個數字沒有什麼特別的意義，但是筆者之前所寫的編譯器叫做 8cc，所以作為下一個作品就稱為 9cc。讀者可以照自己的喜好取名，但是請不要想名字想太久就一直沒有開始動手寫。包含 GitHub repository 在內，名字以後可以再改，可以暫時先隨便取一個名字沒有關係。

{% hint style="info" %}
**小知識：Intel 語法與 AT&T 語法**

本書所用的 Intel 語法以外，還有 AT&T 語法這個以 Unix 為中心廣為人知的組合語言寫法。gcc 或 objdump 預設都是輸出 AT&T 語法的組合語言。

AT&T 語法的結果是放在第二引數，也就是兩個引數要反過來寫。暫存器需要加上`%`引用符號寫成像`%rax`。數值則要加上`$`寫成像`$42`。

除此之外，參照記憶體也有其獨特的寫法，是用`()`取代`[]`。參考兩者的對比的範例：

```text
mov rbp, rsp   // Intel
mov %rsp, %rbp // AT&T

mov rax, 8     // Intel
mov $8, %rax   // AT&T

mov [rbp + rcx * 4 - 8], rax // Intel
mov %rax, -8(rbp, rcx, 4)    // AT&T
```

這次我們要製作的編譯器為了容易閱讀採用 Intel 語法。Intel 指令集的說明也是使用 Intel 語法，所以也有著可以直接照手冊的說明來寫指令的好處。語法的功能 Intel 語法和 AT&T 語法都是一樣的，無論用哪種方法來寫，輸出的機械碼都一樣。
{% endhint %}

## 製作編譯器本體

編譯器的輸入通常都是檔案，但現階段處理開關檔還稍嫌麻煩，我們直接從指令的第1引數來輸入程式碼。以下是從第1引數取值，再把其加到固定的組合語言指令裡的簡單C程式碼：

{% code-tabs %}
{% code-tabs-item title="9cc.c" %}
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "引數數量錯誤\n");
    return 1;
  }

  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");
  printf("  mov rax, %d\n", atoi(argv[1]));
  printf("  ret\n");
  return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

建立一個名為 9cc 的資料夾，把上面的程式碼存成 9cc.c 這個檔案放在資料夾內。然後照以下的指令執行 9cc 確認其運作：

```text
$ gcc -o 9cc 9cc.c
$ ./9cc 123 > tmp.s
```

