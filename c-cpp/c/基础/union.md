

##### 实例

假如一个二叉树，只有叶子节点含有数据，非叶子节点只有指向左子树和右子树的指针（不含数据），创建数据结构为：

**第一种:**

```c
struct node_s {
	struct node_s *left;
	struct node_s *right;
	double data[2];
};
```

一共需要32个字节（8 + 8 + 2 * 8），叶子节点会浪费两个指针，非叶子节点会浪费掉两个double；

**升级**

```c
union node_u {
	struct {
		union node_u *left;
		union node_u *right;
	} internal;
	double data[2];
}
```

只使用16个字节：当为叶子节点时，使用data[2]；非叶子节点时，使用 internal；但是使用这种方法无法判断该节点是非叶子节点还是内部节点；

**再升级**

```c
typedef enum {N_LEAF, N_INTERNAL} nodetype_t;
struct node_t {
	nodetype_t type;
	union {
		struct {
			struct node_t* left;
			struct node_t* right;
		} internal;
		double data[2];
	} info;
};
```

共需要24个字节，其中原来的16个字节 + type 类型的4字节 + 填充4字节；



