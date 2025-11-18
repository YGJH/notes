
## 讀取資料
```python
train = pd.read_csv('../input/GiveMeSomeCredit/cs-training.csv')

test = pd.read_csv('../input/GiveMeSomeCredit/cs-test.csv')

train.head(3)
```

## to_numeric()
```python
import pandas as pd

data = ['10', '20', '30']

numeric_data = pd.to_numeric(data)

print(numeric_data)# 輸出: [10, 20, 30] 以 numpy 數組形式儲存
```

## .apply(pd.to_numeric)是甚麼
```python
train[train.select_dtypes(['object','category']).columns] = train.select_dtypes(['object','category']).apply(lambda x: x.astype('category'))
```

這段程式碼中的 `.apply(pd.to_numeric)` 是在 DataFrame 上使用 `apply` 方法，對每一個被選取的數值欄位（Series）套用 `pd.to_numeric` 函式。也就是說，對於這些欄位中的每個值，都嘗試將其轉換成數值型態，確保資料正確地以數值格式呈現。

## astype
```python
# 將某個 Series 轉成浮點數
numeric_series = some_series.astype('float')
```
在你的程式碼中，`.apply(lambda x: x.astype('category'))` 用於把物件或已分類的欄位轉換成 `category` 型別，有助於節省記憶體並提升分類變數的處理效率。

astype 是 Pandas 中的 DataFrame 或 Series 方法，*用來將資料轉換成指定的資料型別*。也就是說，可以使用 astype 將某欄位從一種型態轉換為另一種型態，例如把整數轉換成浮點數、把字串轉換成分類型別（category）等。 例如： # 將某個 Series 轉成浮點數 numeric_series = some_series.astype('float')在你的程式碼中，.apply(lambda x: x.astype(&#39;category&#39;)) 用於把物件或已分類的欄位轉換成 category 型別，有助於節省記憶體並提升分類變數的處理效率。

## Reduce memory
```python
def reduce_mem_usage(df):

    """ iterate through all the columns of a dataframe and modify the data type

        to reduce memory usage.        

    """
    # 計算資料開始時佔用的記憶體大小 (以 MB 為單位)
    start_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage of dataframe is {:.2f} MB'.format(start_mem))
    # 遍歷 dataframe 中的所有欄位
    for col in df.columns:
        # 取得該欄位的數據型別
        col_type = df[col].dtype
        # 取得數據型別的名稱
        name = df[col].dtype.name
        # 如果該欄位的型別不是 object 且也不是 category
        if col_type != object and col_type.name != 'category':
            # 計算該欄位的最小值和最大值
            c_min = df[col].min()
            c_max = df[col].max()
            # 如果欄位是整數類型，逐步降低整數型別的記憶體使用 
            if str(col_type)[:3] == 'int':
                # 如果數值範圍適合 int8 型別，轉換為 int8
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:

                    df[col] = df[col].astype(np.int8)

                # 如果數值範圍適合 int16 型別，轉換為 int16

                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:

                    df[col] = df[col].astype(np.int16)

                # 如果數值範圍適合 int32 型別，轉換為 int32

                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:

                    df[col] = df[col].astype(np.int32)

                # 如果數值範圍適合 int64 型別，轉換為 int64

                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:

                    df[col] = df[col].astype(np.int64)  

            else:

                # 如果欄位是浮點數類型，逐步降低精度以減少記憶體占用

                # 如果數值範圍適合 float16 型別，轉換為 float16

                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:

                    df[col] = df[col].astype(np.float16)

                # 如果數值範圍適合 float32 型別，轉換為 float32

                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:

                    df[col] = df[col].astype(np.float32)

                # 否則預設轉換為 float64

                else:

                    df[col] = df[col].astype(np.float64)

        else:

            # 如果欄位型別是 object 或 category，轉換為 category 以減少記憶體使用

            df[col] = df[col].astype('category')

  

    # 計算調整後的記憶體使用量（以 MB 為單位）

    end_mem = df.memory_usage().sum() / 1024**2

    # 印出新的記憶體使用量

    print('Memory usage after optimization is: {:.2f} MB'.format(end_mem))

    # 印出記憶體縮減的百分比

    print('Decreased by {:.1f}%'.format(100 * (start_mem - end_mem) / start_mem))

    # 回傳轉換後的 dataframe

    return df

  

# 對 train 和 test 資料集應用 reduce_mem_usage 函數

train = reduce_mem_usage(train)

test = reduce_mem_usage(test)
```

### np.iinfo(np.int8)
- 可以用來檢查型別是否落在int8型別內
- 會回傳一個物件，包含int8的最大值跟最小值
```python
np.iinfo(np.int8).max // 127
np.iinfo(np.int8).min // -128
```
### 應用...可以檢查這個數字能否轉成int8, int16等更小的變數
```python
if str(col_type)[:3] == 'int':
    # 如果數值範圍適合 int8 型別，轉換為 int8
    if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
        df[col] = df[col].astype(np.int8)
    # 如果數值範圍適合 int16 型別，轉換為 int16
    elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
        df[col] = df[col].astype(np.int16)
    # 如果數值範圍適合 int32 型別，轉換為 int32
    elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
        df[col] = df[col].astype(np.int32)
    # 如果數值範圍適合 int64 型別，轉換為 int64
    elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
        df[col] = df[col].astype(np.int64)  
else:
    # 如果欄位是浮點數類型，逐步降低精度以減少記憶體占用
    # 如果數值範圍適合 float16 型別，轉換為 float16
    if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
        df[col] = df[col].astype(np.float16)
    # 如果數值範圍適合 float32 型別，轉換為 float32
    elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
        df[col] = df[col].astype(np.float32)
    # 否則預設轉換為 float64
    else:
        df[col] = df[col].astype(np.float64)
```

## XGBtree 的參數
```python
# 定義 XGBoost 的參數字典，這裡設定多項回歸與樹模型的參數

param =  {  

  "verbosity": 0,                                        # 不輸出太多訓練過程訊息

  #'objective': "binary:logistic",                       # 二元分類（此行被註解）

  #'eval_metric': "auc",                                 # 評估指標：AUC（此行被註解）

  'random_state': 42,                                    # 設定隨機種子，確保模型可重現性

  'objective':'reg:squarederror',                       # 目標函數為回歸: 均方誤差

  'eval_metric': 'mae',                                  # 評估指標為平均絕對誤差

  #'early_stopping_rounds': 100,                         # 早停設定（此行被註解）

  #'gpu_id':0,                                          # 使用 GPU 的編號（此行被註解）

  #'predictor':"gpu_predictor",                          # 使用 GPU 預測器（此行被註解）

  #'tree_method': "exact",                               # 小資料集精確樹方法（被註解）

  #'tree_method': 'gpu_hist',                            # 大資料集使用 GPU 的直方圖算法（被註解）

  'booster': 'gbtree',                                   # 設定使用樹模型

  'lambda': 8.544792472633987e-07,                        # L2 正則化參數

  'alpha': 0.31141671752487043,                          # L1 正則化參數

  'subsample': 0.8779467596981366,                       # 訓練子樣本比例

  'colsample_bytree': 0.9759532762677546,                # 建樹時每棵樹使用的特徵比例

  'learning_rate': 0.008686087328805853,                 # 學習率

  'n_estimators': 6988,                                  # 弱學習器的數量

  'max_depth': 9,                                        # 最大樹深

  'min_child_weight': 2,                                 # 最小子節點權重

  'eta': 3.7603213457541647e-06,                         # eta參數（學習率的一種形式）

  'gamma': 2.1478058456847449e-07,                        # 其他正則化項，控制樹分割的最小損失減少量

  'grow_policy': 'lossguide'                             # 決策樹增長策略，使用 lossguide 演算法

}
```

## Pipeline

Pipeline 語法主要是用來將多個數據處理步驟串聯起來，像是一個流水線。每個步驟都是以 ('name', transformer/estimator) 的形式傳入列表中，然後依序執行。例如：

````python
from sklearn.pipeline import Pipeline

pipe_xgbr1 = Pipeline(
    steps=[
        ('preprocessor', numeric_transformer1),  # 數據預處理：例如補缺、數據轉換
        ('classifier', XGBRegressor(**param))      # 模型訓練：先前定義好參數的 XGBRegressor
    ]
)
````

**語法重點：**

1. **步驟名稱**  
   - 每個步驟需要一個唯一的名字，可以用來在後續存取或調參時引用該步驟。

2. **轉換器和模型**  
   - 第一步驟通常是資料前處理；之後可以是轉換器、特徵選擇器，最終是估計器（例如回歸或分類模型）。  
   - 在 Pipeline 內部，每個步驟的輸出會自動成為下一個步驟的輸入。

3. **一致的流程**  
   - 不論是在訓練（.fit）還是在預測（.predict）時，Pipeline 都會確保所有步驟以相同順序被執行，從而保證流程的一致性。

簡而言之，Pipeline 語法使得數據處理和模型應用更模組化、易維護，同時在調參時也能一併包含所有處理步驟。

## loc
`.loc` 是 Pandas DataFrame 的一個標籤型索引器，用來根據行和列的標籤來選取資料。下面詳細說明其語法和使用方式：

1. **基本語法**  
   語法格式為：  
   ````python
   df.loc[row_selection, column_selection]
   ````
   - `row_selection`：選取資料的行，可以是單一行標籤、標籤列表、布林型索引、或標籤的切片。
   - `column_selection`：同理，選取資料的欄，支持單一欄標籤、標籤列表、布林索引或切片。

2. **完整選取所有行**  
   使用 `:` 表示選取所有行。  
   例如：  
   ````python
   df.loc[:, column_list]
   ````
   這表示對於所有行，只選取 `column_list` 裡的欄位。

3. **舉例說明**  
   在你所提供的程式碼中：  
   ````python
   pipeline1.fit(train.loc[:, num_features1])
   ````
   - `train` 是一個 DataFrame。
   - `num_features1` 為一個包含數值特徵欄位名稱的列表。
   - 使用 `train.loc[:, num_features1]` 表示從 `train` 資料中選取所有行，但只保留 `num_features1` 所指定的欄位。

4. **選擇特定行和欄位**  
   如果想選擇特定行和欄位，可以這樣寫：  
   ````python
   # 選擇行標籤為 'a', 'b' 並且欄位為 'col1', 'col2'
   df.loc[['a', 'b'], ['col1', 'col2']]
   ````
   可以根據行與欄的標籤來過濾資料。

5. **使用布林型索引**  
   `.loc` 也可以搭配布林條件過濾行，例如：  
   ````python
   df.loc[df['col1'] > 100, :]
   ````
   上面的程式碼會選取所有 'col1' 大於 100 的行，並返回所有欄位。

總結來說，`.loc` 是一個非常靈活且強大的資料選取工具，可以根據行和欄標籤來精準定位和過濾資料，非常適用於 DataFrame 的操作。

## fit
[[scikit-learn (fit)]]