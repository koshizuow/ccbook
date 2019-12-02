# 修改指令產生器

我們來利用至此學到的知識，修改指令產生器讓其可以處理包含變數的式子吧。這次的變更為加上判斷式子是否為左邊值的函式。底下程式碼中的`gen_lval`就是代表該功能。`gen_lval`在所給的結點指向變數時，會計算該變數的記憶體位址，並將其推進堆疊中。除此之外的情況像則會回傳錯誤。據此，我們可以排除像`(a+1)=2`這樣的式子。

當變數為右邊值時，首先會先判斷為左邊值處理，然後把堆疊頂端的計算結果視為記憶體位址，將其中的值讀出。程式碼如下所示：

```c
void gen_lval(Node *node) {
  if (node->kind != ND_LVAR)
    error("代入的左邊值並非變數");

  printf("  mov rax, rbp\n");
  printf("  sub rax, %d\n", node->offset);
  printf("  push rax\n");
}

void gen(Node *node) {
  switch (node->kind) {
  case ND_NUM:
    printf("  push %d\n", node->val);
    return;
  case ND_LVAR:
    gen_lval(node);
    printf("  pop rax\n");
    printf("  mov rax, [rax]\n");
    printf("  push rax\n");
    return;
  case ND_ASSIGN:
    gen_lval(node->lhs);
    gen(node->rhs);

    printf("  pop rdi\n");
    printf("  pop rax\n");
    printf("  mov [rax], rdi\n");
    printf("  push rdi\n");
    return;
  }

  gen(node->lhs);
  gen(node->rhs);

  printf("  pop rdi\n");
  printf("  pop rax\n");

  switch (node->kind) {
  case '+':
    printf("  add rax, rdi\n");
    break;
  case '-':
    printf("  sub rax, rdi\n");
    break;
  case '*':
    printf("  imul rax, rdi\n");
    break;
  case '/':
    printf("  cqo\n");
    printf("  idiv rdi\n");
  }

  printf("  push rax\n");
}
```

