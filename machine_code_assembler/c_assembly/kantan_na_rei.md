# 簡單的範例

為了大致掌握C編譯器輸出的長相，我們試著比較一下C的程式和對應的組合語言程式。我們先考慮一下底下這個最簡單的程式：

```c
int main() {
  return 42;
}
```

把上述程式存檔成 test.c，並以以下指令編譯，執行確認`main`有傳回42。

```text
$ gcc -o test1 test1.c
$ ./test1
$ echo $?
42
```

C程式把`main`函式的回傳值作為整個程式的結束碼。程式的結束碼並不會顯示在畫面上，但是 shell 會默默把結果放在`$?`這個變數，在指令結束後可以呼叫`echo`來顯示`$?`的值，確認該指令的結束碼。在範例中可以看到有正確回傳42。

現在，我們來看看這隻C程式對應的組合語言如下：

```text
.intel_syntax noprefix
.global main
main:
        mov rax, 42
        ret
```

在上面的組合語言中，在全域（global）標籤中定義了`main`，隨之就是`main`函式的指令碼。在這邊把42這個值設定到 RAX 這個暫存器後，從`main`裡傳回跳出。可以放整數的暫存器包含 RAX 在內共有16個，但把回傳值放進 RAX 是一般的共識，所以在這邊把回傳值放進 RAX。

接下來實際組譯上述的組合語言並執行看看。組合語言的附檔名是 .s，我們把上述組合語言存成 test2.s，並執行下列指令：

```text
$ gcc -o test2 test2.s
$ ./test2
$ echo $?
42
```

可以看到和C程式的時候一樣結束碼是42。

我們可以先簡單想成是：C編譯器就是讀到 test1.c 的C程式碼時，就會輸出 test2.s 組合語言指令的程式。
