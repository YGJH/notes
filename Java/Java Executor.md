以下以有條理的方式，分段介紹 Java 的 Executor（執行器）框架，並輔以範例程式碼說明其常見用法。

---

## 1. 為什麼需要 Executor？

傳統上，如果要在 Java 中啟動新執行緒，就得撰寫：

```java
Thread t = new Thread(() -> {
    // 工作內容
});
t.start();
```

這樣做雖然簡單，但當專案對非同步任務、執行緒管理、資源重用、安全關閉都有需求時，手動管理大量執行緒便會變得錯綜複雜、難以維護。

**Executor 框架**提供統一介面與工廠方法，讓你：

1. **集中管理**執行緒生命週期
    
2. **可重複利用**執行緒池（降低建立／銷毀成本）
    
3. **方便關閉**與等待所有任務結束
    
4. **易於擴充**，可支援排程、工作竊取（work-stealing）、Fork/Join 等
    

---

## 2. 核心介面與常見子介面

|介面|功能|
|---|---|
|`Executor`|最基本的執行器，只定義 `void execute(Runnable)`|
|`ExecutorService`|繼承自 `Executor`，提供提交（`submit`）、關閉（`shutdown`）等方法|
|`ScheduledExecutorService`|支援延遲、週期性排程任務|
|`ForkJoinPool`|針對分治演算法（Fork/Join）最佳化的執行器|

---

## 3. 建立各種常見的執行器

使用 `java.util.concurrent.Executors` 中的靜態工廠方法即可快速建立：

|方法|說明|
|---|---|
|`newSingleThreadExecutor()`|單一執行緒池|
|`newFixedThreadPool(int nThreads)`|固定大小執行緒池|
|`newCachedThreadPool()`|可動態擴張／回收的執行緒池|
|`newScheduledThreadPool(int corePoolSize)`|可排程延遲／週期任務的執行緒池|
|`newWorkStealingPool()`|採用工作竊取的 `ForkJoinPool`|

```java
ExecutorService fixedPool = Executors.newFixedThreadPool(4);
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
```

---

## 4. 提交與執行任務

### 4.1 使用 `execute`

```java
fixedPool.execute(() -> {
    System.out.println("執行 Runnable 任務");
});
```

### 4.2 使用 `submit`（可取得回傳值或檢查異常）

```java
Future<Integer> future = fixedPool.submit(() -> {
    // Callable 任務可回傳結果
    return 42;
});

// 取得結果（會阻塞直到計算完成）
int result = future.get();
System.out.println("計算結果：" + result);
```

---

## 5. 排程任務範例

```java
// 延遲 1 秒後執行一次
scheduler.schedule(
    () -> System.out.println("1 秒後執行"),
    1, TimeUnit.SECONDS
);

// 每 2 秒執行一次（上次開始後計時）
scheduler.scheduleAtFixedRate(
    () -> System.out.println("週期性執行"),
    0, 2, TimeUnit.SECONDS
);
```

---

## 6. 關閉與回收

使用執行完畢後，務必呼叫：

```java
fixedPool.shutdown();               // 停止接收新任務，等待既有任務執行完畢
if (!fixedPool.awaitTermination(60, TimeUnit.SECONDS)) {
    fixedPool.shutdownNow();        // 強制中斷所有執行緒
}
```

同理可用於 `ScheduledExecutorService`。

---

## 7. 常見注意事項

1. **避免無限增長的任務佇列**：固定池搭配無界佇列可能會 OOM，可考慮使用有界 `BlockingQueue` 並自訂 `ThreadPoolExecutor`。
    
2. **正確處理例外**：任務內若發生未捕捉的 RuntimeException，執行緒可能終止。可在提交前以 `try/catch` 包裝，或實作自訂的 `ThreadFactory`／`RejectedExecutionHandler`。
    
3. **適時調整池大小**：根據 CPU 核心數、I/O 密集度選擇合適的執行緒數，避免過度切換或閒置。
    

---

## 8. 進階擴充：自訂 ThreadPoolExecutor

若內建工廠方法不夠彈性，可自己 new：

```java
ThreadPoolExecutor customPool = new ThreadPoolExecutor(
    2,                  // corePoolSize
    10,                 // maximumPoolSize
    60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(50),               // 任務佇列 (有界)
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.CallerRunsPolicy()   // 拒絕策略
);
```

---

透過 Executor 框架，你可以更乾淨、高效、安全地管理併發任務，並靈活擴充各種排程與執行策略。若有進一步需求（如 Fork/Join、Work Stealing），也可無縫整合相對應的執行器。