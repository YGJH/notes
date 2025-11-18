- 可以用來大致上看到數據的分布，如:
	- 四分位數(25%, 50%, 75%)
	- 均值、標準差、最大值、最小值、
```python
import pandas as pd
data = pd.read_csv('test.csv')

print(data.describe())
```
- 當傳入include='all' 來獲取所有欄位統計數據
