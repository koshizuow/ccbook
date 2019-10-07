# 遞迴下降語法分析

只要給定C語言的的生成規則、不停展開，從生成規則的角度來看，這樣可以機械化地生成出任意的C語言程式。但是我們在 9cc 所做的事情，其實是相反。我們是從外面給定C語言程式，展開成輸入的文字列的展開順序，也就是想要知道變成輸入的文字列的語法樹的結構。

針對某種生成規則，只要給予規則，可以機械性地寫出找出符合該規則生成句子的語法樹的程式。接下來要說明的「遞迴下降語法分析法」就是其中一種技巧。

舉例來說想想四則運算的文法。底下是剛剛的四則運算文法：

```text
expr = mul ("+" mul | "-" mul)*
mul  = term ("*" term | "/" term)*
term = num | "(" expr ")"
```

用遞迴下降分析法來寫分析器的基本策略是，把非終端符號和函式一一對應。於是，分析器有`expr`、`mul`、`num`這3個函式。這幾個函式，會負責解析和它們名字一樣規則的標記列。

我們來考慮具體的程式碼。給分析器的輸入是標記列。因為我們想要用分析器建回抽象語法樹，我們要來定義抽象語法樹結點（node）的型態。底下程式碼描述結點的型態：

```c
// 抽象語法樹結點的種類
typedef enum {
  ND_ADD, // +
  ND_SUB, // -
  ND_MUL, // *
  ND_DIV, // /
  ND_NUM, // 整數
} NodeKind;

typedef struct Node Node;

// 抽象語法樹結點的結構
struct Node {
  NodeKind kind; // 結點的型態
  Node *lhs;     // 左邊
  Node *rhs;     // 右邊
  int val;       // kind只在ND_NUM時使用
};
```

`lhs`和`rhs`代表 left-hand side 和 right-hand side，也就是左邊和右邊的意思。

接下來開始定義建立新結點的函式。在這個文法下，四則運算分為有左邊和右邊的2個運算子、和數值這兩種種類，我們配合這兩種種類的運算準備兩個函式：

```c
Node *new_node(NodeKind kind, Node *lhs, Node *rhs) {
  Node *node = calloc(1, sizeof(Node));
  node->kind = kind;
  node->lhs = lhs;
  node->rhs = rhs;
  return node;
}

Node *new_node_num(int val) {
  Node *node = calloc(1, sizeof(Node));
  node->kind = ND_NUM;
  node->val = val;
  return node;
}
```

再來就用這些函數和資料結構來寫我們的分析器吧。`+`和`-`為左結合的運算子。底下為分析左結合運算子的函式：

```c
Node *expr() {
  Node *node = mul();

  for (;;) {
    if (consume('+'))
      node = new_node(ND_ADD, node, mul());
    else if (consume('-'))
      node = new_node(ND_SUB, node, mul());
    else
      return node;
  }
}
```

`consume`是之前定義的函式，這個函式在下一個輸入串流中的標記和函式引數相符時，會前進一個標記並回傳為真。

接下來仔細看`expr`函式。應該可以理解這是把`expr = mul ("+" mul | "-" mul)*`這個生成規則，直接對應到函式呼叫的迴圈中的函式。從上述`expr`函式回傳的抽象語法樹中，運算子是左結合，也就是說回傳的結點是會從左邊往下長的。

我們來接著定義`expr`中用到的`mul`函式。`*`或`/`也是左結合的演算子，所以我們用同樣的方式來寫。程式碼如下：

```c
Node *mul() {
  Node *node = term();

  for (;;) {
    if (consume('*'))
      node = new_node(ND_MUL, node, term());
    else if (consume('/'))
      node = new_node(ND_DIV, node, term());
    else
      return node;
  }
}
```

上述程式的函式間呼叫的關係，原原本本對應了`mul = term ("*" term | "/" term)*`這條生成規則。

最後我們來定義`term`函式。`term`所讀進的不是左結合的運算子，所以雖然不能照上述的方式來寫，我們可以用底下的`term`函式來描述`term = "(" expr ")" | num`這條生成規則：

```c
Node *term() {
  // 下一個標記如果是"("，則應該是"(" expr ")"
  if (consume('(')) {
    Node *node = expr();
    expect(')');
    return node;
  }

  // 否則應該為數值
  return new_node_num(expect_number());
}
```

好，看起來所有的函式都湊齊了，但這真的能分析標記列嗎？乍看之下可能不好懂，但是使用這些函式充份可以分析標記列。舉例來說我們來考慮`1+2*3`這個算式。

首先會呼叫`expr`。我們先假定算式全體符合`expr`（在這裡確實是這樣）開始讀輸入。我們可以看到`expr`→`mul`→`term`的函式呼叫順序，然後讀到`1`這個標記，`expr`會回傳代表`1`的語法樹。

接著，`expr`裡 `consume('+')`判斷式為真，所以會消費一個`+`標記、再度呼叫`mul`。現在剩下的輸入只剩下`2*3`。

`mul`和上次一樣會呼叫`term`，雖然讀到了`2`這個標記，但這次並不會馬上回傳。因為mul裡的 `consume('*')`判斷式為真，`mul`會再次呼叫`term`並讀進`3`這個標記。最後`mul`會回傳代表`2*3`的語法樹。

回傳到`expr`後，會把代表`1`和代表`2*3`的語法樹結合，建立代表`1+2*3`的語法樹，然後作為回傳值跳出`expr`。也就是正確地分析了`1+2*3`這個式子。

函式呼叫的關係和函式們讀進的標記如下圖所示。下圖中，對應`1+2*3`整個式子的是`expr`層，這代表的是讀進整個輸入的`expr`函式呼叫。`expr`上有兩個`mul`，分別代表讀進`1`和`2*3`的`mul`函式呼叫。

![1+2\*3&#x7684;&#x547C;&#x53EB;&#x5716;&#xFF08;call graph&#xFF09;](../../.gitbook/assets/index%20%286%29.svg)

底下是稍微複雜的例子。下圖表示的是分析`1*2+(3+4)`時的函式呼叫關係。

![1\*2+\(3+4\)&#x7684;&#x547C;&#x53EB;&#x5716;](../../.gitbook/assets/index%20%2815%29.svg)

對於不熟悉遞迴的程式設計師來說，上述的函式可能不太好理解。說實話，對應該非常熟悉遞迴的筆者來說，也覺得這樣的程式運作像某種魔法。遞迴的程式，就算搞懂了背後的原理也還是會覺得不可思議，可能遞迴就是這樣的東西吧。請在腦中多追幾次程式碼，確定程式的運作。

上述這樣把1條生成規則對應到1個函式的手法就叫作「遞迴下降語法解析」。上述的分析器中，會先從左邊讀1個標記，再決定是要呼叫哪一個函式或是要回傳。像這樣先讀1個標記的遞迴下降分析器稱為LL\(1\)分析器。而用LL\(1\)分析器可寫出的文法則稱為LL\(1\)文法。（譯註：關於LL法和LL\(k\)分析器請參考[https://zh.wikipedia.org/zh-tw/LL%E5%89%96%E6%9E%90%E5%99%A8](https://zh.wikipedia.org/zh-tw/LL%E5%89%96%E6%9E%90%E5%99%A8)）

