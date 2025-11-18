```cpp
#include <iostream>

#include <vector>

using namespace std;

  

// 使用深度受限的深度優先搜索（Depth-Limited DFS）

// 若在當前深度內找到目標節點則回傳 true，否則回傳 false

bool depthLimitedDFS(int current, int target, int depth, const vector<vector<int>> &graph) {

    if (depth < 0)

        return false; // 超過限制深度

    if (current == target)

        return true;  // 找到目標

  

    // 遍歷所有鄰接節點

    for (int neighbor : graph[current]) {

        if (depthLimitedDFS(neighbor, target, depth - 1, graph))

            return true;

    }

    return false;

}

  

// 迭代加深搜索：從深度 0 開始，不斷增加深度限制，直到達到 maxDepth 為止

bool iterativeDeepeningSearch(int start, int target, const vector<vector<int>> &graph, int maxDepth) {

    for (int depth = 0; depth <= maxDepth; ++depth) {

        if (depthLimitedDFS(start, target, depth, graph)) {

            cout << "在深度 " << depth << " 找到目標節點。" << endl;

            return true;

        }

    }

    return false; // 若超過最大深度仍未找到目標

}

  

int main() {

    // 範例說明：輸入節點數與邊數，並建立圖結構（此處假設節點以 0 為起始索引）

    int n, m;

    cout << "請輸入節點數 n 與邊數 m：";

    cin >> n >> m;

    vector<vector<int>> graph(n);

  

    cout << "請依次輸入 m 條邊（格式：u v）：\n";

    for (int i = 0; i < m; i++) {

        int u, v;

        cin >> u >> v;

        // 如果輸入的節點是以 1 為起始索引，這裡需要減 1，調整為 0 索引

        graph[u].push_back(v);

        graph[v].push_back(u); // 無向圖，雙向加入

    }

    int start, target;

    cout << "請輸入起始節點與目標節點：";

    cin >> start >> target;

    // 如果輸入的節點編號是從 1 開始，請在此處執行：start--; target--;

    // 設定最大搜尋深度，此處以 n 作為上限（實際情況可根據問題調整）

    if (!iterativeDeepeningSearch(start, target, graph, n)) {

        cout << "目標節點未找到。" << endl;

    }

    return 0;

}
```