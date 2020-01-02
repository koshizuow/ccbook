# 第11步：return

本節會追加`return`的支援，讓我們可以編譯底下的程式碼：

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

首先讓標記解析器可以識別`return`這個標記，以`TK_RETURN`型的標記來代表。像`return`或`while`、`int`這樣在文法上有特別意義的標記（非關鍵字）的數量非常有限，像這樣以不同型態來代表不同標記的作法會比較簡單。

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



