# 第10步：複數文字的區域變數

前面的章節中我們規定變數名為1個字，讓`a`到`z`這26個變數固定存在。在這一章，我們要支援比1個字更長的識別符號，讓其可以編譯如下的程式碼：

```c
foo = 1;
bar = 2 + 3;
return foo + bar; // 回傳6
```

變數現在設計為不需要宣告也可以使用。因此分析器需要一一判斷識別符號在過去使否有被使用過，如果是新出現的話，就要分配堆疊空間給該變數。

首先要修改標記解析器，請讓複數文字組成的識別符號可以作為`TK_IDENT`型的標記來讀取。

變數我們用連結串列來紀錄。以`LVar`結構表示1個變數，並以`locals`這個指標指向開頭。寫成程式碼如下所示：

```c
typedef struct LVar LVar;

// 區域變數型態
struct {
  LVar *next; // 下個變數或NULL
  char *name; // 變數的名稱
  int len;    // 名稱的長度
  int offset; // 從RBP起的offset
} LVar;

// 區域變數
LVar *locals;
```

分析器在出現`TK_IDENT`型的標記時，要確認該識別符號目前為止有沒有出現過。一路爬`locals`這個串列去確認變數名稱，就可以知道該變數是否已經存在。如果變數已經出現過，就直接使用該變數的`offset`。如果是新的變數，就要做出新的`LVar`，設定新的`offset`來使用。

找尋變數名稱的函式如下所示：

```c
// 以名稱搜尋變數。如果找不到就回傳NULL。
LVar *find_lvar(Token *tok) {
  for (LVar *var = locals; var; var = var->next)
    if (var->len == tok->len && !memcmp(tok->str, var->name, var->len))
      return var;
  return NULL;
}
```

分析器則需要追加以下的程式碼：

```c
Token *tok = consume_ident();
if (tok) {
  Node *node = calloc(1, sizeof(Node));
  node->kind = ND_LVAR;

  LVar *lvar = find_lvar(tok);
  if (lvar) {
    node->offset = lvar->offset;
  } else {
    lvar = calloc(1, sizeof(LVar));
    lvar->next = locals;
    lvar->name = tok->str;
    lvar->len = tok->len;
    lvar->offset = locals->offset + 8;
    node->offset = lvar->offset;
    locals = lvar;
  }
  return node;
}
```

{% hint style="info" %}
#### 小知式：機械語言指令的出現頻率

如果看 9cc 輸出的組合語言指令，應該會發現像`mov`或`push`這類移動的指令很多，而像`add`或`mul`這樣「真的進行計算」的指令較少。雖然 9cc 並沒有做最佳化，所以會輸出很多無意義的資料移動指令也是理由之一，但其實有做最佳化的編譯器輸出最多的也是資料移動的指令。在筆者的環境下，對 /bin 底下的所有可執行檔做反組譯，對指令數量進行加總的結果如下圖所示。

可以發現，光是`mov`指令就佔了全部的1/3。電腦是處理資料的機械，而資料處理中最頻繁進行的就是資料的移動。想想「把資料放到正確的位置」是資料處理的本質之一，不少讀者也會認為`mov`指令數多十分合理吧。
{% endhint %}

![&#x6307;&#x4EE4;&#x7684;&#x51FA;&#x73FE;&#x983B;&#x7387;](../.gitbook/assets/index%20%2813%29.svg)

