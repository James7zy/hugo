+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'C语言构建面向对象'
+++

C语言构建面向对象：

```
#include <stdio.h>

// 基类
struct Base {
    int x;
    void (*print)(struct Base*);
};

void basePrint(struct Base* self) {
    printf("Base::x = %d\n", self->x);
}

// 派生类
struct Derived {
    struct Base base;  // 派生类包含基类作为成员
    int y;
    void (*print)(struct Derived*);
};

void derivedPrint(struct Derived* self) {
    // 调用基类的打印函数
    self->base.print((struct Base*)self);
    printf("Derived::y = %d\n", self->y);
}

int main() {
    // 创建基类对象
    struct Base base;
    base.x = 10;
    base.print = basePrint;

    // 创建派生类对象
    struct Derived derived;
    derived.base = base;
    derived.y = 20;
    derived.print = derivedPrint;

    // 调用派生类的打印函数
    derived.print(&derived);

    return 0;
}
```
