# 第11步：return

本節會加上`return`的支援，讓我們可以編譯底下的程式碼：

```c
a = 3;
b = 5 * 6 - 8;
return a + b / 2;
```

`return`也可以出現在程式的中途。就像普通的C語言，程式執行到最初的`return`就會結束，從函式中返回。舉例來說底下的程式會回傳最初`return`的值，也就是5：

```c
return 5;
return 8;
```

為了實作這個功能，我們先來想想加上`return`的文法會長什麼樣吧。至今為止我們的陳述都只有式子，新的文法則需要接受`return <式子>;`這樣的陳述。所以文法會變成像這樣：

```text
program = stmt*
stmt    = expr ";"
        | "return" expr ";"
...
```

實作上，標記解析器、分析器、指令產生器全部都需要逐步修改。

首先讓標記解析器可以識別`return`這個標記，以`TK_RETURN`型的標記來代表。像`return`或`while`、`int`這類文法上有特別意義的標記（非關鍵字）數量非常有限，像這樣以不同型態來代表不同標記的作法會比較簡潔。

接下來，要判斷標記是否為`return`，似乎標記解吸器只要判斷剩餘的輸入字串是否以 return 開頭就可以了，但這樣的話像`returnx`這樣的標記會被標記解吸器誤判為`return`和`x`。所以，除了輸入要以 return 開頭，還需要判斷接下來下一個字不是標記的一部份。

判斷所給定的文字是否為組成標記的字，也就是英文字母、數字或底線的函式如下所示：

```c
int is_alnum(char c) {
  return ('a' <= c && c <= 'z') ||
         ('A' <= c && c <= 'Z') ||
         ('0' <= c && c <= '9') ||
         (c == '_');
}
```

利用這個函式，在tokenize裡加上底下的程式碼，就可以把`return`做標記解析為`TK_RETURN`型了：

```c
if (strncmp(p, "return", 6) == 0 && !is_alnum(p[6])) {
  tokens[i].ty = TK_RETURN;
  tokens[i].str = p;
  i++;
  p += 6;
  continue;
}
```

接著對分析器動手，讓其可以分析包含有`TK_RETURN`型的標記列。為此，我們先加上代表`return`的結點型態`ND_RETURN`。然後，修改讀取陳述的函式，讓其可以分析`return`句的文法。如往例，只要把文法直接對應到函式上，就可以成功分析文法了。新的`stmt`函式如下所示：

```c
Node *stmt() {
  Node *node;

  if (consume(TK_RETURN)) {
    node = calloc(1, sizeof(Node));
    node->kind = ND_RETURN;
    node->lhs = expr();
  } else {
    node = expr();
  }

  if (!consume(';'))
    error_at(tokens[pos].str, "此標記不是';'");
  return node;
}
```

`ND_RETURN`型的結點只有在這裡可以產生，所以在此我們不寫新的函式，而是直接在該處進行`malloc`並設定其值。

最後修改指令產生器，讓其可以輸出對應`ND_RETURN`結點的合適指令。新的`gen`函式的一部份如下所示：

```c
void gen(Node *node) {
  if (node->kind == ND_RETURN) {
    gen(node->lhs);
    printf("  pop rax\n");
    printf("  mov rsp, rbp\n");
    printf("  pop rbp\n");
    printf("  ret\n");
    return;
  }
  ...
```

上述程式碼的`gen(node->lhs)`函式呼叫中，會輸出將作為`return`回傳值式子的指令。該段指令應該會在堆疊頂部留下1個值。`gen(node->lhs)`後接著的指令，會把該值從堆疊中彈出並放在 RAX，然後從函式中回傳。

到前一章為止所實作的功能，在函式的最後一定會輸出1個`ret`指令。照本章所說明的方法實作`return`後，就會在`return`句以外還會輸出多的`ret`指令。也可以這些指令統整起來，但此處為了實作簡單，我們允許其輸出複數的`ret`指令。在現在這個時間點，太在意這些瑣碎的細節也沒用，重要的是以實作簡單為優先。雖然能寫難的程式是很有用的能力，但有時讓程式不要變得太難，是更有用的能力。

