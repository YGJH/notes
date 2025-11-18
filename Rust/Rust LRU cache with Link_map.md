開花算法，下面示範一個簡單的 LRU Cache 實作，採用外部套件 `linked-hash-map` 來維護插入順序。

1. 在你的 `Cargo.toml` 裡加入：
    
    ```toml
    [dependencies]
    linked-hash-map = "0.5"
    ```
    
2. LRU Cache 的定義與實作：
    
    ```rust
    use std::hash::Hash;
    use linked_hash_map::LinkedHashMap;
    
    /// LRU Cache 結構
    pub struct LruCache<K, V> {
        capacity: usize,
        map: LinkedHashMap<K, V>,
    }
    
    impl<K, V> LruCache<K, V>
    where
        K: Eq + Hash + Clone,
    {
        /// 建立一個容量為 capacity 的 LRU Cache
        pub fn new(capacity: usize) -> Self {
            LruCache {
                capacity,
                map: LinkedHashMap::new(),
            }
        }
    
        /// 取得 key 對應的值，若存在則標記為最近使用
        pub fn get(&mut self, key: &K) -> Option<&V> {
            // 先移除再重新插入到尾端，表示「最近使用」
            if let Some(value) = self.map.remove(key) {
                self.map.insert(key.clone(), value);
                self.map.get(key)
            } else {
                None
            }
        }
    
        /// 插入 (key, value)，若已存在就更新並標記最近使用；若超過容量就淘汰最舊的一筆
        pub fn put(&mut self, key: K, value: V) {
            if self.map.contains_key(&key) {
                // 已存在：移除舊值
                self.map.remove(&key);
            } else if self.map.len() == self.capacity {
                // 滿了：淘汰最舊的（front）
                self.map.pop_front();
            }
            // 插入到尾端
            self.map.insert(key, value);
        }
    }
    
    // 測試範例
    #[cfg(test)]
    mod tests {
        use super::LruCache;
    
        #[test]
        fn basic_lru() {
            let mut cache = LruCache::new(2);
            cache.put(1, "one");
            cache.put(2, "two");
            assert_eq!(cache.get(&1), Some(&"one"));
            // 此時存取 1，順序變為 [2, 1]
            cache.put(3, "three"); // 容量為 2，淘汰最舊的 2
            assert_eq!(cache.get(&2), None);
            assert_eq!(cache.get(&3), Some(&"three"));
            cache.put(4, "four"); // 淘汰最舊的 1
            assert_eq!(cache.get(&1), None);
            assert_eq!(cache.get(&3), Some(&"three"));
            assert_eq!(cache.get(&4), Some(&"four"));
        }
    }
    ```
    

**說明：**

- 我們用 `LinkedHashMap` 來同時維護 key→value 的映射與插入／存取順序。
    
- 每次 `get` 都先將該鍵移出再重新插入，以保證它在「最近使用」的位置（尾端）。
    
- 當 `put` 時，如果已經存在就更新並標記為最近使用；若不存在且已滿，就呼叫 `pop_front()` 淘汰最舊的一筆。
    

這樣就完成了一個容量固定、並自動淘汰最久未使用項目的 LRU Cache。