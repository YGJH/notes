在 Rust 中進行多執行緒（multithreading）開發是非常安全而且強大的，主要是因為它的 **所有權系統（ownership system）** 能在編譯期幫你防止資料競爭（data race）等問題。以下是非常詳細的介紹：

---

## 🧵 一、建立新執行緒：`std::thread`

Rust 標準函式庫提供了 `std::thread` 模組來建立和管理執行緒。

### 🔹 基本範例：

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..=5 {
            println!("子執行緒：{}", i);
        }
    });

    for i in 1..=5 {
        println!("主執行緒：{}", i);
    }

    // 等待子執行緒執行完畢
    handle.join().unwrap();
}
```

### 說明：

- `thread::spawn(|| {...})`：建立一個新的執行緒。
    
- 回傳的 `handle` 是一個 `JoinHandle`，代表這個執行緒。
    
- `join()`：讓主執行緒等待子執行緒結束。
    

---

## 📦 二、傳遞資料給執行緒

Rust 為了避免資料競爭，**預設不允許把變數借到執行緒中除非滿足 `Send` trait**。如果你要把變數「擁有權」移進執行緒，請使用 `move` 關鍵字。

### 🔹 傳值給執行緒：

```rust
fn main() {
    let v = vec![1, 2, 3];

    let handle = std::thread::spawn(move || {
        println!("向量內容：{:?}", v);
    });

    handle.join().unwrap();
}
```

---

## 🔄 三、共享資料：使用 `Arc` 與 `Mutex`

如果你要在多個執行緒中共享 **可變資料**，Rust 不允許你直接這樣做，除非你用：

- `Arc<T>`：原子參考計數（Atomic Reference Counted）
    
- `Mutex<T>`：互斥鎖，保護資料的同步性
    

### 🔹 多執行緒共享與修改變數：

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter); // clone Arc 的參考計數
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap(); // 加鎖
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("結果: {}", *counter.lock().unwrap());
}
```

### 📌 說明：

- `Arc::clone()`：不是深拷貝，而是增加參考計數。
    
- `Mutex::lock()`：加鎖，回傳的是一個 `MutexGuard`，自動在 scope 結束時解鎖。
    

---

## 🕵️ 四、避免死鎖與資料競爭

Rust 利用型別系統與所有權模型在 **編譯期** 防止：

- 多個執行緒同時修改資料（data race）
    
- 無限制的共享可變引用（mutable aliasing）
    

但你仍然可能會：

- **邏輯錯誤造成死鎖**（如兩個執行緒交互等待資源）
    

---

## 🧰 五、更多進階工具（必要時使用）

- `RwLock<T>`：可同時多個讀鎖或一個寫鎖。
    
- `Condvar`：條件變數，讓執行緒等待事件發生。
    
- `mpsc`（多對一通道）：執行緒間通訊用。
    

### 範例：執行緒之間傳送訊息

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let msg = String::from("Hello from thread");
        tx.send(msg).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("收到訊息: {}", received);
}
```

---

## 🧭 小結

|概念|用法|
|---|---|
|建立執行緒|`thread::spawn(|
|等待執行緒完成|`handle.join()`|
|傳遞資料進執行緒|使用 `move`|
|多執行緒共享資料|`Arc<Mutex<T>>`|
|執行緒通訊|`mpsc::channel()`|

---

如果你有具體想實作的情境，例如**網頁伺服器、多工下載、模擬任務池、GPU資料處理排程**，可以提出來，我可以幫你做進一步設計與範例撰寫。


---

# ✅ 目標：

1. **模擬任務池（Thread Pool）**：
    
    - 用多個執行緒接收並執行任務（模擬工作分派機制）。
        
    - 使用 `channel` 傳送任務給工作執行緒。
        
2. **GPU 資料處理排程**（簡化版）：
    
    - 模擬 GPU 計算：假設我們有數個資料區塊需要送進 GPU 執行。
        
    - 每個任務代表一個資料區塊運算（假裝跑 GPU kernel）。
        

---

# 🧵 一、Thread Pool 任務池範例

> 這裡我們會自己實作一個簡單的 thread pool，不使用第三方 crate。

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

type Job = Box<dyn FnOnce() + Send + 'static>;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    pub fn new(size: usize) -> Self {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel::<Job>();
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, job: F)
    where
        F: FnOnce() + Send + 'static,
    {
        self.sender.send(Box::new(job)).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Self {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv();
            match job {
                Ok(job) => {
                    println!("Worker {id} 執行任務中...");
                    job();
                }
                Err(_) => {
                    println!("Worker {id} 接收任務失敗，終止。");
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

### 🧪 測試：

```rust
fn main() {
    let pool = ThreadPool::new(4);

    for i in 0..10 {
        pool.execute(move || {
            println!("模擬處理任務 {}", i);
            std::thread::sleep(std::time::Duration::from_millis(500));
        });
    }

    std::thread::sleep(std::time::Duration::from_secs(5)); // 等任務跑完
}
```

---

# ⚙️ 二、模擬 GPU 任務排程

GPU 計算通常是把「資料批次」送進「排程器」，這裡我們模擬資料塊送進 thread pool 模擬排程器，並且假裝每次呼叫 GPU kernel。

### ✅ 資料區塊結構與任務分派：

```rust
use std::sync::{Arc, Mutex};
use rand::Rng;

#[derive(Debug, Clone)]
struct DataBlock {
    id: usize,
    values: Vec<f32>,
}

fn mock_gpu_kernel(block: DataBlock) {
    println!("🚀 [GPU] 開始處理 DataBlock #{}", block.id);
    let result: f32 = block.values.iter().map(|x| x * x).sum(); // 假裝運算
    std::thread::sleep(std::time::Duration::from_millis(300)); // 模擬 GPU 執行時間
    println!("✅ [GPU] DataBlock #{} 完成，結果: {:.2}", block.id, result);
}

fn main() {
    let pool = ThreadPool::new(4);

    let mut blocks = vec![];
    for i in 0..8 {
        let mut rng = rand::thread_rng();
        let values: Vec<f32> = (0..1000).map(|_| rng.gen_range(0.0..10.0)).collect();
        blocks.push(DataBlock { id: i, values });
    }

    for block in blocks {
        pool.execute(move || {
            mock_gpu_kernel(block);
        });
    }

    std::thread::sleep(std::time::Duration::from_secs(5));
}
```

---

# 📌 延伸：真的用 GPU 做的話？

如果你想在 Rust 中用 GPU，以下幾個 Crates 可以幫你：

|Crate|功能|
|---|---|
|[`wgpu`](https://github.com/gfx-rs/wgpu)|跨平台 GPU 運算，支援 Vulkan/Metal/DX12/WebGPU|
|[`cust`](https://docs.rs/cust)|CUDA binding for Rust（可跑 CUDA kernel）|
|[`opencl3`](https://crates.io/crates/opencl3)|使用 OpenCL 執行 GPU 任務|
|[`rust-gpu`](https://github.com/EmbarkStudios/rust-gpu)|把 Rust 編譯成 SPIR-V shader，用在 GPU 上執行|

如果你對 **實際把 Rust 和 GPU 整合（CUDA、OpenCL、WebGPU）** 有興趣，我可以帶你一步一步來建立完整範例。

---

要不要我接著示範用 `cust` 實際在 NVIDIA GPU 上跑一個簡單的平方總和？還是你想先練習任務池架構？