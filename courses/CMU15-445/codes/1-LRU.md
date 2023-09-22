##### Replace

- virtual auto Victim(frame_id_t *frame_id) -> bool = 0：移除
- virtual void Pin(frame_id_t frame_id) = 0
- virtual void Unpin(frame_id_t frame_id) = 0
- virtual auto Size() -> size_t = 0



#### LRUReplacer : public Replacer

##### 成员变量

```cpp
size_t capcity;
size_t size;
std::list<frame_id_t> lists;
std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> hash;
std::mutex mtx;

// using frame_id_t = int32_t;    // frame id type
```



##### 成员函数

```cpp
auto LRUReplacer::Victim(frame_id_t *frame_id) -> bool 
{
    if (lists.empty()) return false;
    mtx.lock();
    frame_id_t last_frame = lists.back();
    lists.pop_back();
    hash.erase(last_frame);
    mtx.unlock();
    *frame_id = last_frame;
    return true;
}
```

1. 删除列表尾部页id；
2. 删除hash表中对应id的页；



```cpp
void LRUReplacer::Pin(frame_id_t frame_id) 
{
    if (!hash.count(frame_id)) return;
    mtx.lock();
    lists.erase(hash[frame_id]);
    hash.erase(frame_id);
    mtx.unlock();
}

void LRUReplacer::Unpin(frame_id_t frame_id) 
{
    if (hash.count(frame_id)) return;
    mtx.lock();
    if (lists.size() == capcity)
        lists.pop_back();
    lists.push_front(frame_id);
    std::list<frame_id_t>::iterator it = lists.begin();
    hash.insert({frame_id, it});
    mtx.unlock();
}
```

针对 Pin和 UnPin：

当有页被线程持有并访问， 无法被LRU淘汰时，需要将frame_id 从 list 和 map中删除记录（内存中依然存在该页），只是再不会被LRU淘汰了；

相对的，UnPin只是将frame_id 恢复回来。



