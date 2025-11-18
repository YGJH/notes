## inversion problem
當高優先權的process所需的資源被低優先權的process把持住，就會變成 **高priority process 等 低priority process**
- 解決辦法：
	- priority_inheritance：讓低優先權的process暫時繼承高優先權
	- 這樣可以讓低優先權的趕快釋放資源

## deadlock avoidance scheme:

- **避免** 意味著 發生後解決
- banker algorithm：當偵測到 deadlock 就會檢查是不是有一些辦法把*可以運行結束 並釋放資源*的程式趕快執行完 釋放資源

##  名詞解釋

- **waiting time**: job 在 queue 裡的時間

- **turnaround time**: 從 job arrive 到 completion time
```
Turnaround Time=Completion Time−Arrival Time
```

 - **completion time**: job 運行好的**時間點**

- **response time**: 從使用者輸入指令之後 到第一次執行的時間（ 不只 I/O 而是所有任務的衡量標準）  
```
Response Time=First Start Time−Arrival Time
```

- **Burst time**: cpu 處裡你這個job要到什麽時候才會釋放
