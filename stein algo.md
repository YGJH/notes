
```rust

fn binary_gcd(mut a: u64, mut b: u64) -> u64 {
    if a == 0 { return b; }
    if b == 0 { return a; }

    // 步驟 1: 找出 a 和 b 共同包含 2 的次冪 (g)
    // trailing_zeros() 在底層通常對應一條 CPU 指令，極快
    let g = (a | b).trailing_zeros();

    // 步驟 2: 將 a 和 b 各自的 2 因數全部除掉，使其變為奇數
    a >>= a.trailing_zeros();

    loop {
        b >>= b.trailing_zeros();

        // 現在 a, b 都是奇數，進行相減（步驟 3）
        if a > b {
            std::mem::swap(&mut a, &mut b);
        }
        b -= a;

        if b == 0 { break; }
    }

    // 步驟 5: 補回最初拿掉的 2 的次冪
    a << g
}
```
