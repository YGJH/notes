要在 Rust 中做到「不同 Hz 頻率播放」，基本上就是寫一個音訊串流（audio stream），不斷地產生對應頻率的波形，然後送到揚聲器去。下面示範兩種常見做法：一種用低階的 `cpal`（Cross-Platform Audio Library）直接操作樣本；另一種用更高階的 `rodio` 來快速上手。一路上我會穿插小幽默，保證你聽著聽著不會當機😉

---

## 1. 用 cpal 做一個簡單的正弦波播放器

### 步驟一：在 `Cargo.toml` 加依賴

```toml
[dependencies]
cpal = "0.13"       # 音訊 I/O
anyhow = "1.0"      # 簡化錯誤處理
```

### 步驟二：程式碼範例

```rust
use std::f32::consts::PI;
use std::time::Duration;
use anyhow::Result;
use cpal::traits::{DeviceTrait, HostTrait, StreamTrait};

fn main() -> Result<()> {
    // 1. 拿到預設 Host 跟預設輸出裝置
    let host = cpal::default_host();
    let device = host.default_output_device()
        .expect("找不到預設輸出設備，請確認聲音裝置");

    // 2. 讀取裝置支援的預設設定（SampleRate、Channels）
    let config = device.default_output_config()?;
    let sample_rate = config.sample_rate().0 as f32;
    let channels = config.channels() as usize;

    // 3. 你想播幾 Hz？可以改成讀參數或互動輸入
    let freq_hz = 440.0; // A4 音高，測試用

    // 4. 產生正弦波產生器：sample_clock 控制相位
    let mut sample_clock = 0f32;
    let mut next_sample = move || {
        // 0..sample_rate-1 走一圈就完成一個週期
        let value = (sample_clock * 2.0 * PI * freq_hz / sample_rate).sin();
        sample_clock = (sample_clock + 1.0) % sample_rate;
        value
    };

    // 5. 根據 SampleFormat 建串流（只示範 f32；i16/u16 同理）
    let stream = device.build_output_stream(
        &config.into(),
        move |data: &mut [f32], _| write_data(data, channels, &mut next_sample),
        move |err| eprintln!("流發生錯誤: {}", err),
    )?;

    // 6. 播起來，維持一段時間後結束
    stream.play()?;
    println!("正在播放 {:.1} Hz 的正弦波，3 秒後自動停止…", freq_hz);
    std::thread::sleep(Duration::from_secs(3));
    Ok(())
}

// 把產生的 sample 塞到每個 channel
fn write_data(output: &mut [f32], channels: usize, generator: &mut dyn FnMut() -> f32) {
    for frame in output.chunks_mut(channels) {
        let sample = generator();
        for slot in frame.iter_mut() {
            *slot = sample;
        }
    }
}
```

**重點解說：**

- `sample_rate`：每秒要送多少樣本到聲卡，絕大多數是 44_100 或 48_000 Hz。
    
- `sample_clock`：用來計算到目前的位置在一個週期上的哪裡。
    
- 每呼叫一次 `next_sample()`，就前進一個樣本的位置，算出對應的正弦值。
    
- 頻率改成 `880.0`，就聽到 A5；改成 `261.6`，聽 C4，就像給電腦換了調音師🔧。
    

---

## 2. 用 rodio 更快速地播音

如果你只是想「丟個頻率進去，快播出來」而不想管太多底層細節，`rodio` 會更簡單：

### Cargo.toml

```toml
[dependencies]
rodio = "0.14"
```

### 程式碼

```rust
use std::time::Duration;
use rodio::{OutputStream, Sink, source::SineWave};

fn main() {
    // 預設串流和 handle
    let (_stream, stream_handle) = OutputStream::try_default()
        .expect("無法開啟輸出串流");
    let sink = Sink::try_new(&stream_handle)
        .expect("無法建立 Sink");

    // 直接產生一個指定頻率的 SineWave source
    let freq_hz = 523; // C5
    let source = SineWave::new(freq_hz);
    sink.append(source);

    println!("播放 {} Hz …", freq_hz);
    sink.sleep_until_end();  // 等聲音播完才結束程式
}
```

這樣大概 10 行就能播一個 tone，不過靈活性不如 `cpal`。未來如果要做更完整的合成器、加 ADSR 包絡線、濾波、混音，就可以往 `cpal` + `dasp`（Digital Audio Signal Processing）方向去進階。

---

## 3. 拓展思考（前瞻觀點）

1. **多音頻 & 并行播放**：同時產生多個 oscillator，做和弦或複雜音色。
    
2. **波形種類**：除了正弦波，還能做方波、三角波、噪聲等，用 `match` 或者動態 `Box<dyn Iterator<Item=f32>>` 切換。
    
3. **調變**：AM、FM 合成；比如用一個 LFO（低頻振盪器）去 modulate 另一個 oscillator。
    
4. **DSP 生態系**：可以結合 [`dasp`](https://crates.io/crates/dasp)、[`fundsp`](https://crates.io/crates/fundsp) 等專門做音訊處理的庫。
    
5. **GUI 控制**：未來可加上 GUI，用 egui 或 iced 實時調頻率、波形等參數，邁向 DIY 軟體 Synth！
    

---

以上就是用 Rust 播「Hz 撥放器」的入門示範，希望能讓你快速起步、彈奏出屬於自己的電子音樂。下次我們可以聊聊如何把這個架構延伸成簡易 DAW（Digital Audio Workstation）──保證比前端框架還好玩！