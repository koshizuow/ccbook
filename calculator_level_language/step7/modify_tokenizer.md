# 修改標記解析器

至今為止，我們處理的符號標記長度都為1，我們也根據此來寫程式碼，但是要處理像`==`這樣的比較演算子就得把程式改得更通用。為了可以在標記紀錄字串的長度，我們要在`Token`結構中加上`len`這個成員。新的資料結構如下：

```c
struct Token {
  TokenKind kind; // 標記的型態
  Token *next;    // 下一個輸入標記
  int val;        // kind為TK_NUM時的數值
  char *str;      // 標記文字列
  int len;        // 標記長度
};
```

修改資料結構的同時，`consume`和`expect`函式也要修改成不只可以處理單個文字而是要可以處理字串。修改如下所示：

```c
bool consume(char *op) {
  if (token->kind != TK_RESERVED ||
      strlen(op) != token->len ||
      memcmp(token->str, op, token->len))
    return false;
  token = token->next;
  return true;
}
```

對由複數文字組成的標記進行標記解析的時候，需要從長的標記開始解析。舉例來說，下一個文字是&gt;的話，首先要先像 `strncmp(p, ">=", 2)`這樣先確認有沒有可能是`>=`，否則如果先判斷`>`的話`>=`就會被誤認為是`>`和`=`兩個標記。

