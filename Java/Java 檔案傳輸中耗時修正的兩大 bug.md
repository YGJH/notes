以下是這次 Java 檔案傳輸中耗時修正的兩大 bug，以及對應的解法：

---

## 一、問題回顧

1. **握手與 ACK 分離在不同 Socket**
    
    - 在建立連線時，你先用「Socket A」做 SYN/ACK 的握手（handshake）。
        
    - 卻又另外新開一個「Socket B」去呼叫 `receiveACK(socketB)` 等待 ACK。
        
    - 由於 TCP 的 ACK 只會回傳到握手時使用的那條連線（Socket A），在 Socket B 上永遠不會收到任何資料，因此「拿不到 ACK」的錯誤並非 Windows 特有的怪異行為，而是程式層面的協議實作錯誤。
        
2. **`receiveACK(...)` 方法過早關閉連線**
    
    - 你的 `receiveACK` 實作使用了 try-with-resources 包裝 `InputStream`：
        
        ```java
        try (DataInputStream dis = new DataInputStream(socket.getInputStream())) {
            // ...
        }
        ```
        
    - 這一寫法在讀完一次後就自動 close 底層的 Socket，導致後續想重複接收或傳輸資料時連線已經中斷。
        

---

## 二、根本原因

- **使用了兩條獨立的 TCP 連線**（Socket A vs. Socket B），導致「應該在同一條通道回應的 ACK」永遠跑不到你讀的那條通道上。
    
- **過早關閉 Socket**，程式假設每次呼叫 `receiveACK` 都是一次性操作，卻忽略了還會接著做資料傳輸。
    

---

## 三、解決方案

1. **統一欄位：同一條 Socket**
    
    - 不要再在握手、ACK、資料傳輸之間切換不同的 Socket。
        
    - 全部都透過同一個 `Socket` 物件的 `getInputStream()` / `getOutputStream()` 完成。
        
2. **不在接收 ACK 時自動關閉**
    
    - 將 `receiveACK(...)` 改成不使用 try-with-resources，改由呼叫端統一在「傳輸完畢後」才關閉 socket。
        
    - 如果要多次呼叫 `receiveACK`，直接重複使用同一組 I/O 流即可。
        

---

## 四、修正示意（伪码）

```java
// 建立唯一的連線
Socket socket = new Socket(serverHost, serverPort);
DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
DataInputStream dis  = new DataInputStream(socket.getInputStream());

// 1. 握手
dos.writeUTF("SYN");
String ack1 = dis.readUTF();   // 在同一條 socket 上收到 ACK

// 2. 資料傳輸
dos.writeInt(dataLength);
dos.write(dataBytes);

// 3. 接收最終確認
String finalAck = dis.readUTF();

// 4. 傳輸結束後再關閉
dos.close();
dis.close();
socket.close();
```

透過上述修正，就能確保握手、ACK、資料都走同一條 TCP 連線，並且不會在中途意外關閉流或連線。