以下內容將從概念、API 方法、範例與進階技巧四個面向，詳細介紹 Java 中的 `Socket`（客戶端）與 `ServerSocket`（伺服器端）用法。

---

## 一、基本概念

- **Socket（插座）**
    
    - 用於在客戶端與伺服器之間建立 TCP 連線的端點。
        
    - 每個 `Socket` 物件代表一條雙向通道，可透過 `InputStream`／`OutputStream` 進行讀寫。
        
- **ServerSocket（伺服器插座）**
    
    - 監聽指定埠號（port）上的連線請求。
        
    - 呼叫 `accept()` 時，若有客戶端連入則回傳對應的 `Socket`，並繼續監聽下一個請求。
        
- **TCP 三次握手**
    
    1. 客戶端發送 SYN
        
    2. 伺服器回應 SYN-ACK
        
    3. 客戶端回應 ACK  
        完成後即可開始資料傳輸。
        

---

## 二、`Socket` 類別詳解

```java
public class Socket implements Closeable {
    // 最常用的建構子
    public Socket(String host, int port) throws IOException { … }

    // 取得輸入／輸出串流
    public InputStream getInputStream() throws IOException { … }
    public OutputStream getOutputStream() throws IOException { … }

    // 設定連線超時（毫秒）
    public void setSoTimeout(int timeout) throws SocketException { … }

    // 關閉連線
    public void close() throws IOException { … }
}
```

1. **建立連線**
    
    ```java
    Socket socket = new Socket("192.168.0.100", 8888);
    ```
    
    若連線失敗（伺服器無回應或拒絕），會拋出 `IOException`。
    
2. **資料讀寫**
    
    ```java
    // 寫入
    OutputStream out = socket.getOutputStream();
    out.write("Hello, Server".getBytes(StandardCharsets.UTF_8));
    
    // 讀取
    InputStream in = socket.getInputStream();
    byte[] buf = new byte[1024];
    int len = in.read(buf);
    String resp = new String(buf, 0, len, StandardCharsets.UTF_8);
    ```
    
3. **例外處理與關閉**
    
    ```java
    try (Socket socket = new Socket(host, port)) {
        // ... 讀寫操作 ...
    } catch (IOException e) {
        e.printStackTrace();
    }
    // try-with-resources 會自動呼叫 close()
    ```
    

---

## 三、`ServerSocket` 類別詳解

```java
public class ServerSocket implements Closeable {
    // 建構：port、backlog（等待佇列長度）、綁定的本機位址
    public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException { … }

    // 監聽並接受連線，阻塞呼叫
    public Socket accept() throws IOException { … }

    // 設定 accept() 超時
    public void setSoTimeout(int timeout) throws SocketException { … }

    public void close() throws IOException { … }
}
```

1. **啟動伺服器**
    
    ```java
    ServerSocket server = new ServerSocket(8888);
    System.out.println("伺服器啟動，監聽 port 8888 …");
    ```
    
2. **接受連線**
    
    ```java
    while (true) {
        Socket client = server.accept();  // 阻塞，直到有新的連線
        System.out.println("客戶端連入：" + client.getRemoteSocketAddress());
        // 可交由新執行緒處理 client
    }
    ```
    
3. **關閉伺服器**
    
    ```java
    server.close();
    ```
    

---

## 四、範例：簡易 Echo Server 與 Client

### 伺服器端 (`EchoServer.java`)

```java
import java.io.*;
import java.net.*;

public class EchoServer {
    public static void main(String[] args) {
        int port = 8888;
        try (ServerSocket server = new ServerSocket(port)) {
            System.out.println("Echo 伺服器啟動於 port " + port);
            while (true) {
                Socket client = server.accept();
                // 每個 client 用執行緒處理
                new Thread(() -> handle(client)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void handle(Socket socket) {
        try (socket;
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {
            String line;
            while ((line = in.readLine()) != null) {
                System.out.println("收到：" + line);
                out.println("Echo: " + line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 客戶端 (`EchoClient.java`)

```java
import java.io.*;
import java.net.*;

public class EchoClient {
    public static void main(String[] args) {
        String host = "localhost";
        int port = 8888;
        try (Socket socket = new Socket(host, port);
             BufferedReader console = new BufferedReader(new InputStreamReader(System.in));
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {
             
            String msg;
            while ((msg = console.readLine()) != null) {
                out.println(msg);
                System.out.println(in.readLine());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 五、進階技巧與注意事項

1. **多執行緒伺服器／連線池**
    
    - 若客戶端數量大，可考慮用 `ExecutorService` 管理工作執行緒，避免無限制 `new Thread()`。
        
2. **設定超時**
    
    - `server.setSoTimeout(ms)`：`accept()` 超時後拋 `SocketTimeoutException`，可用來優雅結束。
        
    - `socket.setSoTimeout(ms)`：`read()` 超時後拋 `SocketTimeoutException`，避免無限等待。
        
3. **Socket 選項（SocketOption）**
    
    - `setReuseAddress(true)`：允許重用先前連線的本地埠，方便伺服器重啟。
        
    - `setTcpNoDelay(true)`：關閉 Nagle 演算法，降低延遲。
        
    - `setKeepAlive(true)`：啟用 TCP Keep-Alive，檢測死掉的連線。
        
4. **資源釋放**
    
    - 一定要 `close()` `Socket` 與 `ServerSocket`，建議用 try-with-resources。
        
5. **錯誤處理**
    
    - 捕捉並記錄 IO 例外，避免伺服器因單一連線錯誤而當機。
        
    - 對拒絕服務（拒絕接受過多連線）可實作自訂 `RejectedExecutionHandler`（搭配執行緒池）。
        

---

透過上述介紹，你應該能理解並實作基本的 Java TCP 通訊客戶端與伺服器。後續可再加入加解密、壓縮、協議封包等功能，或轉向 NIO/Netty 等更高效的非封鎖 IO 解法。希望對你有幫助！