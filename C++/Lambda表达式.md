# Lambda表达式

> 在C++11和更高版本中，Lambda表达式（通常称为Lambda）是一种在被调用的位置或作为参数传递给函数的位置定义匿名函数对象（闭包）的简便方法。Lambda通常用于封装传递给算法或异步函数的少量代码行。[^1]

Lambda表达式是C++11中一个较为重要的特性，其来源于函数式编程的概念，也是现代编程语言的一个特点。

Lambda表达式有以下优点：
+ 声明式编程风格：就地匿名定义目标函数或函数对象，不需要额外写一个命名函数或函数对象。以更直接的方式撰写程序代码，具有较高的可读性和可维护性。
+ 简介：不需要额外写一个函数或函数对象，避免代码膨胀和功能分散，让开发者更加集中精力在手边的问题，同时也具备更高的生产效率。
+ 在需要的时间和地点实现功能闭包，使程序更灵活。[^2]

下文将记录和简述Lambda表达式中的基本概念和用法。

## 一、Lambda表达式组成

ISO C++ 标准展示了作为第三个参数传递给 `std::sort()` 函数的简单 lambda：
~~~ cpp
#include <algorithm>
#include <cmath>

void abssort(float* x, unsigned n) {
	std::sort(x, x+n, 
		// Lambda expression begins
		[](float a, float b) {
			return (std::abs(a) < std::abs(b));
		} // end of lambda expression
	);
}
~~~

根据上面的简单Lambda示例，可以将其按照下图进行概括：

![Lambda组成](https://learn.microsoft.com/zh-cn/cpp/cpp/media/lambdaexpsyntax.png?view=msvc-170)

其中各个组成部分分别表示为：
1. capture子句（C++规范中也称为Lambda引导）
2. 参数列表（可选）。（也称为Lambda声明符）
3. mutable规范（可选）。
4. exception-specification（可选）。
5. trailing-return-type（可选）。
6. Lambda体，即函数体。

## 二、capture子句

Lambda 以 capture 子句开头，它用于指示 Lambda 捕获周边范围中的哪些变量，以及捕获变量的方式（按值捕获、按引用捕获'&'），当然也可以使用空的 capture 子句`[ ]`表示不捕获任何变量。

除了在 capture 子句中直接指定待捕获的外部变量，也可以使用默认捕获模式来指示如何捕获Lambda体中引用的任何外部变量：
1. `[&]`：表示通过“**引用捕获**”引用所有的变量

2. `[=]`：表示通过“**值捕获**”获取外部变量的值

<font color=red>当使用默认捕获模式时，仍然可以为特定变量显式地指定相反的模式</font>。例如，Lambda 体通过引用访问外部变量 `a` ，并通过值访问外部变量 `b` ，则可以参考以下 capture 子句：

~~~ cpp
[&a, b] // 分别指定引用访问a，值访问b
[b, &a] // 单独指定时，不区分先后顺序
[=, &a] // 默认使用值捕获，但是变量a使用引用捕获
[&,  b] // 默认使用引用捕获，但是表量b使用值捕获
// 使用默认捕获时，只有 Lambda 体中提及的变量才会被捕获。
~~~

<mark>在类成员函数中使用 Lambda，如果 Lambda 需要访问类的成员函数和数据成员，则需要将this指针传递给 capture 子句。在C++17及以上版本中，可以通过在 capture 子句中指定`*this`通过值捕获`this`指针</mark>。

在使用 capture 子句时，有以下几点建议：
+ <font color=red>Lambda 能捕获当前作用域内的`non-static`类型变量，对于全局变量，局部静态变量，则可以在 Lambda 体中直接引用</font>，但是此处容易存在依赖问题，在下文章节 ["Effective Modern C++ 中的 Lambda 表达式"](#八、Effective Modern C++ 中的 Lambda 表达式) 中进行介绍。
+ 引用捕获可用于修改外部变量，而值捕获却不能实现此操作。（<font color=red>`mutable`声明允许修改Lambda中的副本，但不会修改原始项</font>）
+ 引用捕获会反映外部变量的更新，而值捕获不会。
+ 避免使用默认捕获模式。（同第一条，将在 ["Effective Modern C++ 中的 Lambda 表达式"](#八、Effective Modern C++ 中的 Lambda 表达式) 章节中详细描述）

## 三、通用捕获/初始化捕获（*init capture*）（C++14）

在某些场景下，如果有一个只能被移动的对象（例如`std::unique_ptr`或`std::future`）要进入闭包中，使用C++11是无法实现的。又或者要复制的对象复制开销非常高，但移动成本却相对比较低（例如stl标准库中的大多数容器），并且开发者期望的是宁愿移动该对象到闭包而不是复制时，C++11也无法实现该目标。

在C++14标准中，增加了通用捕获，又或者叫初始化捕获，移动捕获是它可以执行的技术之一。

使用初始化捕获时，开发者可以指定：
1. 从 lambda 生成的闭包类中的数据成员名称；
2. 初始化该成员的表达式

见以下例子：
~~~ cpp
auto pNums = make_unique<vector<int>>(nums);

auto a = [ptr = move(pNums)]() {
	// use ptr
  };
~~~

## 四、参数列表

参数列表（在标准语法中称为 Lambda 声明符）是可选的。在大多数时候，它类似于参数的参数列表。

例如：
~~~ cpp
auto f = [](int a, int b) {
	return a + b;
  };
~~~

在C++14中，可以使用`auto`关键字作为类型说明符，表示泛型参数类型。
~~~ cpp
auto f = [](auto a, auto b) {
    return a + b;
  };
~~~

在C++14中，Lambda 也接受可变形参
~~~ cpp
auto f = [](auto &&...params) {
    return func(normalize(std::forward<decltype(params)>(params)...));
  };
~~~


>注意：
>由于参数列表是可选的，因此在不将自变量传递到 Lambda 表达式，并且其 Lambda 声明符不包含 exception-specification、trailing-return-type 或 **`mutable`** 的情况下，可以省略空括号。

## 五、mutable 规范

<mark>通常，Lambda 的函数调用运算符是`const-by-value`，但是对`mutable`关键字的使用可以取消。利用`mutable`规范，Lambda 表达式的主题可以修改通过值捕获的变量。</mark>

~~~ cpp
  int *b = nullptr;
  auto f4 = [b]() mutable {
    b = new int;
    *b = 10;
    cout << b << "--" << *b << endl;
    delete b;
  };
  f4();
~~~

## 六、异常规范

你可以使用 noexcept 异常规范来指示 Lambda 表达式不会引发任何异常。 与普通函数一样，如果 Lambda 表达式声明 noexcept 异常规范且 Lambda 体引发异常，Microsoft C++ 编译器将生成警告。

## 七、返回类型

编译器能够自动推导 Lambda 表达值返回值类型，缺省状态下即表示为自动推导。当然也可以手动指定 trailing-return-type。trailing-return-type 类似普通函数的return-type，但是返回类型必须更在参数列表后面，并且必须在返回类型前包含关键字 `->`。

Lambda 表达式可以生成另一个 lambda 表达式作为其返回值。

## 八、Effective Modern C++ 中的 Lambda 表达式

在《Effective Modern C++》中也对 lambda 表达式给出了相关的建议，一共包含了四项条款，将在下文进行介绍和记录。

### 条款31：避免使用默认捕获模式

Item 31: Avoid default capture modes

就个人理解而言，该条款主要还是针对使用默认的按引用捕获时，由于引用存在依赖生命周期的问题，因此极易导致悬空引用的问题。而默认的按值捕获，按照作者的描述，则可能诱骗开发者以为能解决悬空引用的问题（实际上并没有解决），还会让开发者误以为自己所构建的闭包是独立的（事实上也并不是独立的）。

对于默认按引用捕获导致悬空引用的问题，我们先来举一个例子：

~~~ cpp
#include <iostream>
#include <functional>

using namespace std;
using Fn = function<void()>;

Fn capture_by_ref();
Fn capture_by_val();

int main() {
  auto f = capture_by_ref();
  f();
  f = capture_by_val();
  f();

  return 0;
}

Fn capture_by_val() {
  int a = 2;
  return [=](){cout << "capture_by_val: a = " << a << endl;};
}

Fn capture_by_ref() {
  int a = 2;
  return [&](){cout << "capture_by_ref: a = " << a << endl;};
}

~~~
在`Ubuntu 20.04`下编译运行，结果如下图所示：
![](https://user-images.githubusercontent.com/35327600/215710670-8e68c21d-58ba-450f-8342-f49a40bc2589.png)
可以清晰的看到，按值捕获的 a 可以正常输出打印。但是在按引用捕获的情形下，a已经成为一个悬空的引用了，在 cout 时，出现了未定义的行为。每次执行输出的 a 的值都是一个随机值。

为什么会出现这个问题呢？根本原因就在于按引用捕获！

在`Fn capture_by_ref();`函数中，<font color=red>Lambda 捕获了一个临时变量的引用，但是在函数 return 之后，临时变量 a 将被系统自动释放出栈，此时 a 的生命周期已经结束</font>。而 Lambda 中却仍然保存着该临时变量的引用，“这里我们可以暂时将其理解为闭包类中保存的是原临时变量 a 的地址，在函数`capture_by_ref` return 之后，该地址被释放，里面的内容自然变成了未定义的内容”，所以每次调用时程序所打印的内容都是未知的值。

这个例子很好的说明了按引用捕获存在着较高的引用悬空的风险，默认的引用捕获则更是加大了这种风险，因此尽量还是避免使用默认的按引用捕获。

`Fn capture_by_val();`采用了按值捕获的方式，正确打印了被捕获的变量 a 的值，这也确实是解决被引用捕获的变量的生命周期短于 Lambda 导致引用悬空问题的一个正确方法。但是这并不代表着按值捕获不存在问题。按值捕获同样存在着依赖生命周期的问题。

再来看一个简单的例子
~~~ cpp
#include <iostream>
using namespace std;

int main() {
  int *a = new int;
  *a = 1;
  // Lambda 按值捕获指针变量 a，在闭包类中保存 a 的副本
  auto f = [=]() { cout << *a << endl; };
  // 在外部调用 delete 释放指针变量 a
  delete a;
  // 调用 Lambda，运行时会提示 heap-used-after-free
  f();

  return 0;
}
// avoid_default_capture.cc
~~~

运行结果如下：
![](https://user-images.githubusercontent.com/35327600/215921879-efb4fc60-68db-4e35-a1a8-53c75da3bc1b.png)

结果是显而易见的，尽管我们在 Lambda 中使用了按值捕获以获得指针变量 a 的副本，但是我们无法避免外部对这个指针变量的 delete 操作。在这个例子中，Lambda 表达式执行时，指针变量 a 已经成为一个未定义的内容，直接对其解引用操作自然也就存在严重错误了。

当然这个例子放在这里可能比较极端，但是在多线程的异步编程中，变量生命周期长短不同，需要大量线程同步的情况随处可见，因此不得不小心。更何况在多线程中，还存在着大量的异步读写，这也可能会导致 Lambda 对该变量存在脏读的可能。

如果只能使用C++11标准，那么对于这种特定的问题，可以<font color=red>通过给期望捕获的变量做一个局部副本，然后捕获该副本去解决</font>，比如像下面例子：
~~~ cpp
#include <iostream>
using namespace std;

int main() {
  int *a = new int;
  *a = 1;
  // 期望捕获指针变量 a，用以获取 a 中存储的内容
  // 声明一个 _a 变量，作为指针变量 a 的局部副本
  int _a = *a;
  // 通过按值捕获副本，避免出现依赖生命周期的问题
  auto f = [_a]() { cout << _a << endl; };
  // 在外部调用 delete 释放指针变量 a
  delete a;
  // 此时调用函数 f()，就不再出现之前的“heap-used-after-free”问题
  f();

  return 0;
}
// avoid_default_capture.cc
~~~

<mark>如果你被允许使用C++14或者更高的标准，那么使用“初始化捕获”可能是更好的选择。</mark>

默认捕获模式，还存在一个更大的隐患，在类的“non-static”成员函数中使用默认捕获。众所周知，在类的“non-static”成员函数中，都包含了一个隐式指针 “**this**”。这里直接展示《Effective Modern C++》中的例子
~~~ cpp
using FilterContainer =                     //“using”参见条款9，
    std::vector<std::function<bool(int)>>;  //std::function参见条款2

FilterContainer filters;                    //过滤函数
class Widget {
public:
    …                       //构造函数等
    void addFilter() const; //向filters添加条目
private:
    int divisor;            //在Widget的过滤器使用
};

void Widget::addFilter() const
{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
    );
}
~~~

在函数`void Widget::addFilter() const;`中，Lambda 中使用了一个变量 `divisor`。对于不熟悉 lambda 的开发者来说，可能不太能理解变量 divisor 是什么，从哪里来的。C++ 中类的“non-static”成员函数都包含一个隐式的参数——“**this**指针”，而在 lambda 的捕获子句中使用了默认按值捕获的模式，这也就意味着 lambda 捕获了这个类的“**this**”指针，因此可以在 lambda 体中直接访问该类的数据成员和成员函数。<font color=red>由于捕获只能应用与 lambda 被创建时所在作用域内的 “non-static” 局部变量（包括形参）</font>,因此直接显式地指定捕获 divisor 变量，或者删除默认捕获模式，都将导致该代码编译失败。

这也就是前面提到的，默认按值捕获，会诱骗开发者错误地认为当前闭包是独立的。对于这个缺点，《Effective Modern C++》中还给出了一个例子，让我们来简单看一下：
~~~ cpp
void addDivisorFilter()
{
    static auto calc1 = computeSomeValue1();    //现在是static
    static auto calc2 = computeSomeValue2();    //现在是static
    static auto divisor =                       //现在是static
    computeDivisor(calc1, calc2);

    filters.emplace_back(
        [=](int value)                          //什么也没捕获到！
        { return value % divisor == 0; }        //引用上面的static
    );

    ++divisor;                                  //调整divisor
}
~~~

还记得上面提到的一个 Lambda 表达值中 capture 子句的一个限制吗？<font color=red>捕获只能应用与 lambda 被创建时所在作用域内的 “non-static” 局部变量（包括形参）</font>。那么在这个例子中，如果开发者对这个限制不了解，那么可能会错误地理解此处 Lambda 体中对 divisor 变量的使用是基于按值捕获的，也就会错误地认为在 Lambda 体中，divisor 变量始终为初始化的值。

<mark>正确的解释应该是这样。divisor 被声明为 static 类型，因此在 Lambda 体中不需要捕获该变量，即可直接引用。这也就意味着这个被添加的闭包依赖了外部的 divisor，所有的闭包会随着外部 divisor 的变化而变化。所以，这个闭包根本不是独立的！</mark>

我们再回顾一下上面 Widget 的例子，通过默认捕获模式捕获 this 指针，然后在 lambda 中访问类的数据成员，就目前来看没有任何问题。但是其中隐含了一个依赖生命周期的问题。还是《Effective Modern C++》中的例子：
~~~ cpp
void doSomeWork()
{
    auto pw =                               //创建Widget；std::make_unique
        std::make_unique<Widget>();         //见条款21

    pw->addFilter();                        //添加使用Widget::divisor的过滤器

    …
}                                           //销毁Widget；filters现在持有悬空指针！
~~~
在调用 `doSomeWork()` 时，通过 `std::make_unique` 构建了一个 Widget 的对象，并且该对象向全局的 filters 中添加了一个过滤器。但是该过滤器中依赖了 Widget 对象的 this 指针，因为在默认捕获模式下访问了对象的数据成员 divisor。但是在函数 `doSomeWork()` 结束时，基于智能指针 `std::unique_ptr` 的特性，Widget 对象将自动析构，这意味着 this 指针也将被销毁。但是我们在全局的 filters 中仍然保存一个依赖于 this 指针的过滤器，尽管这个指针已经悬空！

简单总结一下《Effective Modern C++》中条款31中对 Lambda 捕获变量过程中存在的限制和隐患：
1. 捕获只能应用与 lambda 被创建时所在作用域内的 “non-static” 局部变量（包括形参）。
2. 静态存储生命周期的对象，这些对象定义在全局空间或者命名空间，或者在类、函数、文件中被声明为 `static` ，这些变量无法被 lambda捕获，但可以直接使用
3. 无论是按值捕获，还是按引用捕获，都存在依赖生命周期的问题。其中按引用捕获可能会导致悬空引用，而按值捕获则对悬空指针很敏感（尤其是 this 指针），并且容易误导开发者产生 lambda 是独立的想法。

基于上面的几点，《Effective Modern C++》建议尽可能避免使用默认捕获模式，显式地指定期望被捕获的变量可能是个更好的选择。

### 条款32：使用初始化捕获来移动对象到闭包中

Item 32: Use init capture to move objects into closures

根据作者的表述，C++14标准中增加“初始化捕获”的初衷，是为了解决C++11中无法移动捕获的缺陷，但是移动捕获只是该捕获机制中的一中执行技术。

为什么初始化捕获可以执行移动捕获？本质上是因为下面两点特性：
1. 可以指定从 lambda 生成的闭包类中的数据成员的名称；
2. 可以指定初始化该成员的表达式；

因此可以通过移动语句的表达式完成对成员的初始化，从而实现移动捕获。下面简单介绍一下在C++14标准下通过初始化捕获将特定内容移动到闭包的方法，以及C++11实现近似移动捕获的方法。

其实在C++14中，这显得非常简单，代码如下：
~~~ cpp
class Widget {                          //一些有用的类型
public:
    …
    bool isValidated() const;
    bool isProcessed() const;
    bool isArchived() const;
private:
    …
};

auto pw = std::make_unique<Widget>();   //创建Widget；使用std::make_unique
                                        //的有关信息参见条款21

…                                       //设置*pw

auto func1 = [pw = std::move(pw)]        //使用std::move(pw)初始化闭包数据成员
            { return pw->isValidated()
                     && pw->isArchived(); };

~~~
在 capture 子句中，“=” 的左侧是指定的数据成员，右侧则是初始化表达式。这里就是通过初始化捕获的方式，将一个 `std::unique_ptr` 移动到了闭包类中。关于“=”左右两侧的作用域，我想应该不比多少，左侧既然是闭包类的数据成员，那么其作用域必然仅仅在闭包类之内，而右侧则是一个外部变量，那么其作用域必然和当前的 lambda 享有同样的作用域。

倘若上述例子中，不需要要设置 \*pw 那么可以再度简化 lambda 的捕获语句，如下所示：
~~~ cpp
auto func1 = [pw = std::make_unique<Widget>()]
            { return pw->isValidated()
                     && pw->isArchived(); };
~~~
因此，在C++14标准下，通过初始化捕获模式完成移动捕获是一件非常简单和便捷的事。但是这并不代表在C++11标准下，移动捕获是不可实现的。

对于在C++11下实现移动捕获，《Effective Modern C++》中给出了两种方式来模拟初始化捕获。

第一种，手写“闭包类”。<font color=red>Lambda 表达式只是生成一个类和创建该类型对象的一种简单方式</font>，因此通过手写实现这个类，同样可以模拟实现初始化捕获。直接看例子：
~~~ cpp
class IsValAndArch {                            //“is validated and archived”
public:
    using DataType = std::unique_ptr<Widget>;
    
    explicit IsValAndArch(DataType&& ptr)       //条款25解释了std::move的使用
    : pw(std::move(ptr)) {}
    
    bool operator()() const
    { return pw->isValidated() && pw->isArchived(); }
    
private:
    DataType pw;
};

auto func2 = IsValAndArch(std::make_unique<Widget>()); // func 的真实类型是什么？ IsValAndArch
~~~

通过将构造函数的参数设置为右值引用类型，通过 `std::move` 完成对数据成员 pw 的初始化。并通过重载括号运算符 `()` ，使对象可以通过 `()` 直接获取 Widget 对象 pw 的状态。虽然此处 func2 的使用方式和上文中的 func1 看起来一致，都形如 `func()`，但是两个 func 在本质上存在着差异，func1 实际为一个闭包对象，可以通过 `std::function` 进行包装，而 fun2 本质上只是普通类 IsValAndArch 的一个实例对象而已！

第二种模拟初始化捕获的方式，则是使用 `std::bind`：
1. 将要捕获的对象移动到由 `std::bind` 产生的函数对象中；
2. 将“被捕获的”对象的引用赋予给 lambda 表达式。
~~~ cpp
std::vector<double> data;               //要移动进闭包的对象

…                                       //填充data

auto func = [data = std::move(data)]    //C++14初始化捕获
            { /*使用data*/ };

auto func_cpp11 =
    std::bind(                              //C++11模拟初始化捕获
        [](const std::vector<double>& data) //译者注：本行高亮
        { /*使用data*/ },
        std::move(data)                     //译者注：本行高亮
    );

~~~
当然上面两个func不能写到同一处，因为在初始化捕获时，data已经将其内容移动到 func 这个闭包对象的数据成员 data 中了，在 func_cpp11 中再次进行 move 自然是无效移动了。

<font color=red>在默认情况下，从 lambda 生成的闭包类中的 operator() 成员函数为 const 的，这能将闭包中的所有数据成员渲染为 const 的效果</font>。<mark>而 std::bind 对象内部的移动构造的 data 副本不是 const的</mark>，因此为了避免 lambda 内部对该副本产生修改，此处形参声明为 reference-to-const 是必要的。

下面是《Effective Modern C++》中对此节内容的总结，一起看一下：
+ 无法移动构造一个对象到C++11闭包，但是可以将对象移动构造进C++11的bind对象。
+ 在C++11中模拟移动捕获包括将对象移动构造进bind对象，然后通过传引用将移动构造的对象传递给lambda。
+ 由于bind对象的生命周期与闭包对象的生命周期相同，因此可以将bind对象中的对象视为闭包中的对象。

### 条款33：对auto&&形参使用decltype用以std::forward（完美转发）它们

Item 33: Use decltype on auto&& parameters to std::forward them

> 由于完美转发掌握的有限，后续在进行补充

### 条款34：考虑lambda而非std::bind

Item 34: Prefer lambdas to std::bind



## 参考资料

[^1]: [C++ 中的 Lambda 表达式（https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170）](https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)
[^2]: [C++11 lambda表达式精讲（http://c.biancheng.net/view/3741.html）](http://c.biancheng.net/view/3741.html)


