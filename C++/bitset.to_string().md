```cpp
#include <iostream>
#include <bitset>
using namespace std;
signed main() {
    bitset<10> b{43};
    cout << b.to_string() << endl;
    cout << b.to_string('x') << endl;
    cout << b.to_string('a' , 'b') << endl;

```
output:
```
0000101011
xxxx1x1x11
aaaabababb
```
