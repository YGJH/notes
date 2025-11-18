下面分別介紹這些網路與加密協議／技術的原理與應用場景：

---

## 1. SNI（Server Name Indication）

- **作用層級**：TLS 握手階段（傳輸層應用層之間）
    
- **解決問題**：在同一台伺服器（或同一組 IP）上要支援多個 TLS 憑證（多個域名）時，客戶端必須告訴伺服器它想訪問哪個「虛擬主機名」。
    
- **工作流程**：
    
    1. 客戶端發起 TCP 連線→TLS ClientHello。
        
    2. 在 ClientHello 裡的「擴展欄位」帶上目標域名（即 SNI）。
        
    3. 伺服器根據 SNI 選擇對應的憑證回應，完成 TLS 握手。
        
- **應用場景**：CDN、多域名共用同 IP、HTTPS 多站點託管。
    

---

## 2. ESNI / ECH（Encrypted SNI / Encrypted Client Hello）

- **演進背景**：  
    傳統 SNI 以明文方式出現在 TLS 握手的 ClientHello 裡，**任何中間人**（DPI、ISP、政府）都能讀取你要訪問的域名。
    
- **ESNI（Encrypted SNI）**：
    
    - 最初草案，用於 **加密 ClientHello 中的 ServerNameExtension**，避免暴露域名。
        
    - 需伺服器端在 DNS 上發佈「公鑰」，客戶端用公鑰加密 SNI。
        
- **ECH（Encrypted Client Hello）**：最新標準，擴展加密**整個 ClientHello**（不只 SNI），包括擴展欄位與協商參數，進一步保護協商隱私。
    
- **好處**：
    
    - 防止中間人根據 SNI 封鎖特定站點
        
    - 提升隱私，連域名、協議擴展訊息都不可見
        
- **部署情況**：需要客戶端（瀏覽器／Clash）與伺服器（Web 伺服器或 CDN）都支援，目前仍在逐步推動中。
    

---

## 3. DPI（Deep Packet Inspection，深度包檢測）

- **作用層級**：介於網路層（IP）與應用層之間，可「查看」封包內部的協議與內容特徵
    
- **核心功能**：
    
    - 不僅看 IP 標頭、Port，還能解析 TCP/UDP payload 裡的協議（HTTP、TLS、SSH、BitTorrent 等），甚至檢測特定簽名或行為模式。
        
    - 可做「流量分類」、「入侵偵測／阻擋（IDS/IPS）」、「內容過濾（反病毒、關鍵字封鎖）」等。
        
- **應用場景**：企業防火牆、ISP 網管、國家級網路審查。
    
- **對抗方式**：
    
    - 全流量加密（TLS、ECH）
        
    - 協議混淆（obfs、XTLS-Reality）
        
    - 域名前置（Domain Fronting）
        

---

## 4. gRPC

- **定位**：一套高效的、基於 HTTP/2 的遠端程式呼叫（RPC，Remote Procedure Call）框架，由 Google 開源。
    
- **主要特點**：
    
    1. **多路複用**：依賴 HTTP/2，多個請求／回應可以在同一 TCP 連線上同時進行。
        
    2. **二進位序列化**：使用 Protocol Buffers（.proto）定義介面與資料結構，傳輸更緊湊、效能高。
        
    3. **多種呼叫類型**：單向請求、伺服端串流、客戶端串流、雙向串流。
        
    4. **跨語言支援**：支援 C++, Java, Python, Go, Rust, Node.js 等多種語言。
        
- **使用方式**：
    
    1. 定義 `.proto` 檔案（訊息與服務介面）。
        
    2. 用 protoc 生成各種語言的服務端／客戶端存根。
        
    3. 實作服務端與呼叫客戶端，就像呼叫本地函式一樣。
        
- **應用場景**：微服務通訊、高效分布式系統、行動／IoT 裝置後端。
    

---

## 5. QUIC（Quick UDP Internet Connections）

- **定位**：由 Google 發起的下一代傳輸協議，運行在 UDP 之上，整合了 TCP＋TLS＋HTTP/2（或 HTTP/3）多種優點。
    
- **核心特性**：
    
    1. **連線建立更快**：0-RTT 和 1-RTT 握手，啟動延遲大幅降低。
        
    2. **內建加密**：把 TLS 1.3 納入協議本身，全程加密（包括標頭）。
        
    3. **多路複用無隊頭阻塞**：在同一 QUIC 連線中不同資料流互不阻塞，避免單一路徑失敗影響整條連線。
        
    4. **連線遷移**：客戶端換網（Wi-Fi ↔ 行動數據）時可保留同一 QUIC 連線。
        
- **與 HTTP/3**：  
    HTTP/3 就是以 QUIC 為底層運輸層協議的新版 HTTP，也因此具備上述優勢。
    
- **應用現狀**：多數現代瀏覽器（Chrome、Edge、Firefox）與大型網站（Google、YouTube、Facebook、Cloudflare）已陸續支援。
    

---

### 綜合比較

|技術|加密層級|主要用途|對抗 DPI／審查|
|---|---|---|---|
|SNI|明文|TLS 多域名憑證匹配|無|
|ESNI/ECH|加密 TLS 握手|隱藏訪問域名與握手訊息|可抵禦 SNI 封鎖|
|DPI|可解析加密流量特徵|流量監測、過濾|被混淆／加密後失效|
|gRPC|透過 TLS|高效 RPC 呼叫|需要 TLS/ECH 才隱蔽|
|QUIC/HTTP3|內建 TLS|低延遲多路複用傳輸|隱藏標頭，提升抗封鎖|

這些技術互相配合，就能在保障安全與效能的同時，最大程度地隱藏流量特徵，提升在嚴格網路審查環境下的可用性和穩定性。