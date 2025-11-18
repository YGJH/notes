**生成Java Swing GUI的JFrame教學**

我要為您準備一個詳細的Java Swing GUI教學。首先，我會選擇以傳統中文來解釋，因為問題是以這樣的方式提問的。這篇教學將會包含Java框架的基礎，如設置窗口大小、關閉操作，並介紹各種組件如JButton和JLabel，再加上布局管理器和事件處理邏輯。我會提供一段代碼範例以步驟形式來解釋如何運行它。

已思考幾秒鐘

以下是一份以 Java Swing 建立 `JFrame` 視窗應用的完整教學，包含基本概念、範例程式碼與常用提示，讓你快速上手：

---

## 1. 環境準備

1. **JDK 安裝**  
    下載並安裝 Java Development Kit (JDK)（建議使用 JDK 8 以上版本）。安裝後，確認終端機輸入 `java -version` 與 `javac -version` 可正確顯示版本號。
    
2. **開發工具**  
    可使用 IntelliJ IDEA、Eclipse 或 VS Code（需安裝 Java Extension Pack）等 IDE 進行開發。
    

---

## 2. Swing 與 JFrame 基本概念

- **Swing**：Java 標準的 GUI 套件，包含視窗元件（components）與佈局管理器（layout manager）。
    
- **JFrame**：最常用的頂層容器（top-level container），相當於一個獨立視窗（window）。
    

---

## 3. 建立最簡單的 JFrame

```java
import javax.swing.JFrame;
import javax.swing.SwingUtilities;

public class SimpleFrame {
    public static void main(String[] args) {
        // 建議將 GUI 建立放到 Event Dispatch Thread (EDT) 中
        SwingUtilities.invokeLater(() -> {
            // 建立 JFrame 實例
            JFrame frame = new JFrame("我的第一個 Swing 視窗");
            
            // 設定視窗關閉時結束程式
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            
            // 設定視窗大小 (寬, 高)
            frame.setSize(400, 300);
            
            // 將視窗置中
            frame.setLocationRelativeTo(null);
            
            // 顯示視窗
            frame.setVisible(true);
        });
    }
}
```

**重點說明：**

- `invokeLater(...)`：確保 GUI 建立與更新在 Swing 的事件處理緒中執行，避免執行緒競爭問題。
    
- `setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)`：點擊關閉按鈕時結束程式。
    
- `setSize(...)`、`setLocationRelativeTo(null)`：設定大小與置中顯示。
    

---

## 4. 在 JFrame 中添加元件

通常我們會在 `JFrame` 上加入 `JPanel`，再將各種元件（按鈕、標籤、文字框……）加到 `JPanel`。

```java
import javax.swing.*;
import java.awt.*;

public class ComponentDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("元件示範");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            
            // 建立一個 JPanel，預設使用 FlowLayout
            JPanel panel = new JPanel();
            panel.setLayout(new FlowLayout(FlowLayout.CENTER, 10, 10));
            
            // 建立並加入 JLabel、JButton、JTextField
            JLabel label = new JLabel("輸入你的名字：");
            JTextField textField = new JTextField(15);
            JButton button = new JButton("送出");
            
            panel.add(label);
            panel.add(textField);
            panel.add(button);
            
            // 將 panel 加到 frame
            frame.getContentPane().add(panel);
            
            frame.pack();            // 根據內容自動調整視窗大小
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**重點說明：**

- `JPanel`：容器元件，用來管理內部元件佈局。
    
- `FlowLayout`：水平從左到右排列，遇滿版則自動換行。
    
- `pack()`：讓視窗根據內部元件及佈局自動計算最適大小。
    

---

## 5. 事件處理 (Event Handling)

加入按鈕點擊事件，示範如何讀取文字框並顯示對話框：

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class EventDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("事件處理示範");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            
            JPanel panel = new JPanel(new FlowLayout(10, 10, 10));
            JLabel label = new JLabel("輸入：");
            JTextField textField = new JTextField(10);
            JButton button = new JButton("顯示訊息");
            
            // 為按鈕註冊 ActionListener
            button.addActionListener((ActionEvent e) -> {
                String input = textField.getText();
                JOptionPane.showMessageDialog(frame, "你輸入的是：" + input,
                                              "提示", JOptionPane.INFORMATION_MESSAGE);
            });
            
            panel.add(label);
            panel.add(textField);
            panel.add(button);
            
            frame.getContentPane().add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

---

## 6. 常見佈局管理器

|佈局管理器|特性|
|---|---|
|`FlowLayout`|自動水平排列，滿版換行|
|`BorderLayout`|分為五區：`NORTH`、`SOUTH`、`EAST`、`WEST`、`CENTER`|
|`GridLayout`|依指定列列數或行數等分格子，均等分配大小|
|`BoxLayout`|垂直或水平線性排列|
|`GridBagLayout`|最靈活但最複雜，可細緻控制元件在格子的位置與權重|

---

## 7. 範例：使用 BorderLayout

```java
import javax.swing.*;
import java.awt.*;

public class BorderLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("BorderLayout 示範");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            
            frame.setLayout(new BorderLayout(5, 5));
            
            frame.add(new JButton("北 (North)"), BorderLayout.NORTH);
            frame.add(new JButton("南 (South)"), BorderLayout.SOUTH);
            frame.add(new JButton("東 (East)"),  BorderLayout.EAST);
            frame.add(new JButton("西 (West)"),  BorderLayout.WEST);
            frame.add(new JButton("中 (Center)"), BorderLayout.CENTER);
            
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

---

## 8. 小結與建議

1. **盡量把所有 Swing 元件的建立與更新都包在 `SwingUtilities.invokeLater` 中**，確保線程安全。
    
2. **多使用 `pack()` 管理視窗大小**，除非要設定固定尺寸。
    
3. **選擇合適的佈局管理器**，不要用絕對定位（`null` layout），以免在不同解析度下顯示異常。
    
4. **事件處理**：依需求選擇 `ActionListener`、`MouseListener`、`KeyListener` 等。
    
5. **擴展元件**：若需求複雜，可自訂類別繼承 `JPanel` 或其他 Swing 元件，將 UI 與邏輯分離。
    

透過上述內容，你已經掌握建立基本 Swing 視窗、添加元件並處理事件的方法。接下來可依需求深究自訂佈局、拖放、繪圖（`paintComponent`）、多執行緒更新 GUI 等進階主題。祝你開發順利！