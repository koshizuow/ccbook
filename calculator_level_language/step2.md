# 第2步：製作可以算加減法的編譯器

在這一步，將擴充前一步製作的編譯器，讓其不是只能接受像42這樣的值，而是可以輸入像 `2+11`或 `5+20-4`這種包含加減法計算的算式。

像`5+20-4`這樣的算式，也可以在編譯時先算好，然後把結果的21塞進組合語言指令內，但那就不是編譯而是像直譯（interpret）了，所以應該要輸出可以在執行時計算加減法的組合語言指令。加法和減法的組合語言指令分別是`add`和`sub`。`add`會讀進兩個暫存器的值，執行加法後把結果寫回第1引數的暫存器內。`sub`和`add`基本上一樣，只是執行的是減法。使用這些指令，我們就可以編譯`5+20-4`為：

{% code title="tmp.s" %}
```text
.intel_syntax noprefix
.global main

main:
        mov rax, 5
        add rax, 20
        sub rax, 4
        ret
```
{% endcode %}

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
* 隨後，有0個以上的「項」
* 項為`+`後面跟隨著數字、或`-`後面跟隨著數字

根據這個定義直接用C語言實作如以下的程式：

{% code title="9cc.c" %}
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
{% endcode %}

程式變得稍長了些，但前半部份和`ret`那行都是和之前一樣的。中間追加了把項讀入的程式碼。這是不是只讀進1個數字的程式，所以在讀進數字後，需要知道要讀到哪裡為止。如果用`atoi`的話，因為`atoi`不會回傳所讀進的文字數目，所以用`atoi`會不知道要從哪裡開始讀取項。因此這邊使用C語言標準的`strtol`函式來實作。

`strtol`函式在讀進數值之後，會更新第2引數的指標位置，指向讀進的最後一個文字的下一個文字。於是，在讀進1個數值之後，如果下一個文字是`+`或`-`，`p`就應該是指向該文字。上述程式就是利用這個功能，在`while`迴圈中逐項讀取，每讀到1個項就輸出1行組合語言指令。

接下來趕緊來試試看改造版的編譯器吧。一更新 9cc.c，只要下`make`指令就可以產生出新的 9cc 執行檔。以下為執行的範例：

```text
$ make
$ ./9cc '5+20-4'
.intel_syntax noprefix
.global main
main:
  mov rax, 5
  add rax, 20
  sub rax, 4
  ret
```

看來有順利輸出組合語言指令。為了測試新的功能，我們在 test.sh 加上新的1行測試：

{% code title="test.sh" %}
```bash
try 21 "5+20-4"
```
{% endcode %}

完成到這裡，把至今為止的變更 commit 到 git 吧。執行以下指令來 commit：

```text
$ git add test.sh 9cc.c
$ git commit
```

執行`git commit`會開啟編輯器，請輸入「新增加法和減法」後儲存、關閉編輯器程式。接著請輸入 `git log -p`指令確認看看 commit 有順利完成。最後，執行`git push`把 commit 給 push 上 GitHub 後，這一步就完成了！

