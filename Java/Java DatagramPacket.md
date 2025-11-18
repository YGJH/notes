在 Java 中，`DatagramPacket` 是用來進行 **UDP (User Datagram Protocol)** 資料傳輸的核心類別之一。它隸屬於 `java.net` 套件，配合 `DatagramSocket` 使用，用來**封裝要傳送或接收的資料**。

---

### 🔹 `DatagramPacket` 是什麼？

`DatagramPacket` 是一個用來**封裝一個 UDP 封包**的類別，裡面包含了：

- 要傳送或接收的資料位元組陣列 (`byte[]`)
    
- 資料的長度
    
- 目的或來源的 IP 位址 (`InetAddress`)
    
- 目的或來源的埠號 (`port`)
    

---

### 🔸 常用建構子（Constructor）

#### 1. **接收用：**

```java
byte[] buffer = new byte[1024];
DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
```

- 這用來接收資料，`buffer` 是用來存資料的陣列，`buffer.length` 表示能接收的最大長度。
    

#### 2. **傳送用：**

```java
byte[] data = "Hello".getBytes();
InetAddress address = InetAddress.getByName("localhost");
int port = 9876;
DatagramPacket packet = new DatagramPacket(data, data.length, address, port);
```

- 這是用來傳送資料的建構方式。
    

---

### 🔸 搭配 `DatagramSocket` 使用

#### ✅ 傳送資料

```java
DatagramSocket socket = new DatagramSocket();
byte[] data = "Hello".getBytes();
InetAddress ip = InetAddress.getByName("127.0.0.1");
int port = 1234;

DatagramPacket packet = new DatagramPacket(data, data.length, ip, port);
socket.send(packet);
```

#### ✅ 接收資料

```java
DatagramSocket socket = new DatagramSocket(1234);
byte[] buffer = new byte[1024];
DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
socket.receive(packet);

String message = new String(packet.getData(), 0, packet.getLength());
System.out.println("Received: " + message);
```

---

### 🔸 常用方法

|方法|說明|
|---|---|
|`getData()`|取得封包的資料（byte[]）|
|`getLength()`|資料的實際長度|
|`getAddress()`|取得來源或目的 IP|
|`getPort()`|取得來源或目的 port|
|`setData(byte[] buf)`|設定要傳送或接收的資料陣列|
|`setAddress(InetAddress addr)`|設定目的 IP|
|`setPort(int port)`|設定目的 port|

---

### 🧠 小提醒

- UDP 是 **不保證可靠傳輸** 的，也就是說：
    
    - 不保證資料一定送到
        
    - 不保證送到的順序
        
    - 不保證資料不重複
        

所以如果你需要可靠性，應該用 TCP。

---

### ✏️ 結語

`DatagramPacket` 是 UDP 程式設計的資料容器，通常跟 `DatagramSocket` 一起使用，透過它你可以進行簡單的無連線資料傳輸。這在實作聊天程式、簡單通訊協定、遊戲中的即時位置同步等情境很常見。

---

如果你想看一個完整的 UDP 傳送/接收範例，我可以幫你寫出來！需要嗎？