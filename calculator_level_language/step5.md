# 第5步：製作可進行四則運算的編譯器

本章會修該到前一章為止我們所做的編譯器，將其擴充為可以計算包含代表優先順序括號的四則運算算式。因為我們已經湊齊了所有必要的材料，要寫的程式只剩下一點點而已。來修改編譯器的main函式看看，讓新做好的分析器（parser）和指令產生器（code generator）可以派上用場吧。程式碼應該會變成像以下這樣：

{% code-tabs %}
{% code-tabs-item title="9cc.c" %}
```c
int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "引數數量錯誤\n");
    return 1;
  }

  // 進行標記解析並分析
  user_input = argv[1];
  token = tokenize(user_input);
  Node *node = expr();

  // 輸出前半部份組合語言指令
  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");

  // 一邊爬抽象語法樹一邊產出指令
  gen(node);

  // 整個算式的結果應該留在堆疊頂部
  // 將其讀到RAX作為函式的返回值
  printf("  pop rax\n");
  printf("  ret\n");
  return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

進行到這個地步，編譯器應該可以正確編譯加減乘除和代表優先順序的括號了。我們來追加幾個測試：

{% code-tabs %}
{% code-tabs-item title="test.sh" %}
```bash
try 47 "5+6*7"
try 15 "5*(9-6)"
try 4 "(3+5)/2"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

此外，至此的程式碼因為說明上方便，一口氣實作了`*`、`/`、`()`，但實際上請不要一口氣實作完。請逐步漸進地、1個 commit 加上一個功能，並且加上實作功能的測試。因為原本的編譯器就可以正確編譯加減法，首先請不要搞壞該功能，先引進有用上抽象語法樹的指令產生器。在這個階段還沒加上新的功能，所以不需要寫新的測試。隨後，請逐步在每一個 commit 上在追加`*`、`/`、`()`功能的同時寫好測試。

{% hint style="info" %}
#### 小知識：9cc的記憶體管理
{% endhint %}

