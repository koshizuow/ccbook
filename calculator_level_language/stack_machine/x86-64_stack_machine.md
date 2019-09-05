# 以x86-64實作堆疊機的方法

目前為止討論的都是假想的堆疊機。真正的x86-64並不是堆疊機，而是暫存器機（register machine）。x86-64的運算多半是針對2個暫存器間定義的，並不是對堆疊頂端的2個元素定義。因此，想要把堆疊機的技巧用在x86-64上的話，就要用某些方法以暫存器機來模擬堆疊機才行。

用暫存器機來模擬堆疊機是相對簡單的。只要把堆疊機1個指令完成的動作用複數指令來實作就可以了。

接下來我們來說明怎麼做的具體方法：

首先，準備1個表示堆疊頂端的暫存器，這個暫存器稱為堆疊指標。如果想從堆疊取出2個值的話，就是從堆疊指標所指的位置取出2個元素，然後根據取出的要素數量去改編堆疊指標所指的位置。同理，推入堆疊時就是在寫入堆疊指標所指的記憶體的同時去改變其指標值。

x86-64的 RSP 暫存器就是以當作堆疊指標用而設計出來的。x86-64的`push`和`pop`指令默認會使用 RSP 暫存器作為堆疊指標，在改變 RSP 值的同時一邊讀寫 RSP 所指向的記憶體位置。於是，x86-64指令集要當成堆疊機使用的時候，用 RSP 暫存器會比較直覺。我們就馬上來試試看把`1+2`這個算式用x86-64編譯成堆疊機指令吧。以下x86-64指令為編譯結果：

```text
// 把左邊和右邊推入
push 1
push 2

// 分別把左邊和右邊彈出到 RAX 和 RDI 然後相加
pop rdi
pop rax
add rax, rdi

// 把相加結果推到堆疊裡
push rax
```

x86-64沒有「把 RSP 所指的兩的元素相加」的指令，所以得先讀出來到暫存器中相加，再把計算的結果推回到堆疊裡。上述`add`指令做的就是這個動作。

同理，`2*3+4*5`用x86-64實作如下面的指令所示：

```text
// 計算2*3並把結果推入堆疊
push 2
push 3

pop rdi
pop rax
mul rax, rdi
push rax

// 計算4*5並把結果推進堆疊
push 4
push 5

pop rdi
pop rax
mul rax, rdi
push rax

// 把堆疊頂端的2個值相加
// 也就是計算2*3+4*5
pop rdi
pop rax
add rax, rdi
push rax
```

像這樣使用x86-64的堆疊操作指令，就算是x86-64上，也可以運作地非常像堆疊機。

接下來的`gen`函式就是把該手法直接變成C函式的程式碼：

```c
void gen(Node *node) {
  if (node->kind == ND_NUM) {
    printf("  push %d\n", node->val);
    return;
  }

  gen(node->lhs);
  gen(node->rhs);

  printf("  pop rdi\n");
  printf("  pop rax\n");

  switch (node->kind) {
  case ND_ADD:
    printf("  add rax, rdi\n");
    break;
  case ND_SUB:
    printf("  sub rax, rdi\n");
    break;
  case ND_MUL:
    printf("  imul rax, rdi\n");
    break;
  case ND_DIV:
    printf("  cqo\n");
    printf("  idiv rdi\n");
    break;
  }

  printf("  push rax\n");
}
```

在分析和產生程式的部份沒有什麼需要特別提及的重點，但是上述程式碼有用到較難解的`idiv`指令的部份，說明如下。

`idiv`是有號除法的指令。x86-64的idiv如果是直覺一點的設計的話，應該是像`idiv rax, rdi`的寫法，但是，x86-64並沒有這樣對2個暫存器操作的除法指令。取而代之的是，`idiv`默認會取 RAX 和 RDX 的值，將其視為128位元的整數，然後以引數的暫存器中的64位元做除數相除，把商放進 RAX、餘數放進 RDX。使用`cqo`指令，可以把 RAX 裡放的64位元的值擴充成128位元，並且放在 RAX 和 RDX 中，所以在`idiv`之前會先呼叫`cqo`。

到此為止，堆疊機已經說明完了。讀者到這裡的進度，應該已經可以做到複雜的語法分析，然後把語法分析出的抽象語法樹寫成機械語言。為了活用這些知識，我們回到製作編譯器的作業吧！

{% hint style="info" %}
#### 小知識：編譯器的最佳化
{% endhint %}

