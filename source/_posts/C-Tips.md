---
title: C++Tips
date: 2020-08-05 14:45:42
tags:
hide: true
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
// 返回一个迭代器，指向键值>= key的第一个元素
// 如果没有找到合适的返回end()
iterator lower_bound(const key_type& key);
// 返回一个迭代器，指向键值> key的第一个元素
// 如果没有找到合适的返回end()
iterator upper_bound(const key_type& key);
vector<int> a{1,2,3,4,5,6,7,8};
auto lowerIt = lower_bound(a.begin(), a.end() ,12);// lowerIt = end()
auto upperIt = upper_bound(a.begin(), a.end() ,12);// upperIt = end()
auto lowerIt = lower_bound(a.begin(), a.end() ,2);// *lowerIt == 2
auto upperIt = upper_bound(a.begin(), a.end() ,2);// *upperIt == 3
```

## 分支预测

gcc 引入，作用是允许将最有可能执行的分支告诉编译器，减少指令跳转带来的性能下降。
这个指令的写法是__builtin_expect(EXP,P)。
一般是将__builtin_expect 封装为likely和unlikely 宏，如下：
```c++
#define likely(x) __builtin_expect(!!(x), 1) //x很可能为真       
#define unlikely(x) __builtin_expect(!!(x), 0) //x很可能为假
```
使用likely()，执行if 后的概率大一些，使用unlikely() 执行else 后面语句的概率大一些。

```c++
// 下面例子编译的指令会预先读取y = 1 这个指令，这是和x  > 0 的概率比较大的时候，如果x > 0 的概率比较小，那么就应该使用 unlikely 这个指令 
int x,y;
if(likely(x > 0)) {
    y = 1;
} else {
    y = -1;
}
```