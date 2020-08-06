---
title: C++Tips
date: 2020-08-05 14:45:42
tags:
---

# Tips

## 移位操作

```c++
// 需要移位的数字 << 移位的次数n
// 4*2^2 = 16
4 << 2 
```

## STL

### lower_bound upper_bound

```c++
iterator lower_bound(const key_type& key);//返回一个迭代器，指向键值>=key的第一个元素
iterator upper_bound(const key_type& key);//返回一个迭代器，指向键值=key的第一个元素
```