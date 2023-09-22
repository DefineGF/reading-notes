详细教程：https://www.cnblogs.com/JayL-zxl/p/14311883.html

### DiskManager

##### ReadPage(page_id_t page_id, char *page_data)

```cpp
void DiskManager::ReadPage(page_id_t page_id, char *page_data) {
  int offset = page_id * PAGE_SIZE; //PAGE_SIZE=4kb 先计算偏移。判断是否越界（因为文件大小有限制）
  if (offset > GetFileSize(file_name_)) {
    LOG_DEBUG("I/O error reading past end of file");
    // std::cerr << "I/O error while reading" << std::endl;
  } else {
    // set read cursor to offset
    db_io_.seekp(offset); 			   //把读写位置移动到偏移位置处
    db_io_.read(page_data, PAGE_SIZE); //把数据读到page_data中
    if (db_io_.bad()) {
      LOG_DEBUG("I/O error while reading");
      return;
    }
    // if file ends before reading PAGE_SIZE
    int read_count = db_io_.gcount();
    if (read_count < PAGE_SIZE) {
      LOG_DEBUG("Read less than a page");
      db_io_.clear();
      // std::cerr << "Read less than a page" << std::endl;
      memset(page_data + read_count, 0, PAGE_SIZE - read_count); //如果读取的数据小于4kb剩下的补0
    }
  }
}

// 获取文件大小
auto DiskManager::GetFileSize(const std::string &file_name) -> int {
  struct stat stat_buf;
  int rc = stat(file_name.c_str(), &stat_buf);
  return rc == 0 ? static_cast<int>(stat_buf.st_size) : -1;
}

struct stat
{
    _dev_t         st_dev;
    _ino_t         st_ino;
    unsigned short st_mode;
    short          st_nlink;
    short          st_uid;
    short          st_gid;
    _dev_t         st_rdev;
    _off_t         st_size;
    time_t         st_atime;
    time_t         st_mtime;
    time_t         st_ctime;
};
```



##### WritePage(page_id_t page_id, const char *page_data)

```cpp
void DiskManager::WritePage(page_id_t page_id, const char *page_data) {
  size_t offset = static_cast<size_t>(page_id) * PAGE_SIZE; //先计算偏移
  // set write cursor to offset
  num_writes_ += 1; //记录写的次数
  db_io_.seekp(offset);
  db_io_.write(page_data, PAGE_SIZE); //向offset处写data
  // check for I/O error
  if (db_io_.bad()) {
    LOG_DEBUG("I/O error while writing");
    return;
  }
  // needs to flush to keep disk file in sync
  db_io_.flush(); //刷新缓冲区
}
```





### BufferPoolManager

#### LRU

##### 成员变量

```cpp
std::mutex latch;  // thread safety
int capacity;      // max number of pages LRUReplacer can handle
std::list<frame_id_t> lru_list;
std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> lruMap;
```



##### 牺牲页

```cpp
bool LRUReplacer::Victim(frame_id_t *frame_id) {
  // 选择一个牺牲frame
  latch.lock();
  if (lruMap.empty()) {
    latch.unlock();
    return false;
  }

  // 选择列表尾部 也就是最少使用的frame
  frame_id_t lru_frame = lru_list.back();
  lruMap.erase(lru_frame);
  // 列表删除
  lru_list.pop_back();
  *frame_id = lru_frame;
  latch.unlock();
  return true;
}
```



##### 避免替换资格

```cpp
void LRUReplacer::Pin(frame_id_t frame_id) {
  // 被引用的frame 不能出现在lru list中
  latch.lock();

  if (lruMap.count(frame_id) != 0) {
    lru_list.erase(lruMap[frame_id]);
    lruMap.erase(frame_id);
  }

  latch.unlock();
}
```



##### 恢复替换资格

```cpp
void LRUReplacer::Unpin(frame_id_t frame_id) {
  // 加入lru list中
  latch.lock();
  if (lruMap.count(frame_id) != 0) {
    latch.unlock();
    return;
  }
  // if list size >= capacity
  // while {delete front}
  while (Size() >= capacity) {
     frame_id_t need_del = lru_list.front();
      lru_list.pop_front();
      lruMap.erase(need_del);
  }
  // insert
  lru_list.push_front(frame_id);
  lruMap[frame_id] = lru_list.begin();
  latch.unlock();
}
```



#### Buffer Pool Manager

page：

- 作为内存中的一段，用以保存从磁盘中读取的物理页；
- 整个生命周期中，可能保存不同的物理页；
- 标识page的page_id 用来跟踪真实的物理页面地址；如果page 不包含 物理页面，则page_id 设置为 INVALID_PAGE_ID；

除此之外（关联页表）：

- 维护 pin 计数器，用以表示固定该页面的线程数；
- 脏标记；



#### 实现

##### find_replace: 寻找空白 frame_id

- 首先通过空闲列表 free_list 来判断是否有空白的 Page ，如果有，直接返回相应的frame_id;
- 如果没有，则通过 LRU 找到要替换的页；

```cpp
bool BufferPoolManager::find_replace(frame_id_t *frame_id) {
  // if free_list not empty then we don't need replace page
  // return directly
  if (!free_list_.empty()) {
    *frame_id = free_list_.front();
    free_list_.pop_front();
    return true;
  }
  
  if (replacer_->Victim(frame_id)) {
    int replace_frame_id = -1;
    for (const auto &p : page_table_) {
      page_id_t pid = p.first;
      frame_id_t fid = p.second;
      if (fid == *frame_id) {
        replace_frame_id = pid;
        break;
      }
    }
    
    if (replace_frame_id != -1) {
      Page *replace_page = &pages_[*frame_id];

      // If dirty, flush to disk
      if (replace_page->is_dirty_) {
        char *data = pages_[page_table_[replace_page->page_id_]].data_;
        disk_manager_->WritePage(replace_page->page_id_, data);
        replace_page->pin_count_ = 0;  // Reset pin_count
      }
      page_table_.erase(replace_page->page_id_);
    }

    return true;
  }
  return false;
}
```

- 如果空闲链表（`std::list<frame_id_t> free_list_;`）非空，则直接取出首个 frame_id 即可；
- 否则，执行LRU算法：
  - 调用  Victim 得到将要牺牲的 frame_id；
  - 在 pages_ (`Page *pages_`) 找到对应的牺牲页，如果为dirty则写回磁盘，同时reset pin count；
  - 同时删除 page_table （`std::unordered_map<page_id_t, frame_id_t> page_table_`） 中删除 page_id <-> frame_id 映射关系；



##### FetchPageImpl(page_id_t page_id) 实现

实现过程：

1. 首先通过 page_table 判断指定的 page_id 是否在缓冲池中；如果存在，则获取指向的 Page，然后增加 pin次数；
2. 如果不存在， 通过 find_replace 获取相应的页 frame_id：
   - 根据 frame_id 获取到相应的Page；如果该Page是dirty，则写回磁盘；
   - 然后释放 page_table 中的 该frame_id对应的 Page 中对应 page_id 的映射关系；同时重新设置新的 page_id 与 空白 Page 的映射关系；
   - 然后根据page_id 将磁盘页读入对应 Page 中的 data_ 指向的内存；
   - 最后初始化 Page 的相关内容：dirty、pin、page_id 等等；



代码如下：

```cpp
Page *BufferPoolManager::FetchPageImpl(page_id_t page_id) {
  latch_.lock();
  std::unordered_map<page_id_t, frame_id_t>::iterator it = page_table_.find(page_id);

  if (it != page_table_.end()) {
    frame_id_t frame_id = it->second;
    Page *page = &pages_[frame_id];

    page->pin_count_++;        // pin the page
    replacer_->Pin(frame_id);  // notify replacer

    latch_.unlock();
    return page;
  }

  frame_id_t replace_fid;
  if (!find_replace(&replace_fid)) {
    latch_.unlock();
    return nullptr;
  }
  Page *replacePage = &pages_[replace_fid];

  if (replacePage->IsDirty()) {
    disk_manager_->WritePage(replacePage->page_id_, replacePage->data_);
  }
  page_table_.erase(replacePage->page_id_);

  page_table_[page_id] = replace_fid;
    
  Page *newPage = replacePage;
  disk_manager_->ReadPage(page_id, newPage->data_);
  newPage->page_id_ = page_id;
  newPage->pin_count_++;
  newPage->is_dirty_ = false;
  replacer_->Pin(replace_fid);
  latch_.unlock();

  return newPage;    
}
```



##### UnpinPageImpl(page_id_t page_id, bool is_dirty) ：恢复替换资格

算法思想：

- 如果缓冲池中没有对应的页，直接返回；
- 否则，找到对应的Page：
  - 设置脏位；
  - 如果 pin > 0，直接 pin--; 如果 pin  == 0，直接恢复被替换的资格，详情为 LRU 中的 unpin



```cpp
bool BufferPoolManager::UnpinPageImpl(page_id_t page_id, bool is_dirty) {
  latch_.lock();
  // 1. 如果page_table中就没有
  auto iter = page_table_.find(page_id);
  if (iter == page_table_.end()) {
    latch_.unlock();
    return false;
  }
  // 2. 找到要被unpin的page
  frame_id_t unpinned_Fid = iter->second;
  Page *unpinned_page = &pages_[unpinned_Fid];
  if (is_dirty) {
    unpinned_page->is_dirty_ = true;
  }
  // if page的pin_count == 0 则直接return
  if (unpinned_page->pin_count_ == 0) {
    latch_.unlock();
    return false;
  }
  unpinned_page->pin_count_--;
  if (unpinned_page->GetPinCount() == 0) {
    replacer_->Unpin(unpinned_Fid);
  }
  latch_.unlock();
  return true;
}
```



##### Page* NewPageImpl(page_id_t *page_id)

算法思想：

1. 首先判断缓冲池中的 Page 的 pin 是否都是 0，如果都不等于 0，表明没有可以替换出去的页（都在被访问中）
2. 通过 find_replace 找到空闲页（或者说是替换页）对应的 frame_id；
3. 完成frame_id 和 page_id 之间的映射；
4. 将新分配的页写入磁盘中！

> 注意：如果不将新分配的页写入磁盘，那么由于其他情况 当前Page 被 LRU 从buffer_pool 中清除的话，重新从磁盘中 fetch 会找不到对应的磁盘页

```cpp
Page *BufferPoolManager::NewPageImpl(page_id_t *page_id) {
  latch_.lock();

  page_id_t new_page_id = disk_manager_->AllocatePage();

  bool is_all = true;
  for (int i = 0; i < static_cast<int>(pool_size_); i++) {
    if (pages_[i].pin_count_ == 0) {
      is_all = false;
      break;
    }
  }
  if (is_all) {
    latch_.unlock();
    return nullptr;
  }
  // 2.
  frame_id_t victim_fid;
  if (!find_replace(&victim_fid)) {
    latch_.unlock();
    return nullptr;
  }
  // 3.
  Page *victim_page = &pages_[victim_fid];
  victim_page->page_id_ = new_page_id;
  victim_page->pin_count_++;
  replacer_->Pin(victim_fid);
  page_table_[new_page_id] = victim_fid;
  victim_page->is_dirty_ = false;
  *page_id = new_page_id;

  // 注意：
  disk_manager_->WritePage(victim_page->GetPageId(), victim_page->GetData());
  latch_.unlock();
  return victim_page;
}
```



##### bool BufferPoolManager::DeletePageImpl(page_id_t page_id)

算法思想：

1. 如果页表中没有当前 page_id， 直接返回；
2. 当前页对应的pin > 0， 表示不能删除（其他占用）；
3. 当前页被修改，则写回磁盘；
4. 最后：
   - 释放页内存；
   - 恢复页成员变量；
   - 并将frame_id 添加到空闲列表后；

```cpp
bool BufferPoolManager::DeletePageImpl(page_id_t page_id) {
  latch_.lock();

  // 1.
  if (page_table_.find(page_id) == page_table_.end()) {
    latch_.unlock();
    return true;
  }
  // 2.
  frame_id_t frame_id = page_table_[page_id];
  Page *page = &pages_[frame_id];
  if (page->pin_count_ > 0) {
    latch_.unlock();
    return false;
  }
  if (page->is_dirty_) {
    FlushPageImpl(page_id);
  }
  // delete in disk in here
  disk_manager_->DeallocatePage(page_id);
  
  page_table_.erase(page_id);
  // reset metadata
  page->is_dirty_ = false;
  page->pin_count_ = 0;
  page->page_id_ = INVALID_PAGE_ID;

  
  free_list_.push_back(frame_id);
  latch_.unlock();
  return true;
}
```



