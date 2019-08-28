# 包含遞迴的生成規則

寫遞迴的文法在生成文法中也是稀鬆平常的事。底下是在四則運算加上代表優先順序的括號的文法生成規則：

```text
expr = mul ("+" mul | "-" mul)*
mul  = term ("*" term | "/" term)*
term = num | "(" expr ")"
```



