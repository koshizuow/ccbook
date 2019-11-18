# 第3步：加入標記解析器（tokenizer）

前一步所做的編譯器有個缺點：如果數入有包含空白的話，遇到空白時編譯器會出錯。例如`5 - 3`這樣包含空白的文字列，在想要讀取`+`或`-`的時候遇到空白，編譯就失敗了。

```text
$ ./9cc '5 - 3' > tmp.s
預料之外的文字: ' '
```

有幾種方法可以解決這個問題。最直覺的一個方法是，在讀`+`或`-`之前跳過空白符號即可。這個方法也沒有什麼問題，但是在這一步我們採用別的方法來解決。這個方法就是：在讀入算式前把輸入的單詞分割。

像日文或英文一樣，數學算式或是程式語言也可以想成是由一列一列的單詞所構成。舉例來說，`5+20-4`可以想成是`5`、`+`、`20`、`-`、`4`這5個單詞所組成。這些「單詞」被稱作「標記」（token）。標記之間的空白符號是為了分開標記，並不是單詞的一部份；所以，把文字列分割成標記列時自然就需要把空白符號拿掉。把文字列分割成標記列的動作就叫作「標記解析」（tokenize）。

把文字列分割成標記列還有其他好處。在把算式分成標記的同時，可以將其分類並賦予型態。舉例來說，`+`或`-`就如字面上是「+」和「-」的符號，而`123`的文字列則是代表123這個數值。在標記化的同時，不是只單純處理文字列的分割，而是要一一解釋這些標記，並且得考慮是在什麼狀況下使用這些標記。

現在處理加減法算式的文法中，標記有「+」、「-」和數值3種型態。考慮到實作的方便性，定義代表標記列結束的特殊型態（就像文字列以`'\0'`結束）可以讓程式更簡潔。這邊我們定義連結串列（linked list），用指標把標記連結起來，就可以處理任意長度的輸入。

以下是稍微變得長了點的程式碼，加入了標記解析器的改良版編譯器：

{% code title="9cc.c" %}
```c
#include <ctype.h>
#include <stdarg.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 標記種類
typedef enum {
  TK_RESERVED, // 符號
  TK_NUM,      // 整數標記
  TK_EOF,      // 代表輸入結束的標記
} TokenKind;

typedef struct Token Token;

// 標記型態
struct Token {
  TokenKind kind; // 標記的型態
  Token *next;    // 下一個輸入標記
  int val;        // kind為TK_NUM時的數值
  char *str;      // 標記文字列
};

// 正在處理的標記
Token *token;

// 處理錯誤的函數
// 取和printf相同的引數
void error(char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  vfprintf(stderr, fmt, ap);
  fprintf(stderr, "\n");
  exit(1);
}

// 下一個標記為符合預期的符號時，讀入一個標記並往下繼續，
// 回傳true。否則回傳false。
bool consume(char op) {
  if (token->kind != TK_RESERVED || token->str[0] != op)
    return false;
  token = token->next;
  return true;
}

// 下一個標記為符合預期的符號時，讀入一個標記並往下繼續。
// 否則警告為錯誤。
void expect(char op) {
  if (token->kind != TK_RESERVED || token->str[0] != op)
    error("不是'%c'", op);
  token = token->next;
}

// 下一個標記為數值時，讀入一個標記並往下繼續，回傳該數值。
// 否則警告為錯誤。
int expect_number() {
  if (token->kind != TK_NUM)
    error("不是數值");
  int val = token->val;
  token = token->next;
  return val;
}

bool at_eof() {
  return token->kind == TK_EOF;
}

// 建立一個新的標記，連結到cur
Token *new_token(TokenKind kind, Token *cur, char *str) {
  Token *tok = calloc(1, sizeof(Token));
  tok->kind = kind;
  tok->str = str;
  cur->next = tok;
  return tok;
}

// 將輸入文字列p作標記解析並回傳標記連結串列
Token *tokenize(char *p) {
  Token head;
  head.next = NULL;
  Token *cur = &head;

  while (*p) {
    // 跳過空白符號
    if (isspace(*p)) {
      p++;
      continue;
    }

    if (*p == '+' || *p == '-') {
      cur = new_token(TK_RESERVED, cur, p++);
      continue;
    }

    if (isdigit(*p)) {
      cur = new_token(TK_NUM, cur, p);
      cur->val = strtol(p, &p, 10);
      continue;
    }

    error("標記解析失敗");
  }

  new_token(TK_EOF, cur, p);
  return head.next;
}

int main(int argc, char **argv) {
  if (argc != 2) {
    error("引數數量錯誤");
    return 1;
  }

  // 標記解析
  token = tokenize(argv[1]);

  // 輸出前半部份組合語言指令
  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");

  // 確認算式必須以數開頭
  // 輸出最初的mov指令
  printf("  mov rax, %d\n", expect_number());

  // 一邊消耗`+ <數>`或`- <數>`的標記，
  // 並輸出組合語言指令
  while (!at_eof()) {
    if (consume('+')) {
      printf("  add rax, %d\n", expect_number());
      continue;
    }

    expect('-');
    printf("  sub rax, %d\n", expect_number());
  }

  printf("  ret\n");
  return 0;
}
```
{% endcode %}

程式碼有將近150行並不算短，但並沒有特別難以理解之處，照順序讀下來想理解應該不成問題。

稍微說明一下上述程式碼用到的程式技巧：

* 以全域變數`token`作為分析器（parser）輸入的標記列，分析器會循著`token`這個連結串列讀入並處理下去。像這樣使用全域變數的程式風格可能稱不上漂亮。但其實像上述這樣，這樣把輸入標記列當成像標準輸入那樣的串流（stream）處理，常常可以提高可讀性，因此我們採用此種風格實作。
* 把實際上會使用到`token`的程式碼分為`consume`和`except`函式，不要讓其他程式碼直接使用到`token`。
* `toeknize`函式的輸出為連結串列。建立連結串列時，先建立一個假的`head`來連結新的要素，最後再回傳`head->next`來簡化程式碼。這個方法宣告`head`的記憶體看起來是白費了，但是區域變數幾乎不花成本，不需在意。
* `calloc`和`malloc`一樣都是動態分配記憶體的函式。但是和`malloc`不同，`calloc`在分配的同時會把分配的記憶體清空為0。這邊省卻初始化為0的麻煩我們使用`calloc`。

這個改良版的編譯器理應可以跳過空白，我們如下在測試中追加1行到 test.sh 中：

{% code title="test.sh" %}
```bash
try 41 " 12 + 34 - 5 "
```
{% endcode %}

Unix 的程式結束碼必須為0~255的數字，在寫測試時，請讓算式的結果落在0~255內。

把測試檔案加到 git repository 後，這一步就完成了。

