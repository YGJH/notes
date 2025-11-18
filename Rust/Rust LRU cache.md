下面是完全只用標準函式庫（`std`）實作的一個 LRU Cache，時間複雜度皆為 O(1)：

```rust
use std::{
    cell::RefCell,
    collections::HashMap,
    hash::Hash,
    rc::{Rc, Weak},
};

/// 雙向鏈結點
struct Node<K, V> {
    key: K,
    value: V,
    prev: Option<Weak<RefCell<Node<K, V>>>>,
    next: Option<Rc<RefCell<Node<K, V>>>>,
}

/// LRU Cache 結構
pub struct LruCache<K, V> {
    capacity: usize,
    map: HashMap<K, Rc<RefCell<Node<K, V>>>>,
    head: Option<Rc<RefCell<Node<K, V>>>>,
    tail: Option<Rc<RefCell<Node<K, V>>>>,
}

impl<K, V> LruCache<K, V>
where
    K: Eq + Hash + Clone,
    V: Clone,
{
    /// 建立容量為 capacity 的 LRU Cache
    pub fn new(capacity: usize) -> Self {
        Self {
            capacity,
            map: HashMap::new(),
            head: None,
            tail: None,
        }
    }

    /// 取得 key 對應的值（複製），若存在則標記為最近使用
    pub fn get(&mut self, key: &K) -> Option<V> {
        if let Some(node_rc) = self.map.get(key) {
            // 取出值的複本
            let val = node_rc.borrow().value.clone();
            // 移動到尾端
            let node = node_rc.clone();
            self.move_to_tail(node);
            Some(val)
        } else {
            None
        }
    }

    /// 插入 (key, value)，若已存在就更新並標記最近使用；
    /// 若超過容量就淘汰最久未使用的
    pub fn put(&mut self, key: K, value: V) {
        if let Some(node_rc) = self.map.get(&key) {
            // 已存在：更新值並移到尾端
            node_rc.borrow_mut().value = value;
            let node = node_rc.clone();
            self.move_to_tail(node);
        } else {
            // 不存在：檢查是否需要淘汰
            if self.map.len() == self.capacity {
                self.pop_head();
            }
            // 建立新節點，插到尾端
            let new_node = Rc::new(RefCell::new(Node {
                key: key.clone(),
                value,
                prev: None,
                next: None,
            }));
            self.append_to_tail(new_node.clone());
            self.map.insert(key, new_node);
        }
    }

    /// 移除並回傳最舊的（頭節點）
    fn pop_head(&mut self) {
        if let Some(old_head) = self.head.take() {
            // 下一個節點要成為新的 head
            if let Some(next) = old_head.borrow().next.clone() {
                next.borrow_mut().prev = None;
                self.head = Some(next);
            } else {
                // 只有一個節點
                self.tail = None;
            }
            // 從 map 中刪除
            let key = old_head.borrow().key.clone();
            self.map.remove(&key);
        }
    }

    /// 把節點從原位置摘下，再接到 tail
    fn move_to_tail(&mut self, node: Rc<RefCell<Node<K, V>>>) {
        // 如果已經是尾端，就不動
        if let Some(tail_rc) = &self.tail {
            if Rc::ptr_eq(&node, tail_rc) {
                return;
            }
        }
        self.remove_node(&node);
        self.append_to_tail(node);
    }

    /// 把節點摘下（修正前後節點連結）
    fn remove_node(&mut self, node: &Rc<RefCell<Node<K, V>>>) {
        let prev = node.borrow().prev.clone();
        let next = node.borrow().next.clone();

        // 前一個點指向 next
        if let Some(prev_weak) = prev {
            if let Some(prev_rc) = prev_weak.upgrade() {
                prev_rc.borrow_mut().next = next.clone();
            }
        } else {
            // 沒有前一個，代表 node 是 head
            self.head = next.clone();
        }

        // 後一個點指向 prev
        if let Some(next_rc) = next {
            next_rc.borrow_mut().prev = prev.clone();
        } else {
            // 沒有後一個，代表 node 是 tail
            self.tail = prev.and_then(|w| w.upgrade());
        }

        // 清掉自己的 prev/next
        node.borrow_mut().prev = None;
        node.borrow_mut().next = None;
    }

    /// 把節點接到尾端
    fn append_to_tail(&mut self, node: Rc<RefCell<Node<K, V>>>) {
        match self.tail.take() {
            Some(old_tail) => {
                old_tail.borrow_mut().next = Some(node.clone());
                node.borrow_mut().prev = Some(Rc::downgrade(&old_tail));
                self.tail = Some(node);
            }
            None => {
                // 空 list
                self.head = Some(node.clone());
                self.tail = Some(node);
            }
        }
    }
}

#[cfg(test)]
mod tests {
    use super::LruCache;

    #[test]
    fn basic_lru() {
        let mut cache = LruCache::new(2);
        cache.put(1, "one");
        cache.put(2, "two");
        assert_eq!(cache.get(&1), Some("one"));  // [2,1]
        cache.put(3, "three");                   // 淘汰 2 → [1,3]
        assert_eq!(cache.get(&2), None);
        assert_eq!(cache.get(&3), Some("three"));
        cache.put(4, "four");                    // 淘汰 1 → [3,4]
        assert_eq!(cache.get(&1), None);
        assert_eq!(cache.get(&3), Some("three"));
        assert_eq!(cache.get(&4), Some("four"));
    }
}
```

**重點說明**

1. **雙向鏈結串列**：用 `Rc<RefCell<Node>>` 與 `Weak<RefCell<Node>>` 建立節點之間的 `prev`／`next` 指標，確保節點在記憶體中有唯一擁有者（HashMap 存放）且可被多處引用。
    
2. **HashMap**：從 key 快速定位到對應的節點指標。
    
3. **O(1) 操作**
    
    - `get`：透過 map 找到節點後，取值並呼叫 `move_to_tail`。
        
    - `put`：若 key 已存在，更新值後 `move_to_tail`；否則必要時 `pop_head` 淘汰，再 `append_to_tail`。
        
4. **記憶體安全**：全程使用安全的 Rust 型別（`Rc`／`RefCell`／`Weak`），無須 `unsafe`。
    

如此一來，你就擁有一個純標準函式庫的 LRU Cache。

