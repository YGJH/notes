
常見的「跨 process 同步 / IPC」手段與適用場景 (Linux/Unix)：

1) Pipe / 匿名管線  
- 單向位元串資料流 (父子或有共同祖先)  
- 同步靠阻塞：讀端 read 在無資料阻塞，寫端 write 在無讀者 SIGPIPE。  
示例：
````cpp
#include <unistd.h>
#include <sys/wait.h>
#include <cstdio>
int main() {
    int fd[2]; pipe(fd);
    if(fork()==0){
        close(fd[1]);
        char buf[32];
        int n=read(fd[0], buf, sizeof(buf));
        write(1, buf, n);
        return 0;
    } else {
        close(fd[0]);
        const char* msg="hello\n";
        write(fd[1], msg, 6);
        close(fd[1]);
        wait(NULL);
    }
}
````

2) FIFO (named pipe)  
- 透過檔案系統路徑 (mkfifo)，不需親緣關係。適合簡單串流。

3) 匿名共享記憶體 + mmap (父子)  
- 使用 mmap(MAP_SHARED|MAP_ANONYMOUS) 在 fork 前建立，共享同一頁。可存放旗標、計數。需顧 race → 搭配 futex(低階) 或 semaphore。  
示例：簡單計數 (用自旋 + volatile 不推薦，只示意)
````cpp
#include <sys/mman.h>
#include <unistd.h>
#include <sys/wait.h>
#include <cstdio>
#include <atomic>
int main(){
    void* p = mmap(NULL, 4096, PROT_READ|PROT_WRITE,
                   MAP_SHARED|MAP_ANONYMOUS, -1, 0);
    auto* cnt = new(p) std::atomic<int>(0); // 置於共享頁
    if(fork()==0){
        cnt->fetch_add(1, std::memory_order_relaxed);
        _exit(0);
    } else {
        cnt->fetch_add(1, std::memory_order_relaxed);
        wait(NULL);
        printf("cnt=%d\n", cnt->load());
    }
}
````

4) POSIX 共享記憶體 (shm_open)  
- 不限定親緣；類似建立一個命名的「檔案」再 mmap。  
````cpp
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <sys/wait.h>
#include <semaphore.h>
#include <cstdio>
struct Shared { int val; };
int main(){
    int fd = shm_open("/myshm", O_CREAT|O_RDWR, 0600);
    ftruncate(fd, sizeof(Shared));
    auto* s = (Shared*)mmap(NULL,sizeof(Shared),PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    s->val = 0;
    sem_t* sem = sem_open("/mysem", O_CREAT, 0600, 1); // binary semaphore
    if(fork()==0){
        sem_wait(sem);
        s->val += 10;
        sem_post(sem);
        _exit(0);
    } else {
        sem_wait(sem);
        s->val += 5;
        sem_post(sem);
        wait(NULL);
        printf("val=%d\n", s->val);
        sem_close(sem);
        sem_unlink("/mysem");
        shm_unlink("/myshm");
    }
}
````

5) POSIX 有名 semaphore (sem_open) / 無名 semaphore (sem_init with pshared=1)  
- 給共享記憶體中的临界區進入控制。  
````cpp
sem_t *sem = sem_open("/mysem", O_CREAT, 0600, 1);
sem_wait(sem);  // lock
// critical
sem_post(sem);  // unlock
````

6) SysV 共享記憶體 / semaphore / message queue  
- 舊 API (shmget, shmat, semget, msgsnd, msgrcv)。功能類似 POSIX 對應物。現代推薦 POSIX，但老程式大量使用。

7) POSIX Message Queue (mq_open)  
- 有優先級的訊息槽列 (固定大小訊息)，適合事件通知。  

8) Unix Domain Socket  
- 雙向、支援資料 + FD 傳遞 (sendmsg/SCM_RIGHTS)。適合複雜協定、非親緣。  
````cpp
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>
#include <cstdio>
int main(){
    int sv[2]; socketpair(AF_UNIX, SOCK_STREAM, 0, sv);
    if(fork()==0){
        close(sv[0]);
        const char* msg="ping";
        write(sv[1], msg, 4);
        _exit(0);
    } else {
        close(sv[1]);
        char buf[16];
        int n=read(sv[0], buf, sizeof(buf));
        write(1, buf, n);
    }
}
````

9) TCP/UDP Socket  
- 跨機器網路 IPC。延遲較高。

10) Signals  
- 極輕量通知 (SIGUSR1, SIGCHLD)。只傳少量資訊 (到 POSIX.1b real-time signals 可附帶值)。需在 handler 中只做 async-signal-safe 操作。

11) File lock (flock / fcntl)  
- 利用同一檔案鎖互斥。簡單但 I/O 系統呼叫成本高。適合低頻同步。

12) Eventfd / Timerfd / Signalfd (Linux 特有)  
- eventfd: 計數型同步原語 (配 epoll)。  
- 讓多 process 可以 atomic 增加/讀取計數並喚醒等待者。  

簡單 eventfd 例：
````cpp
#include <sys/eventfd.h>
#include <unistd.h>
#include <sys/wait.h>
#include <cstdio>
#include <cstdint>
int main(){
    int efd = eventfd(0, 0);
    if(fork()==0){
        uint64_t one=1;
        write(efd, &one, 8); // signal
        _exit(0);
    } else {
        uint64_t v;
        read(efd, &v, 8); // blocks until child writes
        printf("got=%llu\n",(unsigned long long)v);
        wait(NULL);
    }
}
````

13) Futex (低階)  
- Linux 原語；自己在共享記憶體維護狀態字，衝突時使用 syscall 降低核態切換。直接使用較繁瑣，通常用在自行實作高效鎖。多數情況用現成 semaphore/mutex 即可。

14) POSIX pthread process-shared synchronization  
- pthread_mutexattr_setpshared 讓 mutex 放在共享記憶體 (mmap) 中被多個 process 使用。適合需要與 thread 相似語意的鎖。  

範例：process-shared mutex
````cpp
#include <pthread.h>
#include <sys/mman.h>
#include <unistd.h>
#include <sys/wait.h>
#include <cstdio>
struct Shared {
    pthread_mutex_t m;
    int x;
};
int main(){
    auto* sh = (Shared*)mmap(NULL, sizeof(Shared), PROT_READ|PROT_WRITE,
                             MAP_SHARED|MAP_ANONYMOUS, -1, 0);
    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_setpshared(&attr, PTHREAD_PROCESS_SHARED);
    pthread_mutex_init(&sh->m, &attr);
    sh->x = 0;
    if(fork()==0){
        pthread_mutex_lock(&sh->m);
        sh->x += 7;
        pthread_mutex_unlock(&sh->m);
        _exit(0);
    } else {
        pthread_mutex_lock(&sh->m);
        sh->x += 5;
        pthread_mutex_unlock(&sh->m);
        wait(NULL);
        printf("x=%d\n", sh->x);
    }
}
````

選擇建議 (簡化)：
- 單向資料 + 親緣：pipe
- 雙向簡單傳訊 + 親緣：socketpair
- 多 process 共享狀態：mmap + (pthread mutex pshared 或 sem_t)
- 命名跨不相關 process：shm_open + named semaphore
- 事件喚醒：eventfd / pipe / mq
- 大量複雜協定：Unix domain socket

避免的常見錯誤：
- 以為 std::atomic 自動跨 process（要確保所在記憶體真共享；單純 fork 後兩份是分離的）  
- 在 signal handler 內呼叫非 async-signal-safe 函式 (如 printf)  
- 忘記關閉未使用的 pipe 端導致 read 永不 EOF  
- shm_open + sem_open 忘記 unlink → 資源殘留  

若你提供你想同步的「模型」（例如：多個子 process 累加計數、或是 work queue），可再給更精準實作。