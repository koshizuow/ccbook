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

這時讀者可能會想說：「這樣的程式才不算編譯器」，說實話筆者也這樣想。但是，這個程式可以對應由1個整數構成的語言，並且根據該數值輸出對應的程式，照定義來說它有充分的資格能作為一個編譯器。而這樣一個簡單的程式，只要一路改寫很快就能做很複雜的事情了，但首先得先完成這一步。

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

第1行是編譯 9cc.c 做出可執行的 9cc 檔案。第二行是輸入123給 9cc 來輸出組合語言，然後把結果寫進 tmp.s 這個檔案裡。現在來看看 tmp.s 的內容：

```text
$ cat tmp.s
.intel_syntax noprefix
.global main
main:
  mov rax, 123
  ret
```

輸出結果看來沒什麼問題。如果把這個組合語言檔給組譯器就可以輸出可執行檔了。

在 Unix 裡 cc（或 gcc）不只是 C/C++，同時也是很多語言的前端（front-end），會根據輸入檔案的副檔名來執行對應的編譯器或組譯器。所以，就像編譯 9cc 時一樣，把 .s 副檔名的組合語言檔案輸入給 gcc，就會執行組譯。以下就是執行組譯器輸出執行檔的範例：

```text
$ gcc -o tmp tmp.s
$ ./tmp
$ echo $?
123
```

在 shell 中可以用`$?`來使用前一個指令的結束碼。在上面的例子中，顯示和我們給 9cc 的123一致的結果，也就代表程式有正常運作。讀者也試著在123以外從0~255的範圍內輸入看看（Unix 的程式結束碼的範圍為0~255），實際確認看看 9cc 正常運作的結果。

## 製作單元測試

寫程式自娛時沒寫過測試的讀者想必也是大有人在，但是本書在擴充編譯器的同時會寫測試程式碼用的程式。雖然一開始可能會覺得寫測試很麻煩，但想必馬上就會感受到測試的重要性了。不寫測試的話，每次都得非得手動操作測試才行，這比寫測試還要來的麻煩多了。

筆者認為寫測試常讓人感到麻煩之處大概是來自於：測試的框架（framework）太過於殺雞用牛刀、測試的觀念常常太過制式而教條化等。例如說 JUnit 這樣的測試框架雖然有很多方便的功能，但是要引進該框架得先記得使用方法，顯得很麻煩。因此，本章我們不會引進測試框架。取而代之的是用 shell 腳本寫一個簡單的手寫「測試框架」來寫測試。

以下就是名為 test.sh 的測試用 shell 腳本。shell 函式`try`會從引數中把輸入值和預期的結果兩個引數抓下來、把9cc的結果拿去組譯、把結果和期待的值做比較。在這個 shell 腳本中，定義完`try`之後，會用0和42兩個值來確認是否有正常編譯：

```bash
#!/bin/bash
try() {
  expected="$1"
  input="$2"

  ./9cc "$input" > tmp.s
  gcc -o tmp tmp.s
  ./tmp
  actual="$?"

  if [ "$actual" = "$expected" ]; then
    echo "$input => $actual"
  else
    echo "$input => $expected expected, but got $actual"
    exit 1
  fi
}

try 0 0
try 42 42

echo OK
```



