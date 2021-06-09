---
title: c++中unordered_map的实现
category:
  - c++
tags:
  - c++
  - stl
date: 2018-10-29 15:14:34
---

写leetcode时发现很多时候高效的答案会用到unordered_map和unordered_set.从使用角度看,unordered的stl容器与正常的map,set相比,在插入和查询的效率上更高,[《boost::unordered_map 和 std::map 的对比》](https://blog.csdn.net/ljp1919/article/details/50463761?utm_source=blogkpcl7)中用5000万条数据放入两个容器中进行了对比.这是因为实现上,unordered的stl容器使用的是hashtable,而map,set使用的是红黑树.当然map,set插入后天然有序,需要按key排序时会比较方便.

正好面试要求让我写hashtable的实现,今天就研究一下c++中unordered_map的实现,主要参照的是linux下的源码:/usr/include/c++/4.8.2/profile/unordered_map和hashtable(hashtable我在我的c++源码里没找到,可能在编译好的库中了,所以找的网上的代码).

<!-- more -->

## 1.源代码

### i.模板定义

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

### iii.hash

hashtable一个很重要的影响性能的因素是冲突率,即一个桶中的元素个数.而决定冲突率的则是hash函数和桶的个数,所以hash函数的定义和rehash的时机格外重要.

```c++
inline std::pair<bool, std::size_t>
  prime_rehash_policy::
  need_rehash(std::size_t n_bkt, std::size_t n_elt, std::size_t n_ins) const
  {
    if (n_elt + n_ins > m_next_resize)
      {
	float min_bkts = (float(n_ins) + float(n_elt)) / m_max_load_factor;
	if (min_bkts > n_bkt)
	  {
	    min_bkts = std::max(min_bkts, m_growth_factor * n_bkt);
	    const unsigned long* const last = X<>::primes + X<>::n_primes;
	    const unsigned long* p = std::lower_bound(X<>::primes, last,
						      min_bkts, lt());
	    m_next_resize =
	      static_cast<std::size_t>(std::ceil(*p * m_max_load_factor));
	    return std::make_pair(true, *p);
	  }
	else
	  {
	    m_next_resize =
	      static_cast<std::size_t>(std::ceil(n_bkt * m_max_load_factor));
	    return std::make_pair(false, 0);
	  }
      }
    else
      return std::make_pair(false, 0);
  }
```

以上是need_rehash函数,判断是否需要rehash的主要是依靠m_max_load_factor,这是元素个数和桶的个数的比例,默认为1.0,若桶数目过少hashtable便会进行一次rehash.

```c++
template<typename K, typename V,
	   typename A, typename Ex, typename Eq,
	   typename H1, typename H2, typename H, typename RP,
	   bool c, bool ci, bool u>
    void
    hashtable<K, V, A, Ex, Eq, H1, H2, H, RP, c, ci, u>::
    m_rehash(size_type n)
    {
      node** new_array = m_allocate_buckets(n);
      try
	{
	  for (size_type i = 0; i < m_bucket_count; ++i)
	    while (node* p = m_buckets[i])
	      {
		size_type new_index = this->bucket_index(p, n);
		m_buckets[i] = p->m_next;
		p->m_next = new_array[new_index];
		new_array[new_index] = p;
	      }
	  m_deallocate_buckets(m_buckets, m_bucket_count);
	  m_bucket_count = n;
	  m_buckets = new_array;
	}
      catch(...)
	{
	  // A failure here means that a hash function threw an exception.
	  // We can't restore the previous state without calling the hash
	  // function again, so the only sensible recovery is to delete
	  // everything.
	  m_deallocate_nodes(new_array, n);
	  m_deallocate_buckets(new_array, n);
	  m_deallocate_nodes(m_buckets, m_bucket_count);
	  m_element_count = 0;
	  __throw_exception_again;
	}
    }
```

接下来是rehash函数,rehash其实就是新开辟一块空间,重进计算hash将元素放入,然后释放原来的空间,很好理解.

```c++
template<>
    struct Fnv_hash<8>
    {
      static std::size_t
      hash(const char* first, std::size_t length)
      {
	std::size_t result = static_cast<std::size_t>(14695981039346656037ULL);
	for (; length > 0; --length)
	  {
	    result ^= (std::size_t)*first++;
	    result *= 1099511628211ULL;
	  }
	return result;
      }
    };
```

最后是hash函数,这里的hash函数是std::string的.

## 2.总结

c++底层实现的hashtable和基础的数据结构还是很一致的,但细节上很多是经过考量的,比如hash函数,有很多版本.网上看到:

>FNV 有分版本，例如 FNV-1 和 FNV-1a，区别其实就是先异或再乘，或者先乘在异或，这里用的是 FNV-1a，为什么呢，维基里面说，The small change in order leads to much better avalanche characteristics，什么叫 avalanche characteristics 呢，这个是个密码学术语，叫雪崩效应，意思是说输入的一个非常微小的改动，也会使最终的 hash 结果发生非常巨大的变化，这样的哈希效果被认为是更好的。

所以光知道数据结构,在实现时不注重细节处理也很容易造成很多问题.
