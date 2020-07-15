
```
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 〈〉
 *
 * @author binbin
 * @create 2019/8/26
 */
public class LRU<K,V> {
    /**
     * LinkedHashMap负载因子默认0.75
     */
    private static final float hashLoadFactory = 0.75f;
    private LinkedHashMap<K, V> map;
    private int cacheSize;

    /**
     * @param cacheSize 容量
     */
    public LRU(int cacheSize) {
        this.cacheSize = cacheSize;
        int capacity = (int) Math.ceil(cacheSize / hashLoadFactory) + 1;
        map = new LinkedHashMap<K, V>(capacity, hashLoadFactory, true) {
            /*
             * 当map容量大于LRU的容量时，删除最近最少使用的数据
             */
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRU.this.cacheSize;
            }
        };
    }

    public synchronized V get(K key) {
        return map.get(key);
    }

    public synchronized void put(K key, V value) {
        map.put(key, value);
    }

    public synchronized void clear() {
        map.clear();
    }

    public synchronized int usedSize() {
        return map.size();
    }

    public void print() {
        for (Map.Entry<K, V> entry : map.entrySet()) {
            System.out.print(entry.getValue() + "--");
        }
        System.out.println();
    }


    public static void main(String[] args) {
        LRU<String,Integer> lru=new LRU<>(3);
        for(int i=0;i<40;i++) {
            lru.put(i+"", i);
            lru.print();
        }

    }


}

```
