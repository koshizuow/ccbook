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

