# 修改主函式

我們已經湊齊了所有的零件，現在該來修改`main`函式，讓編譯器跑看看了。

```c
int main(int argc, char **argv) {
  if (argc != 2) {
    error("引數數量錯誤");
    return 1;
  }

  // 進行標記解析並分析
  // 將結果存進code中
  user_input = argv[1];
  tokenize();
  program();

  // 輸出前半段組合語言指令
  printf(".intel_syntax noprefix\n");
  printf(".global main\n");
  printf("main:\n");

  // 前言
  // 預留26個變數的空間
  printf("  push rbp\n");
  printf("  mov rbp, rsp\n");
  printf("  sub rsp, 208\n");

  // 從頭開始生成指令
  for (int i = 0; code[i]; i++) {
    gen(code[i]);

    // 作為式子的判斷運算結果，在堆疊中會留下一個值
    // 為了不要讓堆疊溢出，將其彈出
    printf("  pop rax\n");
  }

  // 結語
  // 最後式子的結果會留在 RAX 作為回傳值
  printf("  mov rsp, rbp\n");
  printf("  pop rbp\n");
  printf("  ret\n");
  return 0;
}
```

