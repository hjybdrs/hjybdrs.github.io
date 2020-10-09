---
title: SwitchRemove
date: 2020-10-09 19:53:08
tags:
---

提高switch 代码的可读性和扩展性
<!-- more -->

# 如何干掉又臭又长的switch代码

## 提取法

将case 中的每一个代码块全部抽取为单独的一个函数，但是违背了单一职责原则，多个case 就表明职责的多样性
## 跳表法

```c++
typedef void (*function)(int param);
// 存储函数指针，有点类似于nginx 初始化module数组 一样
DataProcessor processor[20]={&Kill, &Run, &Jump}
```
## 多态+工厂模式

### 创建一个基类

```c++
class DataProcessor{

public:
    virtual void Process(int param) = 0;

};
```

### 每一个case 实现一个类

```c++

class Kill : public DataProcessor {

public:
    void Process(int param) override {
        // something
    }

};

```

### 实现工厂函数

```c++
DataProcessor* GetProcessor(int param) {
    switch(param) {
        case Kill:
            return new Kill();
        case Jump:
            return new Jump();
        case Run:
            return new Run();
    }
    return nullptr;
}
```