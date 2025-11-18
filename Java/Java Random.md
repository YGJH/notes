以下從多個面向、包含使用方式、底層機制、不同實作與最佳實踐，詳細介紹 Java 中的隨機數產生（Random）相關 API。

---

## 1. `java.util.Random` 類別

### 1.1 基本概念

- **演算法**：預設採用線性同餘產生器（Linear Congruential Generator, LCG），公式為
    
    seedn+1=(seedn×25214903917+11)    248 seed_{n+1} = (seed_n \times 25214903917 + 11)\,\bmod\,2^{48}
- **可重現性**：輸入相同的種子（seed）後，每次產生的序列都相同，方便在測試或模擬時重複實驗。
    

### 1.2 常用建構子與方法

```java
// 預設以 System.currentTimeMillis() 為種子
Random rnd = new Random();

// 自訂種子，便於可重現
Random rndWithSeed = new Random(123456L);

// 產生一個 [0, bound) 之間的整數
int n = rnd.nextInt(100);

// 產生一個非負整數，範圍 0 ~ Integer.MAX_VALUE
int nonNeg = rnd.nextInt();

// 產生 [0.0, 1.0) 之間的浮點數
double d = rnd.nextDouble();

// 產生隨機布林值
boolean b = rnd.nextBoolean();

// 填滿 byte 陣列
byte[] buf = new byte[16];
rnd.nextBytes(buf);
```

|方法|說明|
|---|---|
|`nextInt()`|0 ~ 231−12^{31}-1 之間的整數|
|`nextInt(bound)`|0 ~ bound-1 之間的整數|
|`nextLong()`|0 ~ 263−12^{63}-1 之間的長整數|
|`nextFloat()`|0.0f ~ 1.0f|
|`nextDouble()`|0.0 ~ 1.0|
|`nextBoolean()`|`true` or `false`|
|`nextBytes()`|將隨機值填入使用者傳入的 `byte[]`|

---

## 2. 種子（Seed）與可重複性

- **自訂種子**
    
    ```java
    Random rndA = new Random(42L);
    Random rndB = new Random(42L);
    // rndA.nextInt() 與 rndB.nextInt() 會輸出相同的值
    ```
    
- **重複模擬**  
    在科學或測試模擬場景，使用固定種子能保證「每次跑出來的隨機行為都一樣」，便於結果比對與重現。
    

---

## 3. 線程安全性與效能

- `java.util.Random` **不是** 執行緒安全的 (thread-safe)，其多線程競爭時會在內部使用 `AtomicLong` 或 `synchronized`，導致嚴重的 **效能瓶頸**。
    
- **推薦**
    
    - 多線程場景下，使用 `java.util.concurrent.ThreadLocalRandom`
        
    - 或 Java 8 以後的 `java.util.SplittableRandom`
        

---

## 4. `ThreadLocalRandom`

- **簡介**：每個執行緒維護自己的一份隨機狀態，消除鎖競爭
    
- **用法**
    
    ```java
    import java.util.concurrent.ThreadLocalRandom;
    
    // 產生 [0, 100) 的整數
    int x = ThreadLocalRandom.current().nextInt(100);
    
    // 產生 [50, 150) 的整數
    int y = ThreadLocalRandom.current().nextInt(50, 150);
    
    double z = ThreadLocalRandom.current().nextDouble();
    ```
    
- **優勢**：多線程高併發下效能大幅優於 `Random`
    

---

## 5. `SplittableRandom`

- **Java 8 新增**，以更優秀的分割（split）演算法支援並行流 (`Stream`)
    
- **特性**：
    
    - 能夠快速「分割」出新的隨機來源，適合在 parallel stream 中傳遞
        
    - 演算法質量（統計特性）優於 `Random`
        
- **範例**
    
    ```java
    SplittableRandom spr = new SplittableRandom(123L);
    spr.ints(5, 0, 10)          // 產生 5 個 [0,10) 的整數 stream
       .forEach(System.out::println);
    ```
    

---

## 6. 加密安全：`SecureRandom`

- **用途**：產生強隨機數（CSPRNG），用於密碼學金鑰、Token 生成等安全關鍵場景
    
- **實作**：可選不同演算法（如 SHA1PRNG、NativePRNG 等），由安全提供者（Provider）注入
    
- **使用**
    
    ```java
    SecureRandom sr = new SecureRandom();
    byte[] seed = sr.generateSeed(16);    // 產生真隨機種子
    sr.setSeed(seed);
    
    byte[] token = new byte[32];
    sr.nextBytes(token);                  // 安全隨機
    ```
    
- **注意**：比一般 `Random` 慢很多，僅在安全性要求高時使用
    

---

## 7. Java 8 之 Stream API 結合

```java
// 產生無限隨機整數流，過濾出偶數後取前 10 個
Random rnd = new Random();
List<Integer> evens = rnd.ints()           
    .filter(i -> i % 2 == 0)
    .limit(10)
    .boxed()
    .collect(Collectors.toList());
```

- `ints()`, `longs()`, `doubles()` 都可直接拿到 `IntStream`、`LongStream`、`DoubleStream`
    

---

## 8. 常見 Pitfall 與建議

|問題／誤用|建議|
|---|---|
|在多線程共用同一個 `Random` 實例|改用 `ThreadLocalRandom` 或 手動鎖定|
|以時間當種子，每次快速重複 new Random|共用一個實例，避免相同的種子導致序列相同|
|需要密碼學安全的場景卻用 `Random`|改用 `SecureRandom`|
|需要平行處理、分割隨機來源|考慮 `SplittableRandom`|

---

### 小結

1. **日常非安全性需求** → 使用 `java.util.Random`，或更佳的 `ThreadLocalRandom`
    
2. **多執行緒高效能** → `ThreadLocalRandom` 最佳
    
3. **密碼學安全** → `SecureRandom`
    
4. **平行流、分割隨機性** → `SplittableRandom`
    

透過以上不同實作，Java 提供了從簡易到高階、從效能到安全的完整隨機數生成方案；根據場景選擇最合適的 API，才能在效能與正確性間取得平衡。