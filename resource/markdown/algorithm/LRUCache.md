### LRUCache 缓存算法

##### 最少最近使用算法

```java
public class LRUCache<K, V> {
    /**
     * 默认容量
     */
    private static final int DEFAULT_CAPACITY = 1024;
    /**
     * 缓存容量
     */
    private int capacity;
    /**
     * 实际存储/使用的元素大小
     */
    private int size = 0;
    /**
     * 高效访问的散列表
     */
    private Map<K, Node<K, V>> map;

    private Node<K, V> head, tail;

    /**
     * 自定义双向链表中的节点
     */
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;

        Node(Node<K, V> prev, K key, V value, Node<K, V> next) {
            this.key = key;
            this.value = value;
            this.prev = prev;
            this.next = next;
        }
    }

    public LRUCache() {
        this.capacity = DEFAULT_CAPACITY;
        this.map = new HashMap<>(DEFAULT_CAPACITY, 0.75F);
        this.head = null;
        this.tail = null;
    }

    public LRUCache(int capacity) {
        if (capacity <= 0) {
            throw new IllegalArgumentException("Capacity must be positive integer");
        }

        this.capacity = capacity;
        this.map = new HashMap<>(capacity, 0.75F);
        this.head = null;
        this.tail = null;
    }

    public V get(K key) {
        Node<K, V> node = this.map.get(key);
        if (node != null) {
            this.moveToHead(node);
            return node.value;
        } else {
            return null;
        }
    }

    public V put(K key, V value) {
        Node<K, V> node = this.map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
            return value;
        }

        if (size == capacity) {
            node = removeLast();
            map.remove(node.key);
        }

        node = addFirst(key, value);
        map.put(key, node);

        return value;
    }

    /**
     * 对于新添加的元素，应将新元素添加到链表的头部
     */
    private Node<K, V> addFirst(K key, V value) {
        final Node<K, V> h = head;
        final Node<K, V> newNode = new Node<>(null, key, value, h);
        head = newNode;
        if (h == null) {
            tail = newNode;
        } else {
            h.prev = newNode;
        }
        size++;

        return newNode;
    }

    /**
     * 对于被访问的元素，将该元素移动到头部
     */
    private Node<K, V> moveToHead(Node<K, V> node) {
        final Node<K, V> prev = node.prev;
        final Node<K, V> next = node.next;

        if (prev == null) { // 如果是首节点，无需移动
            return node;
        }

        prev.next = next;
        if (next == null) { // 如果是尾节点，需要移动tail
            tail = prev;
        } else {
            next.prev = prev;
        }

        node.prev = null;
        node.next = head;
        head.prev = node;
        head = node;

        return node;
    }

    /**
     * 缓存满时，应删除（淘汰）最后一个节点
     */
    private Node<K, V> removeLast() {
        final Node<K, V> t = tail;
        if (t == null) {
            return null;
        }

        t.value = null; // help GC
        Node<K, V> prev = t.prev;
        t.prev = null; // help GC
        tail = prev; // 移动 tail
        if (prev == null) { // 如果尾节点的前一个节点也为空，说明尾节点也是首节点
            head = null;
        } else {
            prev.next = null;
        }
        size--;
        return t;
    }
}
```



