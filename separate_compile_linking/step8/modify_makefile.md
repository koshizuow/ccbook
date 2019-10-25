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



