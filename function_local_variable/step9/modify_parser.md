# 修改分析器

遞迴下降分析法中，只要理解文法，就可以機械性地對應到函式呼叫。因此，考慮到分析器要做的修改，必須思考加上變數名（識別符號）的新文法會長什麼樣。

識別符號我們以`ident`代表，和`num`一樣是終端符號。變數在所有能用數值的地方都可以用，所以我們在原本是`num`的地方改成`num | ident`，文法中使用數值的地方就也可以使用變數了。

終端符號之後，文法還得補上代入式。變數得要可以代入才算是變數，我們想要文法能接受像`a=1`這樣的式子。在此配合C的語法，來讓文法可以接受像`a=b=1`這樣的寫法吧。

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

首先我們先來確認像`42;`或`a=b=2; a+b`這樣的程式是否合乎文法。

