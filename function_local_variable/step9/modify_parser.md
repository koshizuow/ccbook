# 修改分析器

遞迴下降分析法中，只要理解文法，就可以機械性地對應到函式呼叫。因此，考慮到分析器要做的修改，必須思考加上變數名（識別符號）的新文法會長什麼樣。

識別符號我們以`ident`代表，和`num`一樣是終端符號。變數在所有能用數值的地方都可以用，所以我們在原本是`num`的地方改成`num | ident`，文法中使用數值的地方就也可以使用變數了。

同時，文法還得補上代入式。變數得要可以代入才算是變數，我們想要文法能接受像`a=1`這樣的式子。在此配合C的語法，來讓文法可以接受像`a=b=1`這樣的寫法吧。

還有，我們希望可以用分號來區分不同敘述（statement），最終文法會變成以下這樣：

```text
program    = stmt*
stmt       = expr ";"
expr       = assign
assign     = equality ("=" assign)?
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-")? primary
primary    = num | ident | "(" expr ")"
```

首先請先確認看看像`42;`或`a=b=2; a+b`這樣的程式是否合乎文法。然後，請修改我們到目前做好的分析器，讓其能分析上述的文法吧。在現階段，像`a+1=5`這樣的式子也可以普通地被分析出來，這是正常運作的結果。要排除像那樣不合法的式子，我們等下一階段再來做。改造分析器，並沒有特別難解之處，只要照至今為止的作法讓文法的成員原原本本對應到函式呼叫應該就可以完成。

因為要加上以分號來區分複數式子，我們得把分析的結果存成複數的結點才行。在這個階段，請準備如下的全域陣列，在其中照分析出來的順序把結點存入。在最後一個結點填上`NULL`，就可以知道尾端在哪裡。新增的程式碼的部份如下所示：

```c
Node *code[100];

Node *assign() {
  Node *node = equality();
  if (consume("="))
    node = new_node(ND_ASSIGN, node, assign());
  return node;
}

Node *expr() {
  return assign();
}

Node *stmt() {
  Node *node = expr();
  expect(";");
  return node;
}

void program() {
  int i = 0;
  while (!at_eof())
    code[i++] = stmt();
  code[i] = NULL;
}
```

抽象語法樹也需要能代表「表示區域變數的結點」。為此，將區域變數這個新的型態作為結點的新成員加入。應該變得像會如下範例所示。此資料結構中，分析器遇到識別符號標記會做出`ND_LVAR`型的結點回傳。

```c
// 抽象語法樹結點的種類
typedef enum {
  ND_ADD,  // +
  ND_SUB,  // -
  ND_MUL,  // *
  ND_DIV,  // /
  ND_LVAR, // 區域變數
  ND_NUM,  // 整數
} NodeKind;

typedef struct Node Node;

// 抽象語法樹結點的結構
struct Node {
  NodeKind kind; // 結點的型態
  Node *lhs;     // 左邊
  Node *rhs;     // 右邊
  int val;       // 只在kind為ND_NUM時使用
  int offset;    // 只在kind為ND_LVAR時使用
};
```

`offset`這個成員代表離基底指標的偏移量（離基底指標的距離）。現階段，變數`a`為 RBP-8、變數`b`為 RBP-16……如此，不同名字的區域變數有固定的位置，偏移量可以在語法分析的階段就決定好。以下為讀進識別符號回傳ND\_LVAR型結點的程式碼：

```c
Node *primary() {
  ...

  Token *tok = consume_ident();
  if (tok) {
    Node *node = calloc(1, sizeof(Node));
    node->kind = ND_LVAR;
    node->offset = (tok->str[0] - 'a' + 1) * 8;
    return node;
  }

  ...
```

{% hint style="info" %}
#### 小知識：ASCII碼

在ASCII碼中，以0~127的數值對應到不同文字。ASCII碼和文字的對應如下表所示：

|  |  |  |  |  |  |  |  |  |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | NUL | SOH | STX | ETX | EOT | ENQ | ACK | BEL |
| 8 | BS | HT | NL | VT | NP | CR | SO | SI |
| 16 | DLE | DC1 | DC2 | DC3 | DC4 | NAK | SYN | ETB |
| 24 | CAN | EM | SUB | ESC | FS | GS | RS | US |
| 32 | sp | ! | " | \# | $ | % | & | ' |
| 40 | \( | \) | \* | + | , | - | . | / |
| 48 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| 56 | 8 | 9 | : | ; | &lt; | = | &gt; | ? |
| 64 | @ | A | B | C | D | E | F | G |
| 72 | H | I | J | K | L | M | N | O |
| 80 | P | Q | R | S | T | U | V | W |
| 88 | X | Y | Z | \[ | \ | \] | ^ | \_ |
| 96 | \` | a | b | c | d | e | f | g |
| 104 | h | i | j | k | l | m | n | o |
| 112 | p | q | r | s | t | u | v | w |
| 120 | x | y | z | { | \| | } | ~ | DEL |

0~31為控制符號。現在除了`NUL`和換行符號之外，幾乎沒有使用到這類控制符號的機會，這些控制符號成了佔據編碼表的最佳位置的無用存在，但是ASCII碼制定是在1963年，在當時，實際上是非常常用到這類控制符號的。在制定的時候，還有過比起放進小寫字母應該放進更多控制符號的提案（註）。

48~58是數字、65~90為大寫字母、97~122則是對應到小寫字母。可以發現，這些是以連續的方式對應到編碼，也就是說，0123456789或abcdefg...在編碼上是連續的。讀者可能會覺得定義上連續的文字像這樣以連續方式配置是理所當然的，但是當時主流的 [EBCDIC](https://ja.wikipedia.org/wiki/EBCDIC) 編碼受到打孔機卡的影響，字母在編碼上是不連續的。

在C語言中，文字只是小的整數型態的值，把對應文字的編碼用數值的方式寫也不改變其意義。也就是說，在 ASCII 下，`'a'`和`97`、`'0'`和`48`等價。上述程式中有以數值的意義用文字減去a的式子，這樣可以算出給定的文字和 a 距離有多遠。這是因為在 ASCII 編碼中字母是連續排列才能使用的技巧。
{% endhint %}

> [https://en.wikipedia.org/wiki/ASCII\#cite\_note-Mackenzie\_1980-1](https://en.wikipedia.org/wiki/ASCII#cite_note-Mackenzie_1980-1)
>
> > There was some debate at the time whether there should be more control characters rather than the lowercase alphabet.

