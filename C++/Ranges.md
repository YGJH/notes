
## 例子
- 假如要把 1~100內的數字進行
	- 平方
	- 找出4的倍數
	- 輸出前五個
- 你可能只能這樣寫
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
signed main() {
    std::vector<int> v(100) ,vv;
    std::iota(v.begin() , v.end() , 1);
    std::transform(v.begin() , v.end() , v.begin() ,[](int i) {
    return i * i;});
    std::copy_if(v.begin() , v.end() , std::back_inserter(vv) , [](int i) {
    return i % 4 == 0;});
    for(int i = 0 ; i < 5 ; i++) {
        std::cout << vv[i] << ' ';
    }
}
```


## Ranges 範圍
- 所有可以疊代的集合，基本上所有STL容器都可以視為一個ranges

## Views視圖
- 就像是個代理人，你只能透過這個透鏡去看到range裡的物件，**不擁有物件的操作權限(remove,add,modify...)** 
- **可以透過range的起點跟終點**生成各種View

## View Adapter 適配器\轉接頭
- 其實就只是可以直接把元素計算出來的值透過Adapter存到新的陣列裡，不會修改到原本的容器


```cpp
#include <ranges>
#include <iostream>
using namespace std;

signed main() {

    for(auto i : std::views::iota(1)

    | std::views::transform([] (int i) {return i * i;} )

    | std::views::filter([] (int i) {return i % 4 == 0;})

    | std::views::take(5))
 
	{
        cout << i << ' ';
    }
}
```