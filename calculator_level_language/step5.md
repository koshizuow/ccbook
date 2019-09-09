# 第5步：製作可進行四則運算的編譯器

本章會修該到前一章為止我們所做的編譯器，將其擴充為可以計算包含代表優先順序括號的四則運算算式。因為我們已經湊齊了所有必要的材料，要寫的程式只剩下一點點而已。來修改編譯器的main函式看看，讓新做好的分析器（parser）和指令產生器（code generator）可以派上用場吧。程式碼應該會變成像以下這樣：

```c
int main(int argc, char **argv) {
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

