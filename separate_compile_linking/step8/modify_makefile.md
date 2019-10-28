# 修改 Makefile

已經把程式分成數個檔案了，接著，也該來跟著改 Makefile 了。底下的 Makefile，會把現在所在資料夾的所有 .c 檔全部編譯並連結起來，做出 9cc 這個可執行檔。假設專案的標頭檔只有 9cc.h 這1個檔案，而這個標頭檔會被所有的 .c 檔所引入。

```text
CFLAGS=-std=c11 -g -static
SRCS=$(wildcard *.c)
OBJS=$(SRCS:.c=.o)

9cc: $(OBJS)
        $(CC) -o 9cc $(OBJS) $(LDFLAGS)

$(OBJS): 9cc.h

test: 9cc
        ./test.sh

clean:
        rm -f 9cc *.o *~ tmp*

.PHONY: test clean
```

注意 Makefile 的縮排必須要用 tab 而不能用空白。

`make`是非常強力的工具，精通不是必要的，但是熟悉到可以讀懂上面的 Makefile 的程度的話，在很多地方都能派上用場。我們在此針對上面的 Makefile 進行說明。

Makefile 中，1條規則是由以冒號分隔的行，和以 tab 縮排的0行以上的指令所構成。冒號前的名字稱為「目標」（target）。冒號後面0個以上的檔案名稱被稱為相依檔案。

執行`make foo`，`make`就會試著做出 foo 的檔案。如果所指定的目標檔案已經存在的話，如果目標檔案比相依檔案舊的話，`make`就會重新執行該條目標的規則。因此，就可以只在程式碼有變更的時候才進行重編。

`.PHONY`代表不是真正的目標的特殊名字。`make test`或`make clean`並不是為了做出 test 或 clean 的檔案而執行的，但`make`並不知道這件事，如果剛好有 test 或 clean 的檔案在的話，`make test`或`make clean`就不會做任何事了。只要在在`.PHONY`裡指定像這樣的假目標，就不會真的想要做出該檔案，告訴`make`說：不論是否存在該目標的檔案都要執行目標的規則。

