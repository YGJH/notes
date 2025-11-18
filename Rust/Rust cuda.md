在 Rust 中撰寫 CUDA 程式並不如 C++ 那樣直接，但你可以透過下列兩種方式實現 Rust + CUDA：

---

## 🧩 一、使用 [`rustacuda`](https://github.com/bheisler/RustaCUDA)

這是目前 **最主流、穩定的 Rust CUDA FFI 綁定庫**。它可以：

- 呼叫 `.ptx` 檔案中編譯好的 CUDA kernel。
    
- 操作 device memory、launch kernel。
    
- 與 `nvcc` 編譯的 PTX 串接。
    

---

## 🛠️ 二、建立一個 Rust + CUDA 的基本架構

### 1️⃣ 安裝必要工具

- 安裝 CUDA Toolkit（NVIDIA 官方）
    
- Rust toolchain
    
- `rustacuda` 套件
    

```bash
cargo new rust_cuda_demo
cd rust_cuda_demo
cargo add rustacuda rustacuda_core rustacuda_derive
```

---

### 2️⃣ 建立 CUDA kernel（`.cu`）

新建 `kernel.cu`：

```cpp
extern "C" __global__ void add_kernel(float* x, float* y, float* out, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n) {
        out[i] = x[i] + y[i];
    }
}
```

編譯成 `.ptx`：

```bash
nvcc -ptx kernel.cu -o kernel.ptx
```

---

### 3️⃣ Rust 主程式（`main.rs`）

```rust
use rustacuda::prelude::*;
use rustacuda::memory::DeviceBuffer;
use std::error::Error;
use std::fs;

fn main() -> Result<(), Box<dyn Error>> {
    rustacuda::init(CudaFlags::empty())?;

    // 選擇第一張 GPU
    let device = Device::get_device(0)?;
    let ctx = Context::create_and_push(ContextFlags::MAP_HOST | ContextFlags::SCHED_AUTO, device)?;

    let ptx = fs::read_to_string("kernel.ptx")?;
    let module = Module::load_from_string(&ptx)?;
    let stream = Stream::new(StreamFlags::DEFAULT, None)?;

    let n = 1024;
    let mut host_x = vec![1.0f32; n];
    let mut host_y = vec![2.0f32; n];
    let mut host_out = vec![0.0f32; n];

    let mut dev_x = DeviceBuffer::from_slice(&host_x)?;
    let mut dev_y = DeviceBuffer::from_slice(&host_y)?;
    let mut dev_out = DeviceBuffer::from_slice(&host_out)?;

    // Kernel launch
    unsafe {
        let func = module.get_function("add_kernel")?;
        let args = &[
            &mut dev_x.as_device_ptr(),
            &mut dev_y.as_device_ptr(),
            &mut dev_out.as_device_ptr(),
            &n,
        ];
        let grid_size = (n as u32 + 255) / 256;
        let block_size = 256;
        launch!(func<<<grid_size, block_size, 0, stream>>>(
            dev_x.as_device_ptr(),
            dev_y.as_device_ptr(),
            dev_out.as_device_ptr(),
            n
        ))?;
    }

    stream.synchronize()?;
    dev_out.copy_to(&mut host_out)?;

    println!("Result: {:?}", &host_out[..10]);
    Ok(())
}
```

---

### ✅ 重點解釋

|部分|說明|
|---|---|
|`nvcc -ptx`|將 CUDA kernel 編譯為 Rust 可讀的中介格式|
|`Module::load_from_string(&ptx)`|在 Rust 中載入 kernel|
|`launch!` 宏|用來啟動 GPU kernel|
|`DeviceBuffer`|管理 CUDA GPU 記憶體|

---

## 🔥 三、其他選擇：Rust-GPU / Accel / cust

|專案|說明|
|---|---|
|[`rust-gpu`](https://github.com/EmbarkStudios/rust-gpu)|寫 GPU kernel in Rust，目標是取代 CUDA 編譯器（仍實驗中）|
|[`cust`](https://github.com/denzp/rust-cuda)|另一個 CUDA FFI 綁定，比 `rustacuda` 更底層|
|[`accel`](https://github.com/termoshtt/accel)|更易用的 API 封裝，適合簡單應用，但不如 rustacuda 靈活|

---

## 🔬 若你想做高精度 π on GPU 的方法建議

因為 MPFR 無法直接上 GPU，你可以這樣做：

### 混合策略（推薦）：

1. **用 GPU** 快速計算 double 精度的級數項（如 Chudnovsky 中的大量乘除）。
    
2. **回傳 CPU**，再用 MPFR 精修誤差、提升精度。
    
3. （進階）實作多位數模擬浮點數（如 float128 emulator）直接在 GPU。
    

---

## 📦 補充：如何整合 CUDA 編譯流程

你可以：

- 寫一個 `build.rs` 自動呼叫 `nvcc`：
    

```rust
use std::process::Command;
fn main() {
    Command::new("nvcc")
        .args(&["-ptx", "kernel.cu", "-o", "kernel.ptx"])
        .status()
        .expect("failed to compile CUDA kernel");
}
```

---

是否需要我幫你做一個完整的 **Rust + CUDA + 高精度 π 混合加速專案結構**？我可以提供一份範例倉庫結構與細部教學。