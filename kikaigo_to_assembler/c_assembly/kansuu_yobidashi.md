# 包含呼叫函式的範例

我們來看一個比較複雜的範例，看看有呼叫函式的程式會被編成什麼樣的組合語言指令。

呼叫函式和一般的跳躍不同，在呼叫結束後必須回到原本呼叫的地方，原本執行中的位址被叫做「回傳位址」（return address）。如果說呼叫只會發生一次的話，隨便找一個暫存器存回傳位址就好了；但是函式呼叫可以一層一層呼叫下去，所以必須把回傳位址存在記憶體裡。實務上，回傳位址被存在記憶體中的堆疊（stack）裡。

堆疊，被實作成只能使用堆疊空間最上方位址所存的一個變數。而這個紀錄堆疊最上方的紀錄空間被稱為「堆疊指標」（stack pointer）。x86-64 中，為了方便寫呼叫函式的程式，提供了堆疊指標專用的暫存器，和使用這個暫存器的指令。往堆疊上堆資料的操作是「push」，而取出堆疊資料的操作是「pop」。

接下來我們來看看操作堆疊的範例。試想以下的C程式碼：

```c
int plus(int x, int y) {
  return x + y;
}

int main() {
  return plus(3, 4);
}
```

而底下是與這個C程式碼對應的組合語言指令：

```text
.intel_syntax noprefix
.global plus, main

plus:
        add rsi, rdi
        mov rax, rsi
        ret

main:
        mov rdi, 3
        mov rsi, 4
        call plus
        ret
```

第一行是指定組合語言文法的指令。由`.global`開始的第二行，是指定`main`和`plus`這兩個可見於程式全體（program scope），非檔案範圍（file scope）的函式的組合語言指令。（譯註：有關 program scope 和 file scope，請參考C語言的相關資料。）現階段可以暫時忽略沒有關係。

首先來看`main`。

