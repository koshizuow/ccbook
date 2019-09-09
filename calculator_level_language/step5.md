# 第5步：製作可進行四則運算的編譯器

本章會修該到前一章為止我們所做的編譯器，將其擴充為可以計算包含代表優先順序括號的四則運算算式。因為我們已經湊齊了所有必要的材料，要寫的程式只剩下一點點而已。來修改編譯器的main函式看看，讓新做好的分析器（parser）和指令產生器（code generator）可以派上用場吧。程式碼應該會變成像以下這樣：

{% code-tabs %}
{% code-tabs-item title="9cc.c" %}
```c
int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "引數數量錯誤\n");
    return 1;
  }

  // 進行標記解析並分析
  user_input = argv[1];
  token = tokenize(user_input);
  Node *node = expr();

  // 輸出前半部份組合語言指令
  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");

  // 一邊爬抽象語法樹一邊產出指令
  gen(node);

  // 整個算式的結果應該留在堆疊頂部
  // 將其讀到RAX作為函式的返回值
  printf("  pop rax\n");
  printf("  ret\n");
  return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

進行到這個地步，編譯器應該可以正確編譯加減乘除和代表優先順序的括號了。我們來追加幾個測試：

{% code-tabs %}
{% code-tabs-item title="test.sh" %}
```bash
try 47 "5+6*7"
try 15 "5*(9-6)"
try 4 "(3+5)/2"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

此外，至此的程式碼因為說明上方便，一口氣實作了`*`、`/`、`()`，但實際上請不要一口氣實作完。請逐步漸進地、1個 commit 加上一個功能，並且加上實作功能的測試。因為原本的編譯器就可以正確編譯加減法，首先請不要搞壞該功能，先引進有用上抽象語法樹的指令產生器。在這個階段還沒加上新的功能，所以不需要寫新的測試。隨後，請逐步在每一個 commit 上在追加`*`、`/`、`()`功能的同時寫好測試。

{% hint style="info" %}
#### 小知識：9cc的記憶體管理

本書的讀者讀到這邊，或許會覺得「這個編譯器的記憶體管理到底是怎麼搞的」而感到不可思議。至此為止，我們使用了 `calloc`（與`malloc`類似）但是沒有呼叫`free`。也就是我們並沒有釋放分配的記憶體空間。就算是偷懶，這也太超過了吧？

實際上，這個叫「不做記憶體管理的記憶體管理方針」，是筆者在多方妥協之下，刻意選擇的設計。用這種設計有很多的好處：

首先，不釋放記憶體，就可以像有垃圾回收（garbage collection）機制的程式語言一樣寫程式。也因此，不只是不用寫記憶體管理的程式碼，更可以從根本斷絕手動記憶體管理的難解臭蟲。

再者，因不呼叫`free`而發生問題，如果是在一般PC執行，實際上不可能的。編譯器是讀取1個C語言檔案後輸出組合語言的短命程式，程式結束的時候分配的記憶體空間會被OS自動釋放。於是，問題就只在於總計共使用多少記憶體空間而已，而筆者實測編譯一個頗大的C語言檔案也不過就使用100 MiB 左右而已。所以不呼叫free式很實務且有效的策略。舉例來說，D語言編譯器 DMD 也根據同樣的想法，採用了做`malloc`但是不呼叫`free`的策略。（註）
{% endhint %}

> [http://www.drdobbs.com/cpp/increasing-compiler-speed-by-over-75/240158941](http://www.drdobbs.com/cpp/increasing-compiler-speed-by-over-75/240158941) （連結已失效）
>
> * DMD does memory allocation in a bit of a sneaky way. Since compilers are short-lived programs, and speed is of the essence, DMD just mallocs away, and never frees.

