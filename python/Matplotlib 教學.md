### Matplotlib 教學：讓你記住的「畫圖小秘訣」

嘿！matplotlib 是 Python 的繪圖神器，但很多人像你一樣容易忘記，因為它有點像畫畫工具箱，裡面零件多。為了讓你好記，我會用**「故事式記憶法」**來教：想像你是在「畫一幅畫」，從「準備畫布」到「加裝飾」，最後「展示或保存」。每個步驟用簡單範例，配上記憶口訣。記住核心：**plt** 是 pyplot 的縮寫，像你的「畫筆工具」（plot tool）。

我會用 Python 代碼範例解釋（假設你有 numpy 安裝，用來生成數據；如果沒有，就用列表取代）。範例都超短，抄下來就能跑。讓我們一步步來！

#### 1. **準備畫布：導入和基本設定**
   - **記憶口訣**：想像「plt」是你的「畫圖筆」（plot），先「import」進來，就像拿筆準備畫。
   - 核心代碼：
     ```python
     import matplotlib.pyplot as plt  # 記住：plt 是 pyplot 的簡稱，像你的畫筆
     import numpy as np  # 選用，幫忙生成數據；沒 numpy 就用 list
     ```
   - **為什麼好記**：永遠從 `import matplotlib.pyplot as plt` 開始，別忘了 `plt` 這個暱稱，後面所有命令都用它開頭。

#### 2. **畫基本圖形：從線開始，像畫線條**
   - **記憶口訣**：畫圖就像「plot（畫線） + show（展示）」。先準備數據（x 和 y，像坐標），然後畫出來。
   - **範例：簡單線圖（Line Plot）**
     - 故事：想像畫一條「上升的山路」。
     ```python
     x = [1, 2, 3, 4]  # x 軸：路程
     y = [1, 4, 9, 16]  # y 軸：高度（平方）
     plt.plot(x, y)  # 畫線：plot(x, y)
     plt.show()  # 展示：show() 一定要加，否則不顯示！
     ```
     - 記憶：`plot` 像「畫線」，傳入 x 和 y 就好。如果只有 y，它自動用 0,1,2... 當 x。
   - **變形：加顏色和樣式**
     - 口訣：顏色用英文縮寫（r=紅, b=藍, g=綠），樣式用符號（--=虛線, o=圓點）。
     ```python
     plt.plot(x, y, 'r--o')  # 紅虛線 + 圓點標記
     plt.show()
     ```
     - 好記：'r--o' 像「紅色（r）虛線（--）圓點（o）」的密碼。

   - **其他常見圖形：記住關鍵字像「動作」**
     - 散點圖（Scatter Plot）：像「灑點」，用 `scatter`。
       ```python
       plt.scatter(x, y, color='blue', marker='x')  # 藍色 x 標記
       plt.show()
       ```
       - 記憶：scatter = 「散射點」，適合看分佈。
     - 柱狀圖（Bar Chart）：像「堆柱子」，用 `bar`。
       ```python
       categories = ['A', 'B', 'C']
       values = [3, 7, 5]
       plt.bar(categories, values, color='green')  # 綠柱
       plt.show()
       ```
       - 記憶：bar = 「酒吧柱子」，垂直堆疊。
     - 直方圖（Histogram）：像「統計桶」，用 `hist`。
       ```python
       data = np.random.randn(1000)  # 隨機數據
       plt.hist(data, bins=30, color='purple')  # 30 個桶，紫色
       plt.show()
       ```
       - 記憶：hist = 「歷史統計」，bins 是「分桶數」。

#### 3. **加裝飾：讓圖好看，像加標題和貼紙**
   - **記憶口訣**：想像圖是「一幅畫」，要加「標題（title）」、「x/y 軸說明（label）」、「圖例（legend）」。
   - 範例：完整線圖 + 裝飾
     ```python
     x = np.linspace(0, 10, 100)  # 生成 100 點從 0 到 10
     y1 = np.sin(x)  # 正弦波
     y2 = np.cos(x)  # 餘弦波

     plt.plot(x, y1, 'b-', label='Sin')  # 藍實線，標籤 Sin
     plt.plot(x, y2, 'r--', label='Cos')  # 紅虛線，標籤 Cos

     plt.title('Sine and Cosine Waves')  # 標題：像畫的名字
     plt.xlabel('X Axis (Time)')  # x 軸標籤
     plt.ylabel('Y Axis (Value)')  # y 軸標籤
     plt.legend()  # 顯示圖例：記住要加 label 才有效
     plt.grid(True)  # 加格線，像畫紙上的線
     plt.show()
     ```
     - **好記技巧**：
       - title = 「頂部標題」。
       - xlabel/ylabel = 「x/y 說明」。
       - legend = 「圖例」，先在 plot 裡加 `label='名字'`。
       - grid = 「加網格」，True 開啟。

#### 4. **多圖或子圖：像畫多幅畫在同一張紙**
   - **記憶口訣**：用 `subplot` 分割畫布，像「row x col」的格子，然後指定位置。
   - 範例：兩個子圖
     ```python
     fig, axs = plt.subplots(1, 2)  # 1 行 2 列的畫布

     axs[0].plot(x, y1)  # 第一個子圖畫 Sin
     axs[0].set_title('Sin')

     axs[1].plot(x, y2)  # 第二個子圖畫 Cos
     axs[1].set_title('Cos')

     plt.tight_layout()  # 自動調整間距，避免擠
     plt.show()
     ```
     - 記憶：subplots(行, 列) = 「分割畫布」，axs 是陣列，像格子[0], [1]。

#### 5. **保存圖片：像拍照存檔**
   - **記憶口訣**：不用 show，直接 `savefig` 存檔，像「保存圖片」。
   - 範例：
     ```python
     plt.plot(x, y)
     plt.savefig('my_plot.png')  # 存成 PNG；可加 dpi=300 提高品質
     # plt.show()  # 如果要顯示再加
     ```
     - 好記：savefig = 「保存圖」，檔名帶副檔名（png, jpg, pdf）。

#### 額外記憶小Tips，讓你永遠不忘：
- **常見錯誤避免**：忘記 `plt.show()` → 圖不顯示；數據型別錯 → 用 list 或 numpy array。
- **顏色/樣式速記**：顏色（c='red' 或 'r'），樣式（'-'實線, '--'虛線, 'o'圓點, '^'三角）。
- **進階但好記**：想 3D 圖？用 `from mpl_toolkits.mplot3d import Axes3D`，但先掌握 2D。
- **練習法**：抄以上範例跑一次，改改數據，就記住了。想像每次畫圖是「plot + 裝飾 + show/save」。
- 如果你有特定圖型忘記（如餅圖 pie），告訴我，我可以補充！（pie 用 `plt.pie(values, labels='A B C'.split())`）

這樣教，希望像故事一樣好記。下次忘記，就想「畫筆 plt → 畫線 plot → 加標題 → 展示 show」。如果需要代碼執行示範或圖片，告訴我確認喔！