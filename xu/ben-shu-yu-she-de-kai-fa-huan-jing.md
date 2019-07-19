# 本書的開發環境

本書預設使用者使用64位元的 Linux 環境運行於 Intel 或是 AMD 等常見的 PC 平台上。請讀者根據所使用的發行版安裝 gcc 或是 make 等常見的開發工具。Ubuntu 的使用者可以根據以下指令安裝本書所使用到的工具：

```text
$ sudo apt install gcc make git binutils libc6-dev
```

macOS 雖然與 Linux 有相當程度的相容性，但並非完全相容（具體來說是不支援「靜態連結」功能）。依照本書的內容去製作對應 macOS 的C編譯器不是說不可能，都是實際做起來會因為一些細節上的不相容而多不少煩惱。筆者並不推薦同時學習「C編譯器的開發技術」和「macOS 與 Linux 的差異」這兩件事情。因為如果在什麼地方卡住了的話，會搞不清楚到底是哪部分的理解錯誤。因此，現階段本書是不支援 macOS 的。macOS 的使用者請準備 Docker 等可以使用 Liunx 的環境。

Windows 和 Linux 的組合語言也並不相容。但是 Windows 10可以讓 Linux 像一個應用程式一樣運作在Windows 上，可以利用該環境在 Windows 上進行開發，名稱是 Windows Subsystem for Linux \(WSL\)。想要在 Windows 上實作本書的內容的話，請安裝 WSL 並在其中進行開發。

{% hint style="info" %}
**小知識：交叉編譯器（Cross compiler）**

執行編譯器的機器被稱之為「host」，而執行編譯器產出的程式碼的機器被稱之為「target」。本書的兩者都是64位元的 Linux 環境，但其實 host 和 target 不一定要是一樣的。

Host 和 target 不同的編譯器就稱之為交叉編譯器。舉例來說，在 Windows 上編譯要在Raspberry Pi 上執行檔案的就是交叉編譯器。交叉編譯器在 target 的機器效能不足以執行編譯器的環境、或是其他特殊的情況下常被使用（譯註：例如許多 RTOS 環境並沒有執行編譯器的執行環境，例如說缺少許多常見 PC 支援的函式庫等等）。
{% endhint %}

