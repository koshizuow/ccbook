# 堆疊上的變數空間

C的變數存在於記憶體中，也可以說，變數就是把記憶體位址取了名字而已。把記憶體位址取名之後，就可以不必講「存取記憶體的0x6080號位置」而是能以「存取變數`a`」的方式來表現。

但是，每一次的函式呼，其區域變數都必須獨立存在。如果只考慮到實作的方便，舉例來說像「函式 f 的變數 a 放在 0x6080 號位置」這樣決定位址非常簡單，但是如果 f 是遞迴呼叫的話就無法正常運作。為了讓區域變數在每次函式呼叫可以獨立，C語言中變數會放在堆疊裡。

我們來以具體的例子想看看堆疊裡的內容。有1個函式`f`有`a`和`b`2個變數，假設有某個其他函式呼叫了`f`。呼叫函式的`call`指令會把回傳位址推進堆疊，所以在`f`被呼叫的時間點的堆疊頂部，放的就是該回傳位址。除此之外，其他還有原本就放在堆疊裡的某些值，這些值具體是什麼在此處並不重要，我們以「......」表示。圖示如下：

| ...... |  |
| :---: | :--- |
| 回傳位址 | ← RSP |

此處以「← RSP」來表示現在 RSP 暫存器的值是指向該位址。`a`和`b`的大小各為8 bytes。

堆疊成長的方向為向下成長。眼下要保留`a`和`b`的空間，得將 RSP 向下2個變數的份，總共加上16 bytes。執行之後會變成以下這樣：

| ...... |  |
| :---: | :--- |
| 回傳位址 |  |
| `a` |  |
| `b` | ← RSP |

如上分配完之後，用 RSP+8 的值就可以存取 a、用 RSP 的值就可以存取`b`了。像這樣在每次函式呼叫保留下來的記憶體空間，稱作「函式框架」（function frame）或「活動紀錄」（activation records）。（譯注：台灣較少使用這兩個名稱，可以搜尋「呼叫堆疊」（call stack）會比較容易找到相關資訊。）

其他函式看不到 RSP 的值要改多少 bytes、和要用什麼樣的順序去放變數，所以可以根據編譯器實作上的方便來自己決定作法。

區域變數的實作，基本上就是這麼單純。

但是，這個作法有一個缺點，所以實際實作的時候還會多用到一個暫存器。想想我們的編譯器（其他編譯器也一樣）在執行期間 RSP 會可能會改變這回事。9cc 在計算式子的過程中會使用 RSP 來推入／彈出堆疊，所以 RSP 的值其實變更地相當頻繁。因此，無法從 RSP 透過固定的 offset 去存取`a`或`b`。

為了解決這個問題，最常見的解決方法就是在 RSP 之外，另外準備一個隨時指著函式框架起點的暫存器。這樣的暫存器一般稱為「基底暫存器」（base register），而其裏面所放的值稱作「基底指標」（base pointer）。習慣上，x86-64 中會用 RBP 暫存器作為基底暫存器。

在函式執行過程中，基底暫存器的值不能改變（說起來這就是準備基底暫存器的理由）。不能發生如果函式呼叫了別的函式，等到回傳之後變成了其他的值這種情況，所以在每次呼叫函式都得保存原本的基底指標，在回傳前要寫回原本的值。

使用了基底指標的函式呼叫其堆疊狀態如下圖所示。假定由有區域變數`x`和`y`的函式`g`呼叫了`f`。在`g`執行期間，其堆疊會像下面這樣：

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP | ← RBP |
| `x` |  |
| `y` | ← RSP |

接著呼叫`f`之後會變成：

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP |  |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RBP |
| `a` |  |
| `b` | ← RSP |

如此，我們隨時都可以用 RBP-8 來存取`a`、用 RBP-16 來存取`b`。來想想要做出這樣的堆疊狀態具體來說要用什麼樣的組合語言指令，我們只要在各自函式的開頭，編譯器輸出以下的組合語言指令就可以了：

```text
push rbp
mov rbp, rsp
sub rsp, 16
```

像這樣編譯器在函式的開頭輸出的固定指令稱為「序言」（prologue）。除此之外，16這個數字其實是根據每個函式所不同，需要配合其變數的個數和大小而改變。

我們來確認在 RSP 指向返回位址的狀態下，執行以上的指令是否能如預期地保留函式框架下來。底下以1個指令為單位顯示狀態。

1. 以`call`呼叫`f`後當下的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP | ← RBP |
| `x` |  |
| `y` |  |
| `f`的回傳位址 | ← RSP |

2. 執行`push rbp`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP | ← RBP |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RSP |

3. 執行`mov rbp, rsp`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP |  |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RSP, RBP |

4. 執行`sub rsp, 16`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP |  |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RBP |
| `a` |  |
| `b` | ← RSP |

從函式回傳的時候，把 RBP 寫回原本的值，讓 RSP 指向回傳位址的狀態下，呼叫`ret`（`ret`指令為從堆疊中彈出位址，並跳到那裡的指令）。可以很簡結地寫成如下這段程式：

```text
mov rsp, rbp
pop rbp
ret
```

像這樣編譯器在函式最後輸出的固定指令稱作「結語」（epilogue）。

結語執行時的堆疊狀態如下所示。RSP 所指位址以下的堆疊空間，已經可以視為是無效資料了，圖上將予以省略。

1. `mov rsp, rbp`執行前的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP |  |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RBP |
| `a` |  |
| `b` | ← RSP |

2. 執行`mov rsp, rbp`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP |  |
| `x` |  |
| `y` |  |
| `f`的回傳位址 |  |
| `f`被呼叫時的 RBP | ← RSP, RBP |

3. 執行`pop rbp`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP | ← RBP |
| `x` |  |
| `y` |  |
| `f`的回傳位址 | ← RSP |

4. 執行`ret`後的堆疊

| ...... |  |
| :---: | :--- |
| `g`的回傳位址 |  |
| `g`被呼叫時的 RBP | ← RBP |
| `x` |  |
| `y` | ← RSP |

如此，執行結語就可以把呼叫者函式`g`的堆疊回復如初。`call`指令會把`call`指令本身的下一個指令的位址給推進堆疊，結語的`ret`會將其彈出並跳到該位址，於是會從`call`指令的下一個指令開始繼續執行`g`函式。這樣的運作，和我們所知的函式的運作方式完全地一致。

函式的呼叫和函式的區域變數，就是像這樣實作的。

{% hint style="info" %}
#### 小知識：堆疊的成長方向

x86-64 的堆疊，如上所述是由大往小的方向成長。是不是感覺方向好像反了，堆疊向上成長似乎比較自然，到底為什麼會設繼承堆疊向下成長呢？

其實堆疊在技術上不一定要往下長。雖然實際上主流的 CPU 和 ABI 是把堆疊的起點設在高的位址向下成長，但也有極少數的架構堆疊是反方向長的。舉例來說8051微控制器、PA-RISC 的 ABI（註1），還有 Multics（註2） 等堆疊都是往高位址方向成長的。

但是，其實堆疊向下成長，並不是不自然的問題設計。

剛開機時，CPU 從全白的狀態開始執行程式時，開始執行的位址一般來說是由 CPU 規格所決定的。常見的設計中，CPU 會從像0之類的低位址開始執行。如此，一般程式的指令就會放在較低的位址。為了避免堆疊和程式指令重疊到，儘可能將兩者離的愈遠愈好，就會把堆疊往高位址擺，設計程記憶體位址空間是往中間成長。於是，堆疊就變成往下成長了。

當然，能再想出和上述 CPU 不同的設計，讓堆疊向上成長變成比較自然的配置。但老實說這個問題兩種方法都可行，實際上在業界普遍的認知中，機器的堆疊是就是向下成長的。
{% endhint %}

> 1. [https://parisc.wiki.kernel.org/images-parisc/b/b2/Rad\_11\_0\_32.pdf](https://parisc.wiki.kernel.org/images-parisc/b/b2/Rad_11_0_32.pdf) The 32-bit PA-RISC run-time architecture document, v. 1.0 for HP-UX 11.0, 2.2.3章
>
>    > When a process is initiated by the operating system, a virtual address range is allocated for that process to be used for the call stack, and the stack pointer \(GR 30\) is initialized to point to the low end of this range. As procedures are called, the stack pointer is incremented to allow the called procedure frame to exist at the address below the stack pointer. When procedures are exited, the stack pointer is decremented by the same amount.
>
> 2. [https://www.acsac.org/2002/papers/classic-multics.pdf](https://www.acsac.org/2002/papers/classic-multics.pdf)
>
>    > Thirty Years Later: Lessons from the Multics Security Evaluation, "Third, stacks on the Multics processors grew in the positive direction, rather than the negative direction. This meant that if you actually accomplished a buffer overflow, you would be overwriting unused stack frames, rather than your own return pointer, making exploitation much more difficult.



