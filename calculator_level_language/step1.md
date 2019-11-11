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
#### 小知識：Intel 語法與 AT&T 語法

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

{% tabs %}
{% tab title="9cc.c" %}
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
{% endtab %}
{% endtabs %}

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

{% tabs %}
{% tab title="test.sh" %}
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
{% endtab %}
{% endtabs %}

請把上述內容存成 test.sh，並下`chmod a+x test.sh`來追加執行權限，然後實際執行 test.sh 看看。如果沒有出現什麼錯誤的話，如下最後 test.sh 會顯示 OK 並結束：

```text
$ ./test.sh
0 => 0
42 => 42
OK
```

如果有發生錯誤的話，test.sh 不會顯示 OK，而是會顯示失敗的測試預期值和實際上的結果：

```text
$ ./test.sh
0 => 0
42 expected, but got 123
```

想要用測試腳本來除錯的時候，下`bash -x`來執行 test.sh 看看。加上`-x`參數時，`bash`會顯示如下的執行紀錄：

```text
$ bash -x test.sh
+ try 0 0
+ expected=0
+ input=0
+ gcc -o 9cc 9cc.c
+ ./9cc 0
+ gcc -o tmp tmp.s
+ ./tmp
+ actual=0
+ '[' 0 '!=' 0 ']'
+ try 42 42
+ expected=42
+ input=42
+ gcc -o 9cc 9cc.c
+ ./9cc 42
+ gcc -o tmp tmp.s
+ ./tmp
+ actual=42
+ '[' 42 '!=' 42 ']'
+ echo OK
OK
```

我們在本書所使用的「測試框架」就只是這樣一個 shell 腳本而已。這腳本和 JUnit 等正式的測試框架比起來看起來可能是過於簡單了，但是就是這個簡單的腳本，和 9cc 本體的簡單程度才能取得平衡，所以簡單點才剛剛好。單元測試的重點，其實就只是自己執行自己寫的程式碼、機械化地比較執行結果而已。所以不要想太難，重要的是執行測試。

## 用`make`來建構

這本書從頭到尾，讀者應該會編譯 9cc 上百次、甚至上千次吧。編出 9cc 的執行檔、然後執行測試腳本這個過程不會變，交給工具來做要方便得多。`make`這個指令就是為了這個目的的標準工具。

`make`會在其執行的目錄底下，尋找名為 Makefile 的檔案並讀取，然後執行該檔案裏面的指令。Makefile 由以冒號結尾的規則，和規則所對應的指令列們構成。底下的 Makefile 就是這一步想要執行的指令的自動化：

{% tabs %}
{% tab title="Makefile" %}
```text
CFLAGS=-std=c11 -g -static

9cc: 9cc.c

test: 9cc
        ./test.sh

clean:
        rm -f 9cc *.o *~ tmp*

.PHONY: test clean
```
{% endtab %}
{% endtabs %}

請把上述的檔案在 9cc.c 的同一個目錄底下存成名為 Makefile 的檔案。如此一來，就可以執行`make`來編出 9cc、執行`make test`來執行測試了。`make`可以知道檔案之間的相依性，所以每當修改 9cc.c 之後，不需要在`make test`前先執行`make`。只要`make`發現 9cc 比 9cc.c 舊，在執行測試前就會重編 9cc。

`make clean`是清除暫存檔的規則。雖然也可以用rm來手動清除，但是如果不小心砍到不想砍的檔案也很麻煩，所以基於提高效率的目的也寫在 Makefile 裡。

此外，在寫 Makefile 時有件事必須注意的是，縮排必須為 tab。如果是4個或8個空白的話會出錯。雖然這是滿不順手的一個文法，但`make`是在1970年代開發的古老工具，這已經被當成是一個傳統。

## 用 git 做版本管理

本書以 git 作為版本管理系統。通過本書會一步一步地完成編譯器，請為這其中的每一步，都做一個 git commit ，並且寫上 commit message。Commit message 可以用中文寫（譯註：原文為日文）沒有關係，請用1行總結來描述實際上做了什麼樣的變更。如果想要寫1行以上的詳細說明時，在第1行後隔一行的空行後再寫下說明。

用 git 做版本管理的只有大家親手所寫的檔案。執行 9cc 所輸出生成的檔案，只要重新執行指令就會再次產生，不需要列為版本管理的對象。反而，如果把這些檔案列入的話，每一個 commit 會有太多不必要的變更，所以有必要從版本管理中除去，不要讓這些檔案進到 repository 裡。

git 可以在名為 .gitignore 的檔案中，寫要被排除在版本管理之外的檔名格式。在 9cc.c 的同一個目錄底下把地下的內容存成 .gitignore 來讓 git 能無視暫存檔案或編輯器的備份檔案：

{% tabs %}
{% tab title=".gitignore" %}
```text
*~
*.o
tmp*
9cc
```
{% endtab %}
{% endtabs %}

第一次使用 git 的讀者們，請告訴 git 你的名字和 email 信箱吧，你跟 git 講的名字和信箱會顯示在 git commit 上。底下是以筆者的名字和信箱設定的範例，請讀者設定自己的名字和信箱：

```text
$ git config --global user.name "Rui Ueyama"
$ git config --global user.email "ruiu@cs.stanford.edu"
```

要 commit 到 git 上，首先得先把變更的檔案以`git add`加入。因為這次是最初的 commit，首先先以`git init`新增一個 repository，然後再把至今為止所寫的所有檔案都以`git add`加入：

```text
$ git init
Initialized empty Git repository in /home/ruiu/9cc
$ git add 9cc.c test.sh Makefile .gitignore
```

然後，執行`git commit`：

```text
$ git commit -m "做出可以編譯一個整數的編譯器"
```

`-m`是用來指定 commit message 的參數。沒有加上`-m`的話，git 會啟動編輯器。執行`git log -p`來確認 commit 有成功：

```text
$ git log -p
commit 0942e68a98a048503eadfee46add3b8b9c7ae8b1 (HEAD -> master)
Author: Rui Ueyama <ruiu@cs.stanford.edu>
Date:   Sat Aug 4 23:12:31 2018 +0000

    做出可以編譯一個整數的編譯器

diff --git a/9cc.c b/9cc.c
new file mode 100644
index 0000000..e6e4599
--- /dev/null
+++ b/9cc.c
@@ -0,0 +1,16 @@
+#include <stdio.h>
+#include <stdlib.h>
+
+int main(int argc, char **argv) {
+  if (argc != 2) {
...
```

最後，把做好的 git repository 上傳到 Github 上吧。並沒有一定要上傳到 GitHub 的必要，但也沒有不上傳到 GitHub 的理由，但上傳到 GitHub 可以作為備份。要上傳到 GitHub 的話，先新增一個 repository（範例為以 rui314 帳號新增一個叫 9cc 的 repository），然後依下列指令把該 repository 作為 remote repository 加入：

```text
$ git remote add origin git@github.com:rui314/9cc.git
```

然後，執行`git push`的話，就會把手邊的 repository 給 push 到 GitHub 上。執行`git push`之後，請打開瀏覽器看看 GitHub 上你的程式碼有沒有成功上傳。

到此為止第1步的編譯器就做完了。這一步的編譯器以編譯器來說是一個簡單過頭的程式，但包含了所有充份作為一個編譯器的要素。雖然讀者可能還無法相信，但今後我們會對這個編譯器追加功能，讓其成長為成熟的C編譯器。首先請好好品味一下所完成的這一步吧。

