# C++17/14/11

# 概述
本文所述之出处多种多样（见[致谢](#致谢)部分），并用自己理解后的文字总结。

同时，每个C++主要版本中也包含专门的README。

C++17 包含以下新语言特性：
- [类模板之模板参数推导](#类模板之模板参数推导)
- [声明模板参数为auto类型](#声明模板参数为auto类型)
- [展开表达式](#展开表达式)
- [括号初始化列表的auto推导新规则](#括号初始化列表的auto推导新规则)
- [constexpr lambda](#constexpr-lambda)
- [inline变量](#inline-variables)
- [嵌套命名空间](#nested-namespaces)
- [结构化绑定](#structured-bindings)
- [带初始化语句的选择表达式](#selection-statements-with-initializer)
- [constexpr if](#constexpr-if)
- [utf-8字面量](#utf-8-character-literals)
- [enum的直接列表初始化](#direct-list-initialization-of-enums)

C++17 包含以下新library特性：
- [std::variant](#stdvariant)
- [std::optional](#stdoptional)
- [std::any](#stdany)
- [std::string_view](#stdstring_view)
- [std::invoke](#stdinvoke)
- [std::apply](#stdapply)
- [std::filesystem](#stdfilesystem)
- [std::byte](#stdbyte)
- [maps和sets切片](#splicing-for-maps-and-sets)
- [并行算法](#parallel-algorithms)

C++14 包含以下新语言特性：
- [二进制字面量](#binary-literals)
- [通用lambda表达式](#generic-lambda-expresstions)
- [返回类型推导](#return-type-deduction)
- [decltype(auto)](#decltypeauto)
- [给constexpr函数松绑](#relaxing-constraints-on-constexpr-functions)
- [参数化模板](#variable-templates)#可变模板？

C++14 包含以下新library特性：
- [为标准库类型的用户定义字面量](#user-defined-literals-for-standard-library-types)?
- [编译时整数序列](#compile-time-integer-sequences)
- [std::make_tuple](#stdmake_tuple)

C++11 包含以下新语言特性：
- [移动语义](#move-semantics)
- [可变参数模板](#variadic-templates)
- [右值引用](#rvalue-references)
- [转发引用](#forwarding-references)完美转发？
- [初始化列表](#initializer-lists)
- [静态断言](#static-assertioins)
- [auto关键字](#auto)
- [lambda表达式](#lambda-expressions)
- [decltype](#decltype)
- [模板别名](#template-aliases)
- [nullptr](#nullptr)
- [强类型enums](strongly-typed-enums)
- [属性](#attributes)
- [constexpr](#constexpr)
- [委托构造函数](#delegating-constructors)
- [用户定义字面量](#user-defined-literals)
- [显式虚函数重写](#explicit-virtual-overrides)
- [final标识符](#final-specifier)?
- [默认函数](#default-functions)
- [删除函数](#delete-functions)
- [基于范围的for循环](#range-based-for-loops)
- [移动语义下的特殊成员函数](#special-member-functions-for-move-semantics)
- [转换构造函数](#converting-constructors)
- [显式转换函数](#explicit-conversion-functions)
- [inline命名空间](#inline-namespaces)
- [非静态数据成员初始化表达式](#non-static-data-member-initializers)
- [右尖括号](#right-angle-brackets)

C++11 包含以下新library特性：
- [std::move](#stdmove)
- [std::forward](#stdforward)
- [std::thread](#stdthread)
- [std::to_string](#stdto_string)
- [type traits](#type-traits)
- [smart pointers](#smart-pointers)
- [std::chrono](@stdchrono)
- [tuples](#tuples)
- [std::tie](#stdtie)
- [std::array](#stdarray)
- [unordered containers](#unordered-containers)
- [std::make_shared](#stdmake_shared)
- [memory model](#memory-model)
- [std:;async](#stdasync)

## C++17 语言特性

### 类模板之模板参数推导
模板参数自动推导与函数参数自动推导非常相似，现在构造函数也能进行模板参数推导。
```C++
template <typename T = float>
struct MyContainer {
    T val;
    MyContainer(): val() {}
    MyContainer(T val):  val(val) {}
    // ...
};

MyContainer c1 {1}; // OK MyContainer<int>
MyContainer c2; // OK MyContainer<float>
```

### 用auto声明模板参数类型
遵循`auto`的推导规则，并且模板参数列表中未指明类型的参数为规则允许的类型[\*]，则模板参数可由参数值推导出其类型：
```C++
template <auto... seq>
struct my_integer_sequence {
    // Implementation here ...
};

// 显式传递`int`作为模板参数。
auto seq = std::integer_sequence<int, 0, 1, 2>();

// 编译器推导出模板参数为`int`
auto seq2 = my_integer_sequence<0, 1, 2>();
```
\* - 比如，`double`无法作为模板参数类型，并且`double`在这里不是有效的`auto`推导结果。

### 展开表达式
展开表达式表示将模板参数包围绕二元操作符展开。
* 形如`(... op e)`或者`(e op ...)`的表达式，其中`op`表示展开操作符，`e`表示未展开的参数包，称为_单元展开_。
* 形如`(e1 op ... op e2)`的表达式，其中`op`是展开操作符，称为_二元展开_。`e1`或`e2`中有且仅有一个为未展开参数包。
```C++
template <typename... Args>
bool logicalAnd(Args... args) {
    // 二元展开
    return (true && ... && args);
}
bool b = true;
bool& b2 = b;
logicalAnd(b, b2, true); // == true
```
```C++
template <typename... Args>
auto sum(Args... args) {
    // 一元展开
    return (... + args);
}
sum(1.0, 2.0f, 3); // == 6.0
```

### 括号初始化列表的自动推导新规则
当使用括号初始化列表的统一初始化语法时，其推导规则同`auto`推导。之前，`auto x {3};`推导为`std:;initializer_list<int>`，现在推导为`int`。
```C++
auto x1 {1, 2, 3}; // 错误： 多于一个元素
auto x2 = {1, 2, 3}; // decltype(x2) 为 std:;initializer_list<int>
auto x3 {3}; // decltype(x3) 为 int
auto x4 {3.0}; // decltype(x4) 为 double
```

### constexpr lambda
用`constexpr`获得编译时lambda表达式。
```C++
auto identity = [](int n) constexpr { return n };
static_assert(identity(123) == 123);
```
```C++
constexpr auto add = [](int x, int y) {
    auto L = [=] { return x; };
    auto R = [=] { return y; };
    return [=] { return L() + R(); };
};

static_assert(add(1, 2)() == 3);
```
```C++
constexpr int addOne(int n) {
    return [n] { return n + 1; }();
}
static_assert(addOne(1) == 2);
```

### Lambda按值捕获`this`指针
之前的规范是按引用捕获`this`指针。按值捕获`this`的一个可能出问题的场景是，在异步代码中进行回调，回调时要求对象必须存在，而实际上对象可能已经超出了它的声明周期。`*this`(C++17)会拷贝当前对象，而`this`(C++11)仍然按值捕获。
```C++
struct MyObj {
    int value { 123 };
    auto getValueCopy() {
        return [*this] { return value; };
    }
    auto getValueRef() {
        return [this] { return value; };
    }
};

MyObj mo;
auto valueCopy = mo.getValueCopy();
auto valueRef = mo.getValueRef();
mo.value = 321;
valueCopy(); // 123
valueRef(); // 321
```

# 内联变量
