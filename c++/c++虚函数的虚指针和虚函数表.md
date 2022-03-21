c++为了实现对象在运行时的动态调用函数使用了一种虚函数(vtable)的技术。
虚函数表是编译器在编译阶段为类分配的静态数组，每一个使用了虚函数的类都有自己的虚函数表，表中的存放的都是函数指针，指向该类访问的虚函数。
同时，编译器还是添加一个指向vtable的指针，称之为vptr，vptr在创建实例时自动设置。vptr的设定和重置都由每个类的构造、析构和拷贝赋值运算符自动完成。

```c++
#include <iostream>
using namespace std;

typedef void (*Fun)();
class Base {
public:
  Base() {}
  virtual void fun1() { cout << "Base : fun1()" << endl; }
  virtual void fun2() { cout << "Base : fun2()" << endl; }
  virtual void fun3() { cout << "Base : fun3()" << endl; }
  ~Base() {}
};

/**
 * @brief
 * 获取vptr地址与func地址,vptr指向的是一块内存，这块内存存放的是虚函数地址，这块内存就是我们所说的虚表
 *
 */
class Derived : public Base {
public:
  Derived() {}
  void fun1() { cout << "Derived : fun1()" << endl; }
  void fun2() { cout << "Derived : fun2()" << endl; }
  void fun3() { cout << "Derived : fun3()" << endl; }
  void fun4() { cout << "Derived : fnu4()" << endl; }
  ~Derived() {}
};

Fun getAddr(void *obj, unsigned int offset) {
  cout << "====================" << endl;
  void *vptr_addr =
      (void *)*(unsigned long *)obj; // 64位操作系统，占8字节，通过*(unsigned
                                     // long *)obj取出前8字节，即vptr指针
  printf("vptr_addr:%p\n", vptr_addr);
  /**
   * @brief 通过vptr指针访问virtual table，
   * 因为虚表中每个元素(虚函数指针)在64位编译器下是8个字节，因此通过*(unsigned
   * long*)vptr_addr取出前8个字节，后面加上偏移量就是每个函数的地址！
   *
   */
  void *func_addr = (void *)*((unsigned long *)vptr_addr + offset);
  printf("func_addr:%p\n", func_addr);
  return (Fun)func_addr;
}

int main(void) {
  Base ptr;
  Derived d;
  Base *pt = new Derived();
  Base &pp = ptr;
  Base &p = d;
  cout << "基类对象直接调用" << endl;
  ptr.fun1();
  cout << "基类引用指向基类实例" << endl;
  pp.fun1();
  cout << "基类指针指向派生类实例并调用虚函数" << endl;
  pt->fun1();
  cout << "基类引用指向派生类实例并调用虚函数" << endl;
  p.fun1();

  /* 手动查找vptr和vtable */
  Fun f1 = getAddr(pt, 0);
  (*f1)();
  Fun f2 = getAddr(pt, 1);
  (*f2)();
  delete pt;
  Base *pt1 = new Derived(); // 基类指针指向派生类实例
  cout << "基类指针指向派生类实例并调用虚函数" << endl;
  pt1->fun1();
  return 0;
}
```

在getAddr函数中断点调试可以看到：
```c++
(gdb) p obj
$8 = (void *) 0x55555556aeb0
(gdb) x 0x55555556aeb0
0x55555556aeb0: 0x0000555555557cf0
(gdb) x 0x0000555555557cf0
0x555555557cf0 <_ZTV7Derived+16>:       0x00005555555556ac
(gdb) x 0x00005555555556ac
0x5555555556ac <Derived::fun1()>:       0xe5894855fa1e0ff3
(gdb) x 0x0000555555557cf8
0x555555557cf8 <_ZTV7Derived+24>:       0x00005555555556e8
(gdb) x 0x0000555555557d00
0x555555557d00 <_ZTV7Derived+32>:       0x0000555555555724
(gdb) x 0x00005555555556e8
0x5555555556e8 <Derived::fun2()>:       0xe5894855fa1e0ff3
(gdb) x 0x0000555555555724
0x555555555724 <Derived::fun3()>:       0xe5894855fa1e0ff3
(gdb) 
```
可以看到实例对象指针pt存放着vptr指针地址 0x0000555555557cf0，该地址指向虚函数表.取出该地址存放的值，也是一个地址 0x00005555555556ac 。该地址指向 `Derived::fun1()`。同理取虚函数表中的下一个地址，可以发现指向的就是`Derived::fun2()`。