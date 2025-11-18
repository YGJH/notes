**nline Asm 架構**

- __asm__ __volatile__("...": outputs : inputs : clobbers); 是 GCC/Clang 的內嵌組合語法：雙底線 __asm__ 表示這段使用者指定的組合指令，__volatile__ 讓編譯器不可移除或重排（避免它認為這段沒用）。參數分四區，以冒號分隔：輸出運算元、輸入運算元、clobber（被破壞的暫存器）。你現在全部寫在第一區，後三區為空。詳見 testROB.cpp (line 32), testROB.cpp (lines 58-64), testROB.cpp (line 80).

**AT&T vs Intel 語法**

- GCC 內嵌組合預設採 AT&T 語法，和常見的 Intel 語法有幾個差異： (1) 目的運算元在最後；(2) 暫存器前面加 %；(3) 立即值前面加 $；(4) 指令尾端加資料大小後綴，例如 movl 表示 32 位（long）。因此 addl %ebx, %eax 表示「把 %ebx 加到 %eax，結果留在 %eax」（Intel 寫法會是 add eax, ebx）。testROB.cpp (line 32) 的語法就是這樣。

**具體指令與暫存器**
```
movl $1, %eax / movl $2, %ebx: mov=搬移資料；l=32 位；$1 表示常數 1，%eax 是 x86-64 的 32 位暫存器。這兩行把初始值放進 eax 與 ebx (testROB.cpp (lines 58-60))。
addl %ebx, %eax: 把 %ebx 加到 %eax 上，為量測作業產生一串相依加法 (testROB.cpp (line 32)).
movq %rax, %0: q=64 位，%0 是對應 C++ 輸出變數 final_result 的暫存記號，這行把 64 位結果回存到 C++ (testROB.cpp (line 80)).
rdtsc: CPU 指令，讀取時間戳計數器，回傳在 edx:eax，在 rdtsc() 函式中 (testROB.cpp (lines 16-19)) 用 inline asm："rdtsc" : "=a"(lo), "=d"(hi) 指定 lo 對應 eax，hi 對應 edx。
```


**Clobber 清單**

- :"eax","ebx" 告知編譯器這段 asm 修改了 %eax, %ebx（否則它可能以為這些暫存器不變）。這在 addl 的 asm 後面 (testROB.cpp (line 32)) 以及初始化區塊 (testROB.cpp (lines 61-63)) 都宣告，確保編譯器不會錯誤重排或重用這些寄存器。

**模板展開**

- DependentAdds<N>::run() 透過模板遞迴展開，把 addl %ebx, %eax 重複 N 次，確保編譯器真的生成 N 條指令而不是用迴圈 (testROB.cpp (lines 28-44))。這是 C++ 模板技巧，本身不涉組語，但能讓 __asm__ 出現多次。

總結：關鍵是熟悉 GCC 內嵌組語的四段式語法與 AT&T 的記法，並在 clobber 清單中宣告被改動的暫存器，就能控制 x86 指令、量測等低階邏輯。