# 渐进式哈希实现哈希表以及rediscore接口的完善和实现

## 渐进式哈希实现哈希表
> 代码
```java
public class Dict<K,V> {

    private static final int INITIAL_SIZE = 4;
    private static final double LOAD_FACTOR = 0.75;
    private static final int REHASH_MAX_SIZE =5;
    private static final int MAX_EMPTY_PERCENT = 10;


    private DictHashTable<K,V> ht0;
    private DictHashTable<K,V> ht1;
    private int rehashIndex;

    static class DictEntry<K,V>{
        K key;
        V value;
        DictEntry<K,V> next;

        DictEntry(K key, V value){
            this.key = key;
            this.value = value;
        }
    }

    static class DictHashTable<K,V>{
        DictEntry<K,V>[] table;
        int size;
        int sizemask;
        int used;

        @SuppressWarnings("unchecked")
        DictHashTable(int size)
        {
            this.table = (DictEntry<K, V>[]) new DictEntry[size];
            this.size = size;
            this.sizemask = size - 1;
            this.used = 0;
        }
    }

    public Dict(){
        ht0 = new DictHashTable(INITIAL_SIZE);
        ht1 = null;
        rehashIndex = -1;
    }

    private int hash(Object key){
        if(key == null) return 0;
        int h = key.hashCode();
        return h^(h>>>16);
    }

    private int keyIndex(Object key, int size){
        return hash(key) & (size-1);
    }

    public boolean containsKey(K key) {
        if(key == null) return false;
        if(rehashIndex != -1) rehashStep();
        return find(key) != null;
    }

    private DictEntry<K,V> find(K key){
        if(key == null) return null;

        if(rehashIndex != -1) rehashStep();

        int idx = keyIndex(key, ht0.size);
        DictEntry<K,V> entry = ht0.table[idx];
        while(entry != null){
            if(entry.key.equals(key)){
                return entry;
            }
            entry = entry.next;
        }

        if(rehashIndex != -1 || ht1 != null){
            idx = keyIndex(key, ht1.size);
            entry = ht1.table[idx];
            while(entry != null){
                if(entry.key.equals(key)){
                    return entry;
                }
                entry = entry.next;
            }
        }
        return null;
    }

    public V put(K key, V value){
        if(key == null) throw new IllegalArgumentException("key can not be null");
        //如果不在rehash
        if(rehashIndex == -1){
            double loadFactor = (double) ht0.used / ht0.size;
            if(loadFactor > LOAD_FACTOR){
                startRehash(ht0.size);
            }
        }
        //如果在rehash
        if(rehashIndex != -1) rehashStep();

        V oldValue = null;
        DictEntry<K,V> entry = find(key);

        if(entry != null){
            oldValue = entry.value;
            entry.value = value;
            return oldValue;
        }

        int idx;
        if(rehashIndex != -1){
            idx = keyIndex(key, ht1.size);
            DictEntry<K, V> newEntry = new DictEntry<>(key, value);
            newEntry.next = ht1.table[idx];
            ht1.table[idx] = newEntry;
            ht1.used++;
        }
        else{
            idx = keyIndex(key, ht0.size);
            DictEntry<K, V> newEntry = new DictEntry<>(key, value);
            newEntry.next = ht0.table[idx];
            ht0.table[idx] = newEntry;
            ht0.used++;
        }

        return null;
    }

    public V get(K key){
        DictEntry<K, V> entry = find(key);
        return entry!=null?entry.value:null;
    }

    public V remove(K key){
        if(key ==null) return null;
        if(rehashIndex != -1) rehashStep();

        int idx = keyIndex(key, ht0.size);
        DictEntry<K, V> entry = ht0.table[idx];
        DictEntry<K,V> prev = null;

        while(entry != null){
            if(key.equals(entry.key)){
                if(prev ==null){
                    ht0.table[idx] = entry.next;
                }else{
                    prev.next = entry.next;
                }
                ht0.used--;
                return entry.value;
            }
            prev = entry;
            entry = entry.next;
        }


        if(rehashIndex != -1 || ht1 != null){
            idx = keyIndex(key, ht1.size);
            entry = ht1.table[idx];
            prev = null;
            while(entry != null){
                if(key.equals(entry.key)){
                    if(prev ==null){
                        ht1.table[idx] = entry.next;
                    }else{
                        prev.next = entry.next;
                    }
                    ht1.used--;
                    return entry.value;
                }
                prev = entry;
                entry = entry.next;
            }
        }
        return null;
    }

    private void startRehash(int size){
        ht1 = new DictHashTable(size*2);
        rehashIndex =0;
    }

    private void rehashStep(){
        if(rehashIndex ==-1 || ht1 ==null) return;

        int emptyVisited = 0;
        int processed =0;

        while(emptyVisited < MAX_EMPTY_PERCENT
                && processed < REHASH_MAX_SIZE
                && rehashIndex <ht0.size){
            if(ht0.table[rehashIndex] == null){
                rehashIndex++;
                emptyVisited++;
                continue;
            }

            DictEntry<K, V> entry = ht0.table[rehashIndex];
            while(entry != null){
                DictEntry<K,V> next = entry.next;

                int idx = keyIndex(entry.key, ht1.size);
                entry.next = ht1.table[idx]; //头插法
                ht1.table[idx] = entry;
                ht0.used--;
                ht1.used++;

                entry =next;
            }

            ht0.table[rehashIndex] = null;
            rehashIndex++;
            processed++;
        }
        //检查是否已经完成rehash过程
        if(rehashIndex >=  ht0.size){
            ht0 = ht1;
            ht1 =null;
            rehashIndex = -1;
        }
    }

    public Set<K> keySet(){
        Set<K> keys = new HashSet<>();
        if(rehashIndex != -1) rehashStep();

        for(int i =0; i < ht0.size; i++){
            DictEntry<K,V> entry = ht0.table[i];
            while(entry != null){
                keys.add(entry.key);
                entry = entry.next;
            }
        }
        if(rehashIndex != -1 && ht1 !=null){
            for(int i =0; i < ht1.size; i++){
                DictEntry<K,V> entry = ht1.table[i];
                while(entry != null){
                    keys.add(entry.key);
                    entry = entry.next;
                }
            }
        }
        return keys;
    }

    public Map<K,V> getAll(){
        Map<K,V> map = new HashMap<>();

        if(rehashIndex != -1) rehashStep();

        for(int i =0; i < ht0.size; i++){
            DictEntry<K,V> entry = ht0.table[i];
            while(entry != null){
                map.put(entry.key, entry.value);
                entry = entry.next;
            }
        }

        if(rehashIndex != -1 && ht1 !=null){
            for(int i =0; i < ht1.size; i++){
                DictEntry<K,V> entry = ht1.table[i];
                while(entry != null){
                    map.put(entry.key, entry.value);
                    entry = entry.next;
                }
            }
        }

        return map;
    }


    public void clear(){
        ht0 = new DictHashTable<>(INITIAL_SIZE);
        ht1 = null;
        rehashIndex = -1;
    }

    public int size(){
        return ht0.used+(ht1 != null?ht1.used:0);
    }

}
```

## rediscore接口的完善和实现
> rediscore接口的完善
```java
public interface RedisCore {
    Set<byte[]> keys();
    void put(byte[] key, RedisData value);
    RedisData get(byte[] key);
    void selectDB(int dbIndex);
    int getDBNum();
    int getCurrentDBIndex();
}
```
> rediscore接口的实现
```java
public class RedisCoreImpl implements RedisCore{
    private final List<RedisDB> databases;

    private final int dbNum;
    private int currentDBIndex =0;

    public RedisCoreImpl(int dbNum) {
        this.dbNum = dbNum;
        this.databases = new java.util.ArrayList<>(dbNum);
        for(int i =0; i < dbNum; i++){
            databases.add(new RedisDB(i));
        }
    }

    @Override
    public Set<byte[]> keys() {
        int dbIndex = getCurrentDBIndex();
        RedisDB db = databases.get(dbIndex);
        return db.keys();
    }

    @Override
    public void put(byte[] key, RedisData value) {
        RedisDB db = databases.get(getCurrentDBIndex());
        db.put(key, value);
    }

    @Override
    public RedisData get(byte[] key) {
        RedisDB db = databases.get(getCurrentDBIndex());
        if(db.exist(key)){
            return db.get(key);
        }
        return null;
    }


    @Override
    public void selectDB(int dbIndex) {
        if(dbIndex >= 0 && dbIndex < dbNum){
            currentDBIndex = dbIndex;
        }
        else{
            throw new RuntimeException("dbIndex out of range");
        }
    }

    @Override
    public int getDBNum() {
        return dbNum;
    }


    @Override
    public int getCurrentDBIndex() {
        return currentDBIndex;
    }
}
```

## 定义了一个RedisDB类来表示一个Redis数据库
> RedisData是个接口还未实现只是暂时定义
```java
@Setter
@Getter
public class RedisDB {
    private final Dict<byte[], RedisData> data;

    private final int id;

    public RedisDB(int id) {
        this.id = id;
        this.data = new Dict<>();
    }

    public Set<byte[]> keys(){
        return data.keySet();
    }

    public boolean exist(byte[] key){
        return data.containsKey(key);
    }

    public void put(byte[] key, RedisData value){
        data.put(key, value);
    }

    public RedisData get(byte[] key){
        return data.get(key);
    }

    public int size(){
        return data.size();
    }


}
```