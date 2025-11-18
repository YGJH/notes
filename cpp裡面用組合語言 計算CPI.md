```cpp
#include <iostream>
#include <vector>
#include <cstdint> // For uint64_t

// --- 第 2 步：測量工具 (ARM64 Cycle Counter) ---
// 讀取 ARM64 CPU 的「虛擬週期計數器」暫存器 (CNTVCT_EL0)
// (在 M1/M2/M3 上，你需要讀取 PMCCNTR_EL0，這需要更複雜的設定)
// 為了簡化，我們先用 CNTPCT_EL0
//static inline uint64_t read_cycle_counter() {
//    uint64_t val;
    // 'mrs' = Move from System Register
    // 將 CNTPCT_EL0 暫存器的值讀取到 val (%0)
//    asm volatile("mrs %0, CNTPCT_EL0" : "=r"(val));
//    return val;
//}
inline uint64_t rdtsc() {
    uint32_t lo, hi;
    __asm__ __volatile__ ("rdtsc" : "=a" (lo), "=d" (hi));
    return ((uint64_t)hi << 32) | lo;
}

// --- 第 1 步：測試酬載 (The Payload) ---
// 這是我們要執行的 N 條相依指令
// 我們使用 C++ 的模板和宏來「自動展開」迴圈
// 這能騙過編譯器，讓它產生 N 條真實的指令

// 這是一個模板，它會遞迴地產生組合語言
template<int N>
struct DependentAdds {
    static void run() {
        // 執行 1 次 add
        __asm__ __volatile__("addl %%ebx, %%eax" ::: "eax", "ebx");
        // 遞迴呼叫，執行剩下的 N-1 次
        DependentAdds<N - 1>::run();
    }
};

// 遞迴的終點
template<>
struct DependentAdds<0> {
    static void run() {
        // N=0，什麼都不做
    }
};


int main() {
    // --- 第 3 步：實驗迴圈 (The Experiment) ---
    // 我們要測試的 N 值
    // 實際上你會用一個 for 迴圈來跑 (N=100; N<=1000; N+=10)
    // 這裡我們只示範一個 N = 600
    
    const int N_TO_TEST = 600; 

    uint64_t start_cycles, end_cycles;

    // 準備：在暫存器中放入一些初始值
    __asm__ __volatile__ (
        "movl $1, %%eax\n\t"  // eax = 1
        "movl $2, %%ebx\n\t"  // ebx = 2 (我們要加上的數字)
        : // no outputs
        : // no inputs
        : "eax", "ebx" // 告訴編譯器我們弄亂了 eax 和 ebx
    );

    // --- 開始測量 ---
    start_cycles = rdtsc();

    // 執行 N=600 次的 add x0, x0, x1
    DependentAdds<N_TO_TEST>::run();

    end_cycles = rdtsc();
    // --- 測量結束 ---

    uint64_t total_cycles = end_cycles - start_cycles;
    
    // 為了防止編譯器把上面所有東西都優化掉
    // 我們「假裝」需要 x0 的最終結果
    uint64_t final_result;
    __asm__ __volatile__ ("movq %%rax, %0" : "=r"(final_result));

    std::cout << "--- N = " << N_TO_TEST << " ---" << std::endl;
    std::cout << "Total Cycles: " << total_cycles << std::endl;
    std::cout << "Cycles per Instruction: " << (double)total_cycles / N_TO_TEST << std::endl;
    std::cout << "Final register value (to prevent optimization): " << final_result << std::endl;

    return 0;
}

```