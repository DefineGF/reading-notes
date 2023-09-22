

### 类结构

#### BPlusTreePage

作为 B+树页面的头部分、：

##### 属性

- page_type：页类型：内部 & 叶子
- lsn_：日志序列号
- size_：页面中 K & V 个数
- max_size：页面 K & V 容量
- parent_page_id：父page id
- page_id_：自身id

每个属性4个字节，因此 Page 的 Header format 为 24 字节！





#### BPlusTreeLeafPage : public BPlusTreePage

叶子页面的格式：

> | HEADER | KEY(1) + RID(1) | KEY(2) + RID(2) | ... | KEY(n) + RID(n)



##### 新增属性

- page_id_t next_page_id_：下一个叶子节点的id

共28B！



##### MappingType array_[1]：

- #define MappingType std::pair<KeyType, ValueType>



### 插入节点

#### BPlusTree

(注意与page中的 BPlusTreePage 和 BPlusTreeLeafPage 进行区分)



针对B+树为空，即新插入Key - Value 时，调用函 数：

BPlusTree.StartNewTree

- BufferPoolManagerInstance -> NewPage  // 第一个节点创建的是 叶子节点
- leaf_page -> Init()
- leaf_page -> Insert()

##### void BPLUSTREE_TYPE::StartNewTree(const KeyType &key, const ValueType &value)

```cpp
#define BPLUSTREE_TYPE BPlusTree<KeyType, ValueType, KeyComparator>

void BPLUSTREE_TYPE::StartNewTree(const KeyType &key, const ValueType &value) {
  Page *new_page = buffer_pool_manager_->NewPage(&root_page_id_);
  if (new_page == nullptr) {
    throw "out of memory";
  }

  LeafPage *leaf_page = reinterpret_cast<LeafPage *>(new_page);
  leaf_page->Init(root_page_id_,INVALID_PAGE_ID,leaf_max_size_);

  //update root page id
  UpdateRootPageId(1);
  // Insert entry directly into leaf page.
  // For a new B+ tree, the root page is the leaf page.
  leaf_page->Insert(key, value, comparator_);
  buffer_pool_manager_->UnpinPage(root_page_id_, true);  // unpin
}
```



#### BPlusTreeLeafPage

> \#define INDEX_TEMPLATE_ARGUMENTS template <typename KeyType, typename ValueType, typename KeyComparator>
>
> \#define B_PLUS_TREE_LEAF_PAGE_TYPE BPlusTreeLeafPage<KeyType, ValueType, KeyComparator>



##### KeyIndex(const KeyType &key, const KeyComparator &comparator) const -> int

```cpp
auto B_PLUS_TREE_LEAF_PAGE_TYPE::KeyIndex(const KeyType &key, const KeyComparator &comparator) const -> int {
  auto iter = std::lower_bound(array_, array_ + GetSize(), key,
                               [&comparator](const auto &pair1, auto key) 
                               { return comparator(pair1.first, key) < 0; });
  return std::distance(array_, iter);
}
```

- std::lower_bound: 返回第一个大于等于 key 的迭代器；
- std::distance：返回相较于查找起始点的索引；



##### Insert(const KeyType &key, const ValueType &value, const KeyComparator &comparator) -> int

在 叶子节点插入

```cpp
INDEX_TEMPLATE_ARGUMENTS
auto B_PLUS_TREE_LEAF_PAGE_TYPE::Insert(const KeyType &key, const ValueType &value, const KeyComparator &comparator)
    -> int {
  int index = KeyIndex(key, comparator);
  if (index == GetSize()) {
    array_[index] = {key, value};
    IncreaseSize(1);
    return GetSize();
  }
  if (comparator(array_[index].first, key) == 0) {
    return GetSize();
  }
  std::move_backward(array_ + index, array_ + GetSize(), array_ + GetSize() + 1);
  array_[index] = {key, value};
  IncreaseSize(1);
  return GetSize();
}
```

1. 如果插入目标的下标索引 == GetSize() (包括 size == 0)，则直接插入；
2. 如果当前key等于已有key，返回size；
3. 其他情况，即插入数组内部，则需要将元素右移一位再插入；





#### 添加后续节点

首先是找到需要插入的叶子节点；

##### Page *BPLUSTREE_TYPE::FindLeafPage(const KeyType &key, bool leftMost)

```cpp
Page *BPLUSTREE_TYPE::FindLeafPage(const KeyType &key, bool leftMost) {
  if (root_page_id_ == INVALID_PAGE_ID) {
    throw std::runtime_error("Unexpected. root_page_id is INVALID_PAGE_ID");
  }
  Page *page = buffer_pool_manager_->FetchPage(root_page_id_);  // now root page is pin
  BPlusTreePage *node = reinterpret_cast<BPlusTreePage *>(page);
  while (!node->IsLeafPage()) {
    InternalPage *internal_node = reinterpret_cast<InternalPage *>(node);
    page_id_t next_page_id = leftMost ? internal_node->ValueAt(0) : internal_node->Lookup(key, comparator_);
      
    Page *next_page = buffer_pool_manager_->FetchPage(next_page_id);  // next_level_page pinned
    BPlusTreePage *next_node = reinterpret_cast<BPlusTreePage *>(next_page);
    buffer_pool_manager_->UnpinPage(node->GetPageId(), false);  // curr_node unpinned
    page = next_page;
    node = next_node;
  }
  return page;
}
```



其中 LookUp 详细如下：即从当前内部节点中，找到下一 page_id

```cpp
ValueType B_PLUS_TREE_INTERNAL_PAGE_TYPE::Lookup(const KeyType &key, const KeyComparator &comparator) const {
  // 从第二个节点开始
  for (int i = 1; i < GetSize(); i++) {
    KeyType cur_key = array_[i].first;
    if (comparator(key, cur_key) < 0) {  // key < cur_key
      return array_[i - 1].second;
    }
  }
  return array_[GetSize() - 1].second;
}
```

