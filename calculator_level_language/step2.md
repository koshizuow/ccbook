# 第2步：製作可以算加減法的編譯器

在這一步，將擴充前一步製作的編譯器，讓其不是只能接受像42這樣的值，而是可以輸入像 `2+11`或 `5+20-4`這種包含加減法計算的算式。

像`5+20-4`這樣的算式，也可以在編譯時先算好，然後把結果的21塞進組合語言指令內，但那就不是編譯而是像直譯（interpret）了，所以應該要輸出可以在執行時計算加減法的組合語言指令。加法和減法的組合語言指令分別是`add`和`sub`。`add`會讀進兩個暫存器的值，執行加法後把結果寫回第1引數的暫存器內。`sub`和`add`基本上一樣，只是執行的是減法。使用這些指令，我們就可以編譯`5+20-4`為：

{% code-tabs %}
{% code-tabs-item title="tmp.s" %}
```text
.intel_syntax noprefix
.global main

main:
        mov rax, 5
        add rax, 20
        sub rax, 4
        ret
```
{% endcode-tabs-item %}
{% endcode-tabs %}

上述的組合語言中，先以`mov`指令把 RAX 設成5，再把 RAX 加上20，最後從中減去4。在執行`ret`指令的時候 RAX 的值應該就是`5+20-4`，也就是21。實際執行來確認看看吧。把上述檔案存成 tmp.s 並組譯、執行看看：

```text
$ gcc -o tmp tmp.s
$ ./tmp
$ echo $?
21
```

如上所述，正確顯示21。

接下來，該如何做出這樣的組合語言檔案呢？把包含加減法的算試想成是「語言」的話，我們可以這樣定義這個語言：

* 最初有一個數字
* 隨後，有0個以上的「項目」
* 項目為`+`後面跟隨著數字、或`-`後面跟隨著數字

根據這個定義直接用C語言實作如以下的程式：

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

  char *p = argv[1];

  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");
  printf("  mov rax, %ld\n", strtol(p, &p, 10));

  while (*p) {
    if (*p == '+') {
      p++;
      printf("  add rax, %ld\n", strtol(p, &p, 10));
      continue;
    }

    if (*p == '-') {
      p++;
      printf("  sub rax, %ld\n", strtol(p, &p, 10));
      continue;
    }

    fprintf(stderr, "預料之外的文字: '%c'\n", *p);
    return 1;
  }

  printf("  ret\n");
  return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



