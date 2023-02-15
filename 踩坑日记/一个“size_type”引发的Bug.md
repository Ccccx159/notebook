# 一个“size_type”引发的Bug

## 问题描述

~~~ cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int main(int argc, char* argv[]) {
  vector<int> v = {10};
  // 这是一个极端示例
  int pos = 0;
  if (pos <= v.size() - 2) {
    cout << "pos <= v.size() - 2" << endl;
  } else {
    cout << "pos > v.size() - 2" << endl;
  }
  return 0;
}
~~~

通过观察上述示例代码，请回答出程序输出内容是什么？答案是 “pos <= v.size() - 2”，还是 “pos > v.size() - 2” 呢？

可能有的人和我一样，第一反应是 `std::vector v` 中只有一个元素，因此 `v.size() == 1`，那 `0` 和 `1 - 2` 比较大小，肯定是结果是 “>” 嘛。但是程序运行结果却告诉我们，这里输出的内容是 `pos <= v.size() - 2` !

在 Ubuntu20.04 和 Debian10.2.1 中进行测试，结果均为 `pos <= v.size() - 2`。运行结果如下

![](https://user-images.githubusercontent.com/35327600/218906559-3314a6d2-d277-45a6-b50c-d9005bd3f496.png)

![](https://user-images.githubusercontent.com/35327600/218907216-e6132f30-a3e5-4ed8-8e52-63a242cc8242.png)

## 问题定位

既然运行结果是 `pos <= v.size() - 2`，那我们把这个 “<=” 号左右两边的值打印出来看看，到底是否与我们认为的 `pos == 0` 和 `v.size() - 2 == -1` 的结论一致。将日志输出行修改一下：
~~~ cpp
...
if () {
  cout << "pos[" << pos << "] <= v.size() - 2[" << v.size() - 2 << ']' << endl;
}
...
~~~

再次编译执行，结果如下：

![](https://user-images.githubusercontent.com/35327600/218909328-2a63cad0-6eae-4bef-9529-bea27192ef65.png)

在控制台输出打印中可以看到，表达式 `v.size() - 2` 并非像我们认为的那样等于 “1 - 2 == -1”，而是一个非常大的数值。其实到这里，有一定经验的程序员已经大概知道这是为什么了。负数，巨大数值，根据这两个因素基本可以确定是<font color=red>有符号类型（-1）被隐式转换成了无符号类型导致的溢出</font>！

这个问题，具体情况，我们可以通过gdb进行反汇编调试来仔细跟踪一下。
>真正的地址需要程序运行起来之后才能正确反汇编出来，否则反汇编出来的是偏移地址。

先设置一个程序入口断点，确保程序已运行，我们在 main 函数的入口设置一个断点 `b *main`，并运行命中断点：
~~~ shell
(gdb) b *main
Breakpoint 1 at 0x11f5: file ./size_t.cc, line 7.
(gdb) r
Starting program: /home/openwrt/tmp/size_t/unittest_size_t

Breakpoint 1, main (argc=1, argv=0x11bf) at ./size_t.cc:7
7       int main(int argc, char* argv[]) {
(gdb)

~~~
然后再确定一下 if 这个判断语句的地址范围和反汇编内容。在 gdb 模式下输入 `disas /m main` :
~~~ shell
(gdb) disas /m main
Dump of assembler code for function main(int, char**):
7       int main(int argc, char* argv[]) {
=> 0x00005555555551f5 <+0>:     push   %rbp

......

11        if (pos <= v.size() - 2) {
   0x0000555555555261 <+108>:   mov    -0x24(%rbp),%eax
   0x0000555555555264 <+111>:   movslq %eax,%rbx
   0x0000555555555267 <+114>:   lea    -0x50(%rbp),%rax
   0x000055555555526b <+118>:   mov    %rax,%rdi
   0x000055555555526e <+121>:   call   0x5555555554d4 <_ZNKSt6vectorIiSaIiEE4sizeEv>
   0x0000555555555273 <+126>:   sub    $0x2,%rax
   0x0000555555555277 <+130>:   cmp    %rax,%rbx
   0x000055555555527a <+133>:   setbe  %al
   0x000055555555527d <+136>:   test   %al,%al
   0x000055555555527f <+138>:   je     0x5555555552f5 <main(int, char**)+256>

......
(gdb)

~~~

我们先看其中这一段：
~~~ shell
   0x000055555555526e <+121>:   call   0x5555555554d4 <_ZNKSt6vectorIiSaIiEE4sizeEv>
   0x0000555555555273 <+126>:   sub    $0x2,%rax
   0x0000555555555277 <+130>:   cmp    %rax,%rbx
~~~

这里第一个 call 语句中，我们可以看到调用了 `std::vector::size()` 的方法，我们将断点设置在这一处 `b *0x000055555555526e`，并执行 continue，直至命中断点：
~~~ shell
(gdb) b *0x000055555555526e
Breakpoint 2 at 0x55555555526e: file ./size_t.cc, line 11.
(gdb) c
Continuing.

Breakpoint 2, 0x000055555555526e in main (argc=1, argv=0x7fffffffe4a8)
    at ./size_t.cc:11
11        if (pos <= v.size() - 2) {
(gdb) x/3i $pc
=> 0x55555555526e <main(int, char**)+121>:
    call   0x5555555554d4 <_ZNKSt6vectorIiSaIiEE4sizeEv>
   0x555555555273 <main(int, char**)+126>:      sub    $0x2,%rax
   0x555555555277 <main(int, char**)+130>:      cmp    %rax,%rbx

~~~
观察 pc 指针，已经运行到断点所在地址，从这三句汇编语句中，我们不难看出寄存器 rax 中存放的是表达式 `v.size() - 2` 的结果，rbx 中则存放的是变量 pos 的值。我们进行单步调试，并在每次步进后查看这两个寄存器的值和寄存器标志位的状态：
~~~ shell
(gdb) ni  ## call   0x5555555554d4 <_ZNKSt6vectorIiSaIiEE4sizeEv>
11        if (pos <= v.size() - 2) { 
(gdb) i r rax rbx eflags
rax            0x1                 1
rbx            0x0                 0
eflags         0x202               [ IF ]
(gdb) ni  ## sub    $0x2,%rax
11        if (pos <= v.size() - 2) {
(gdb) i r rax rbx eflags
rax            0xffffffffffffffff  -1
rbx            0x0                 0
eflags         0x297               [ CF PF AF SF IF ]
(gdb) x/3i $pc
=> 0x555555555277 <main(int, char**)+130>:      cmp    %rax,%rbx
   0x55555555527a <main(int, char**)+133>:      setbe  %al
   0x55555555527d <main(int, char**)+136>:      test   %al,%al
(gdb) ni  ## cmp    %rax,%rbx
0x000055555555527a      11        if (pos <= v.size() - 2) {
(gdb) i r rax rbx eflags
rax            0xffffffffffffffff  -1
rbx            0x0                 0
eflags         0x213               [ CF AF IF ]
(gdb)

~~~

通过记录标志位，我们不难发现，当执行 `sub $0x2 %rax` 时，标志位 CF 被置位了，这代表了<font color=red>这次的减法运算，是无符号类型数的减法运算，并且存在借位，即溢出</font>，同时 SF 也被置位了，表明当前的减法计算结果是一个负数（由于计算机中存放的数据以其补码形式存放，所以此处 0xffffffffffffffff 为补码，转换为源码就是 0x8000000000000001，十进制表示就是-1）。但是在后续的 `cmp %rax %rbx` 语句中，<mark>标志位 CF 再次被置位，也就意味着计算机将 rax 和 rbx 中的值都按照无符号数进行了减法计算</mark> `0x0 - 0xffffffffffffffff` 自然产生了借位的情况，所以计算机自然而然地认为 “0 < -1”！

所以问题的根本原因在于计算机执行 cmp 指令时，将原本应该是有符号数 “-1” 当成了无符号数 “0xffffffffffffffff” 进行比较。因此在判断大小时，出现了异常的结果。

## 问题跟踪

那为什么计算机会将 “-1” 当成是无符号数呢，我们来看一下 `std::vector::size()` 方法的声明：
~~~ cpp
size_type size() const noexcept;
~~~

因此 `v.size()` 返回的 1 是 size_type 类型的。这个类型在 cplusplus 网站中，被释义为无符号整型，通常境况下同 size_t。到这里就真相大白了，由于 `v.size()` 这个方法返回了一个无符号整形结果，因此后续的减法运算和大小比较中，C++ 默认对此进行了隐式转换有符号整型转换为无符号整形，所有的运算都变成了无符号数的运算（对于无符号整形作减法的溢出，编译器不会做出任何警告）。

结合上文中通过反汇编调试得到的结论，证明问题的原因和最初我们的猜想是一致的。

## 问题解决

既然知道了问题的根本原因，那么解决方法也相对简单，只要保证进行运算操作时类型转换是合法的即可，

因为示例代码中 `v.size()` 获得的无符号整形较小，因此我们可以将其直接显式转换为有符号整形：
~~~ cpp
...
  if (pos <= (int)v.size() - 2) {
...
~~~

但是这并不意味着只要显式地转换数据类型，就不会发生错误了。比如负数转换成无符号数，无符号数的最大值转换成有符号数，这两种就是典型的类型转换导致数值溢出的问题。

我们应该在不得不进行数据类型转换前，保证转换后不会出现溢出的问题！
