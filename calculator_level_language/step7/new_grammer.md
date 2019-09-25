# 新的文法

我們來考慮加入了比較運算子的文法會變成什麼樣子吧。至今為止出現的運算子的優先順序由低至高表列如下：

1. `==` `!=`
2. `<` `<=` `>` `>=`
3. `+` `-`
4. `*` `/`
5. 單項`+` 單項`-`
6.  `()`

把優先順序分別對應到不同的終端符號，就可以用生成文法來表現優先順序。像`expr`和`mul`一樣的方法來思考，加上比較運算子的文法如下所示：

```text
expr       = equality
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-")? term
term       = num | "(" expr ")"
```

`equality`代表 `==`和 `!=`，`relational`代表`<`、`<=`、`>`、`>=`。這類非終端符號運用左結合運算子的分析模式，可以直接對應成函式。

除此之外，為了以`equality`表示算式整體，把`expr`和`equality`分離。也可以把`equality`的右邊直接寫在`expr`，但是上述的寫法應該比較好讀。

{% hint style="info" %}
#### 小知識：單純但冗長的程式碼和進階的簡潔程式碼
{% endhint %}

