## pandas describe
- 可以用來大致上看到數據的分布，如:
	- 四分位數(25%, 50%, 75%)
	- 均值、標準差、最大值、最小值、
```python
import pandas as pd
data = pd.read_csv('test.csv')

print(data.describe())
```
- 當傳入include='all' 來獲取所有欄位統計數據

## pandas nunique()
- 可以用來計算 Dataframe 或 Series 中不重複值的數量，這個方法可以用來了解每個欄位的唯一個數，藉此查閱資料的離散程度或個別數量。
```python
import pandas as pd
# 假設我們有一個 DataFrame
data = {
    'A': [1, 2, 2, 3, 4, 4],
    'B': ['a', 'b', 'a', 'a', 'c', 'c']
}
df = pd.DataFrame(data)

# 計算每個欄位的不重複值的數量

print(df.nunique())
```
- 輸出的結果可能是：
```output
A    4
B    3
dtype: int64
```

## DataFrame
pandas 的 DataFrame 是一種非常靈活且功能強大的資料結構，用來儲存和處理表格型資料。它類似於 Excel 工作表或 SQL 中的資料表，具有行與列的結構，可以讓你對各種資料進行讀取、篩選、分組、彙總、連接等操作。

DataFrame 的主要特點包括：

- **多樣的資料來源匯入**  
    你可以從 CSV、Excel、SQL 資料庫、JSON 等多種格式攫取資料，輕鬆建立 DataFrame。
    
- **異質資料類型**  
    每一欄可以包含不同的資料型態（例如數值、字串、日期等），這使得它非常適合作為現實生活中各種資料的容器。
    
- **豐富的操作功能**  
    DataFrame 提供了大量內建方法，比如篩選行、排序、合併、樞紐分析等，讓資料前處理與分析變得更加方便。
    
- **索引與標籤**  
    每一行和每一列都有自己的索引，允許你使用標籤來存取資料，增強了操作的靈活性和可讀性。