# 遞迴下降語法分析

只要給定C語言的的生成規則、不停展開，從生成規則的角度來看，這樣可以機械化地生成出任意的C語言程式。但是我們在 9cc 所做的事情，其實是相反。我們是從外面給定C語言程式，展開成輸入的文字列的展開順序，也就是想要知道變成輸入的文字列的語法樹的結構。

針對某種生成規則，只要給予規則，可以機械性地寫出找出符合該規則生成句子的語法樹的程式。接下來要說明的「遞迴下降語法分析法」就是其中一種技巧。

舉例來說想想四則運算的文法。底下是剛剛的四則運算文法：

{% code-tabs %}
{% code-tabs-item title="四則運算的生成規則" %}
```text
expr = mul ("+" mul | "-" mul)*
mul  = term ("*" term | "/" term)*
term = num | "(" expr ")"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

用遞迴下降分析法來寫分析器的基本策略是，把非終端符號和函式一一對應。於是，分析器有`expr`、`mul`、`num`這3個函式。這幾個函式，會負責解析和它們名字一樣規則的標記列。

我們來考慮具體的程式碼。給分析器的輸入是標記列。因為我們想要用分析器建回抽象語法樹，我們要來定義抽象語法樹結點（node）的型態。底下程式碼描述結點的型態：

{% code-tabs %}
{% code-tabs-item title="parser code" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

`lhs`和`rhs`代表 left-hand side 和 right-hand side，也就是左邊和右邊的意思。



