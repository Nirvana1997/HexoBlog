---
title: c++中unordered_map的实现
category:
  - category
tags:
  - tag
date: 2018-10-29 15:14:34
---

##1.前言

写leetcode时发现很多时候高效的答案会用到unordered_map和unordered_set.从使用角度看,unordered的stl容器与正常的map,set相比,在插入和查询的效率上更高,[《boost::unordered_map 和 std::map 的对比》](https://blog.csdn.net/ljp1919/article/details/50463761?utm_source=blogkpcl7)中用5000万条数据放入两个容器中进行了对比.这是因为实现上,unordered的stl容器使用的是hashtable,而map,set使用的是红黑树.当然map,set插入后天然有序,需要按key排序时会比较方便.

正好面试要求让我写hashtable的实现,今天就研究一下c++中unordered_map的实现,主要参照的是linux下的源码:/usr/include/c++/4.8.2/profile/unordered_map和hashtable(hashtable我在我的c++源码里没找到,可能在编译好的库中了,所以找的网上的代码).

<!-- more -->

##2.源代码

###i.模板定义

以下是map和unordered_map的模板定义:

```c++
template<typename _Key, typename _Tp, typename _Compare = std::less<_Key>,
	 typename _Allocator = std::allocator<std::pair<const _Key, _Tp> > >
class map
```

```c++
template<typename _Key, typename _Tp,
	 typename _Hash = std::hash<_Key>,
	 typename _Pred = std::equal_to<_Key>,
	 typename _Alloc = std::allocator<std::pair<const _Key, _Tp> > >
class unordered_map
```

可以看到map需要key实现比较函数,默认使用std的less函数,所以可以通过重载operator<()实现,unordered_map需要key实现hash函数和equal_to函数,hash可以注入std,equal_to可以重载operator==().从这个模板的定义也可以大概了解实现的数据结构.

### ii.operator[]

```c++
template<typename K, typename Pair, typename Hashtable>
    typename map_base<K, Pair, extract1st<Pair>, true, Hashtable>::mapped_type&
    map_base<K, Pair, extract1st<Pair>, true, Hashtable>::
    operator[](const K& k)
    {
      Hashtable* h = static_cast<Hashtable*>(this);
      typename Hashtable::hash_code_t code = h->m_hash_code(k);
      std::size_t n = h->bucket_index(k, code, h->bucket_count());
 
      typename Hashtable::node* p = h->m_find_node(h->m_buckets[n], k, code);
      if (!p)
	return h->m_insert_bucket(std::make_pair(k, mapped_type()),
				  n, code)->second;
      return (p->m_v).second;
    }
```

可以看到主要还是依靠Hashtable实现的.

步骤是:

1. 计算key的hash_code

2. 找到对应的bucket
3. 在bucket中寻找key值对应的node
4. 若存在对应node,则返回node对应值,若不存在,则插入一个新节点,返回新节点的值(对应类型默认值)

