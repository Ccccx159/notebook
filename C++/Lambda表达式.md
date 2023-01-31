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
~~~c++
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

当使用默认捕获模式时，仍然可以为特定变量显式地指定相反的模式。例如，Lambda 体通过引用访问外部变量 `a` ，并通过值访问外部变量 `b` ，则可以参考以下 capture 子句：

~~~ c++
[&a, b] // 分别指定引用访问a，值访问b
[b, &a] // 单独指定时，不区分先后顺序
[=, &a] // 默认使用值捕获，但是变量a使用引用捕获
[&,  b] // 默认使用引用捕获，但是表量b使用值捕获
// 使用默认捕获时，只有 Lambda 体中提及的变量才会被捕获。
~~~

在类成员函数中使用 Lambda，如果 Lambda 需要访问类的成员函数和数据成员，则需要将this指针传递给 capture 子句。在C++17及以上版本中，可以通过在 capture 子句中指定`*this`通过值捕获`this`指针。

在使用 capture 子句时，有以下几点建议：
+ Lambda 能捕获当前作用域内的`non-static`类型变量，对于全局变量，局部静态变量，则可以在 Lambda 体中直接引用，但是此处容易存在依赖问题，在下文章节 ["Effective Modern C++ 中的 Lambda 表达式"](#八、Effective Modern C++ 中的 Lambda 表达式) 中进行介绍。
+ 引用捕获可用于修改外部变量，而值捕获却不能实现此操作。（`mutable`声明允许修改Lambda中的副本，但不会修改原始项）
+ 引用捕获会反映外部变量的更新，而值捕获不会。
+ 避免使用默认捕获模式。（同第一条，将在 ["Effective Modern C++ 中的 Lambda 表达式"](#八、Effective Modern C++ 中的 Lambda 表达式) 章节中详细描述）

## 三、通用捕获/初始化捕获（*init capture*）（C++14）

在某些场景下，如果有一个只能被移动的对象（例如`std::unique_ptr`或`std::future`）要进入闭包中，使用C++11是无法实现的。又或者要复制的对象复制开销非常高，但移动成本却相对比较低（例如stl标准库中的大多数容器），并且开发者期望的是宁愿移动该对象到闭包而不是复制时，C++11也无法实现该目标。

在C++14标准中，增加了通用捕获，又或者叫初始化捕获，移动捕获是它可以执行的技术之一。

使用初始化捕获时，开发者可以指定：
1. 从 lambda 生成的闭包类中的数据成员名称；
2. 初始化该成员的表达式

见以下例子：
~~~ c++
auto pNums = make_unique<vector<int>>(nums);

auto a = [ptr = move(pNums)]() {
	// use ptr
  };
~~~

## 四、参数列表

参数列表（在标准语法中称为 Lambda 声明符）是可选的。在大多数时候，它类似于参数的参数列表。

例如：
~~~ c++
auto f = [](int a, int b) {
	return a + b;
  };
~~~

在C++14中，可以使用`auto`关键字作为类型说明符，表示泛型参数类型。
~~~c++
auto f = [](auto a, auto b) {
    return a + b;
  };
~~~

在C++14中，Lambda 也接受可变形参
~~~c++
auto f = [](auto &&...params) {
    return func(normalize(std::forward<decltype(params)>(params)...));
  };
~~~


>注意：
>由于参数列表是可选的，因此在不将自变量传递到 Lambda 表达式，并且其 Lambda 声明符不包含 exception-specification、trailing-return-type 或 **`mutable`** 的情况下，可以省略空括号。

## 五、mutable 规范

通常，Lambda 的函数调用运算符是`const-by-value`，但是对`mutable`关键字的使用可以取消。利用`mutable`规范，Lambda 表达式的主题可以修改通过值捕获的变量。

~~~c++
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




### 条款32：使用初始化捕获来移动对象到闭包中

Item 32: Use init capture to move objects into closures



### 条款33：对auto&&形参使用decltype用以std::forward（完美转发）它们

Item 33: Use decltype on auto&& parameters to std::forward them


### 条款34：考虑lambda而非std::bind

Item 34: Prefer lambdas to std::bind



## 参考资料

+ [C++ 中的 Lambda 表达式（https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170）](https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)
+ [C++11 lambda表达式精讲（http://c.biancheng.net/view/3741.html）](http://c.biancheng.net/view/3741.html)
















[^1]: [C++ 中的 Lambda 表达式（https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170）](https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)
[^2]: [C++11 lambda表达式精讲（http://c.biancheng.net/view/3741.html）](http://c.biancheng.net/view/3741.html)

