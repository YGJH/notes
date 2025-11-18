### Seaborn 教學：讓你記住的「美化統計圖小秘訣」

嘿！既然你記不住 matplotlib，seaborn 就是它的「美妝版」升級包——基於 matplotlib，但專注統計圖形，預設顏色和樣式超美，適合資料分析。想像 seaborn 是「畫圖的化妝師」（sns = seaborn's nickname），它幫 matplotlib 自動加濾鏡，讓圖看起來專業又好記。

我會用**「故事式記憶法」**教你：想像你在「打扮一張統計圖」，從「選衣服（主題）」到「加配件（裝飾）」，最後「拍照保存」。每個步驟配簡單範例（用 Python 代碼，假設你有 pandas 和 numpy；seaborn 內建數據集如 iris、tips，讓你不用自己準備數據）。範例超短，抄下去就能跑。記住核心：**sns** 是 seaborn 的縮寫，像你的「統計美妝師」（stats n' style）。

**先提醒**：seaborn 依賴 matplotlib，所以永遠先 import matplotlib.pyplot as plt。seaborn 圖最後用 plt.show() 顯示。

#### 1. **準備化妝台：導入和基本設定**
   - **記憶口訣**：想像「sns」是「seaborn 的美妝箱」，先 import 進來，就像準備化妝品。加 `set_theme()` 選風格，像挑衣服。
   - 核心代碼：
     ```python
     import seaborn as sns  # 記住：sns 是 seaborn 的簡稱，像美妝師
     import matplotlib.pyplot as plt  # 必備，因為 seaborn 用它畫底圖
     import pandas as pd  # 常用，處理數據；或用 numpy
     sns.set_theme(style="whitegrid")  # 選主題：whitegrid（白格子）、darkgrid、white、dark、ticks
     ```
     - **為什麼好記**：永遠從 `import seaborn as sns` 開始，`set_theme()` 讓圖預設美（顏色、格線）。不設也行，但設了更好看。
     - **內建數據集**：seaborn 有免費樣本數據，像「試衣模特兒」。用 `sns.load_dataset('iris')` 載入（iris 是花朵數據，tips 是小費數據）。

#### 2. **畫基本圖形：從統計線開始，像畫曲線美**
   - **記憶口訣**：seaborn 的函數像「動作 + plot」（e.g., lineplot = 畫線），傳入數據框（DataFrame），自動處理 x/y/hue（hue=顏色分組）。
   - **範例：簡單線圖（Line Plot）**
     - 故事：想像畫「不同花種的瓣長變化」。
     ```python
     iris = sns.load_dataset('iris')  # 載入內建數據：iris 花朵
     sns.lineplot(data=iris, x='sepal_length', y='sepal_width', hue='species')  # 畫線：x/y 軸 + hue 分顏色
     plt.show()  # 展示：記住用 plt.show()
     ```
     - 記憶：`lineplot` 像「線條畫」，data=數據，x/y=欄位名，hue=分組顏色（自動美化）。如果數據是 list，用 sns.lineplot(x=..., y=...)。

   - **其他常見圖形：記住關鍵字像「統計動作」**
     - 散點圖（Scatter Plot）：像「灑統計點」，用 `scatterplot`。
       ```python
       tips = sns.load_dataset('tips')  # 載入小費數據
       sns.scatterplot(data=tips, x='total_bill', y='tip', hue='sex', style='time')  # 散點：hue=顏色分性別，style=形狀分時間
       plt.show()
       ```
       - 記憶：scatterplot = 「散射點」，適合看關係；hue/style 加分組變美。
     - 柱狀圖（Bar Plot）：像「堆統計柱」，用 `barplot`。
       ```python
       sns.barplot(data=tips, x='day', y='total_bill', hue='sex')  # 柱狀：x=類別，y=值，hue=分組
       plt.show()
       ```
       - 記憶：barplot = 「酒吧統計柱」，自動加誤差線（信心區間）。
     - 直方圖/密度圖（Histogram/KDE）：像「統計分佈桶」，用 `histplot`。
       ```python
       sns.histplot(data=iris, x='petal_length', hue='species', kde=True)  # 直方圖：kde=True 加密度曲線
       plt.show()
       ```
       - 記憶：histplot = 「歷史分佈」，bins=桶數（預設自動），kde=平滑曲線。
     - 箱形圖（Box Plot）：像「盒子統計」，用 `boxplot`。
       ```python
       sns.boxplot(data=tips, x='day', y='total_bill', hue='smoker')  # 箱形：看分佈、中位數
       plt.show()
       ```
       - 記憶：boxplot = 「盒子圖」，顯示四分位、離群點。
     - 熱圖（Heatmap）：像「顏色地圖」，用 `heatmap`。
       ```python
       flights = sns.load_dataset('flights').pivot(index='month', columns='year', values='passengers')
       sns.heatmap(data=flights, annot=True, cmap='coolwarm')  # 熱圖：annot=加數字，cmap=顏色方案
       plt.show()
       ```
       - 記憶：heatmap = 「熱力圖」，適合矩陣數據；pivot 先轉形。

#### 3. **加裝飾：讓圖更美，像加濾鏡和標籤**
   - **記憶口訣**：seaborn 用自己的 despine() 去邊框，但標題/軸用 plt 的，像「seaborn 美顏 + matplotlib 文字」。
   - 範例：完整散點圖 + 裝飾
     ```python
     sns.scatterplot(data=tips, x='total_bill', y='tip', hue='sex')
     plt.title('Tips by Total Bill and Sex')  # 標題：用 plt.title
     plt.xlabel('Total Bill ($)')  # x 軸標籤
     plt.ylabel('Tip ($)')  # y 軸標籤
     plt.legend(title='Sex')  # 圖例：調整標題
     sns.despine()  # 去掉邊框，讓圖乾淨
     plt.show()
     ```
     - **好記技巧**：
       - title/xlabel/ylabel = 同 matplotlib。
       - legend = 調整圖例，seaborn 自動從 hue 生成。
       - despine() = 「去脊」（remove spines），讓邊框簡潔。
       - 顏色：用 palette='deep' 或 'muted' 在 plot 裡加（e.g., sns.scatterplot(..., palette='Set2')）。

#### 4. **多圖或分面圖：像畫多張照片在相冊**
   - **記憶口訣**：用 `FacetGrid` 或 `pairplot` 分割，像「格子相冊」，自動按變數分圖。
   - 範例：分面圖（FacetGrid）
     ```python
     g = sns.FacetGrid(tips, col='time', row='sex')  # 分格：col=按時間分欄，row=按性別分行
     g.map(sns.scatterplot, 'total_bill', 'tip')  # 每格畫散點
     g.add_legend()  # 加圖例
     plt.show()
     ```
     - 記憶：FacetGrid = 「面格圖」，map(函數, x, y) 填內容。
   - 範例：配對圖（Pair Plot）
     ```python
     sns.pairplot(iris, hue='species', palette='husl')  # 自動畫所有變數配對
     plt.show()
     ```
     - 記憶：pairplot = 「配對圖」，適合探索數據；hue 分顏色。

#### 5. **保存圖片：像存美照**
   - **記憶口訣**：用 plt.savefig，像 matplotlib 一樣。
   - 範例：
     ```python
     sns.lineplot(data=iris, x='sepal_length', y='sepal_width')
     plt.savefig('my_seaborn_plot.png')  # 存 PNG；加 dpi=300 高品質
     # plt.show()  # 如果要顯示再加
     ```
     - 好記：savefig = 「保存圖」，seaborn 圖也用 plt 保存。

#### 額外記憶小Tips，讓你永遠不忘：
- **與 matplotlib 整合**：seaborn 畫完後，用 plt 調整細節（如 plt.xlim(0, 10) 設範圍）。
- **常見錯誤避免**：數據不是 DataFrame → 用 pd.DataFrame() 轉；忘 plt.show() → 不顯示。
- **顏色/樣式速記**：palette='viridis'（漸變色）、'pastel'（柔和）；style='darkgrid' 暗格。
- **進階但好記**：想回歸線？用 `regplot` 或 `lmplot`（linear model plot）。
- **練習法**：用內建數據如 iris/tips 跑範例，改改 hue/x/y，就記住了。想像每次畫圖是「sns.動作plot(data, x, y, hue) + plt.show()」。
- 如果你忘特定圖（如小提琴圖 violinplot），告訴我補充！（violinplot 像箱形+密度，用 sns.violinplot()）。

這樣教，希望像美妝教程一樣好記。下次忘記，就想「美妝師 sns → 選主題 set_theme → 畫統計 plot → 加 plt 裝飾/顯示」。如果需要代碼執行示範或圖片，告訴我確認喔！