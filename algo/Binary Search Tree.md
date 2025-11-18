
## Binary Search Tree
- 右節點比父節點大
- 左節點比父節點小

## Successor

- Assuming that all keys are distinct, **the successor of a node x is the node y such that $key[y]$ is the smallest $key$ > $key[x]$.**
- x 的 successor 就是 **最小但比自己大的節點**
**If x has the largest key in the binary search tree, then we say that x’s successor is NIL**

#### 個人認為Successor 最重要的概念就是，right child 所構成的子樹當中的最小值就是本node的succcessor.
### pseudo code
![[Pasted image 20250430070237.png]]
- 13 的 successor 是 15
- **Time** : The running time O(h)
- Tree-predecessor 是 symmetric(對稱的) to Tree-successor

## predecessor (前任)
- 就是比目前 node 還小的上最大值
- 像是上面那張圖 13 的 predecessor 就是 9
#### 最重要的概念就是往左子樹走的最大值就是此node的predecessor


## insertion (插入)
- 其實就實作而已
![[Pasted image 20250430073048.png]]
### Time O(h)
## Delete
### Case 1 此點沒有child
- 就把父節點的child 指針指向 null

### Case 2 此點有一個child
- 就把父節點的child指針指向此(被刪除的節點)的子節點
### Case 3 此點有兩個child


