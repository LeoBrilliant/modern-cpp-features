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
inline标识符可以应用到变量上，形式和语义和inline函数一样。
```C++
// 使用Compiler Explorer获得反汇编实例
struct S { int x; };
inline S x1 = S{321}; // mov esi, dword ptr [x1]
                      // x1: .long 321

S x2 = S{123};  // mov eax, dword ptr [.L_ZZ4mainE2x2]
                // mov dword ptr [rbp - 8], eax
                // .L_ZZ4main#2x2: .long 123
```

## 嵌套命名空间
使用命名空间解析操作符定义嵌套命名空间。
```C++
namespace A {
    namespace B {
        namespace C {
            int i;
        }
    }
}
// vs.
namespace A::B::C {
    int i;
}
```

### 结构化绑定
现在C++允许这样的写法`auto [ x, y, z ] = expr;`，其中`expr`是类似于tuple的对象，该语句将其元素分别绑定到`x`，`y`，`z`（`expr`声明时也包含三个元素）。_类似于tuple的对象_包括[`std::tuple`](#stdtuple)，std::pair`，[`std::array`](#stdarray)，以及聚合结构体。
```C++
using Coordinate = std::pair<int, int>;
Coordinate origin() {
    return Coordinate{0, 0};
}

const auto [ x, y ] = origin();
x; // == 0
y; // == 0
```

## 带初始化表达式的选择语句
新版的`if`和`switch`语句简化了共用代码模式，并且使共用代码的作用域更加贴合使用意图。
```C++
{
    std::lock_guard<std:;mutex> lk(mx);
    if (v.empty()) v.push_back(val);
}
// vs.
if (std::lock_guard<std::mutex> lk(mx); v.empty()) {
    v.push_back(val);
}
```
```C++
Foo gadget(args);
switch (auto s = gadget.status()) {
    case OK: gadget.zip(); break;
    case Bad: throw BadFoo(s.message());
}
// vs.
switch(Foo gadget(args); auto s = gadget.status()) {
    case OK: gadget.zip(); break;
    case Bad: throw BadFoo(s.message());
}
```

### constexpr if
代码可以根据编译时条件而实例化。
```C++
template <typename T>
constexpr bool isIntegral() {
    if constexpr (std::is_integral<T>::value) {
        return true;
    } else {
        return false;
    }
}
static_assert(isIntegral<int>() == true);
static_assert(isIntegral<char>() == true);
static assert(isIntegral<double>() == false);
struct S {};
static_assert(isIntegral<S>() == false);
```

### UTF-8 字符字面量
`u8`开头的字符字面量是`char`类型的UTF-8字符字面量。UTF-8字符字面量采用ISO 10646编码。
```C++
char x = u8'x';
```

### Enums直接用列表初始化
Enums现在可以用花括号语法来初始化。
```C++
enum byte : unsigned char {};
byte b {0}; // OK
byte c {-1}; // ERROR
byte d = byte{1}; // OK
byte e = byte{256}; // ERROR
```

## C++17 Library特性

### std::variant
类模板`std::variant`表示类型安全的`union`。`std::variant`的实例在任何时候都持有一个候选类型的值（当然也有可能是无值）。
```C++
std:;variant<int, double> v {12};
std::get<int>(v); // == 12
std::get<0>(v); // == 12
v = 12.0
std::get<double>(v); // == 12.0
std::get<1>(v); // == 12.0
```

### std::optional
类模板`std:;optional`管理一个可选的值，比如，一个值可能存在也可能不存在。optional常见的使用场景是作为一个可能失败的函数的返回值。
```C++
std::optional<std::string> create(bool b) {
    if (b) {
        return "Godzilla";
    } else {
        return {};
    }
}

create(false).value_or("empty"); // == "empty"
create(true).value(); // == "Godzilla"
// 由函数返回的optional返回值可以直接作为while和if的条件
if (auto str = create(true)) {
    // ...
}
```

### std::any
一种类型安全的容器，其中只有一个值，但是可以是任意类型。
```C++
std::any x {5};
x.has_value(); // == true
std::any_cast<int>(x); // == 5
std::any_cast<int&>(x) = 10; 
std::any_cast<int>(x); // == 10
```

### std::string_view
对string的非拥有引用。可以提供基于string的抽象，在解析字符串时非常有用。
```C++
// 常规string
std::string_view cppstr {"foo"};
// 宽字符string
std::wstring_view wcstr_v {L"baz"};
// 字符数组
char array[3] = {'b', 'a', 'r'};
std::string_view array_v(array, std::size(array));
```
```C++
std::string str {"    trim me"};
std::string_view v {str};
v.remove_prefix(std::min(v.find_first_not_of(" "), v.size()));
str; // == "    trim me"
v; // == "trim me"
```

### std::invoke
调用一个`Callable`对象，并且传入参数。常见的可调用对象有`std::function`和`std::bind`，调用方式和常规函数非常像。
```C++
template <typename Callable>
class Proxy {
    Callable c;
public:
    Proxy(Callable c): c(c) {}
    template <class... Args>
    decltype(auto) operator()(Args&&... args) {
        // ...
        return std::invoke(c, std::forward<Args>(args)...);
    }
};
auto add = [](int x, int y) {
    return x + y;
};
Proxy<decltype(add)> p {add};
p(1, 2); // == 3
```

### std::apply
调用一个可调用对象，参数以tuple传入。
```C++
auto add = [](int x, int y) {
    return x + y;
};
std::apply(add, std::make_tuple(1, 2)); // == 3
```

### std::filesystem
`std::filesystem`库提供了管理文件系统中文件、目录和路径的一个标准方法。

这里有一个例子，如果临时路径下有足够的磁盘空间，我们把一个大文件拷贝到这里。
```C++
const auto bigFilePath {"bigFileToCopy"};
if (std::filesystem::exists(bigFilePath)) {
    const auto bigFileSize {std::filesystem::file_size(bigFilePath)};
    std::filesystem::path tmpPath {"/tmp"};
    if (std::filesystem::space(tmpPath).available > bigFileSize) {
        std::filesystem::create_directory(tmpPath.append("example"));
        std::filesystem::copy_file(bigFilePath, tmpPath.append("newFile"));
    }
}
```

### std::byte
`std::byte`提供了一个将数据表示为byte的标准方式。`std:;byte`较`char`或`unsigned char`的优势在于，`std::byte`并不是字符类型，也不是算术类型，而且只能重载位运算操作符。
```C++
std::byte a {0};
std::byte b {0xFF};
int i = std::to_integer<int>(b); // 0xFF
std:;byte c = a & b;
int j = std::to_integer<int>(c); // 0
```
std::byte`是一个简单的enum，所以也可是使用花括号的初始化方式。

### Splicing for maps and sets
移动元素，合并容器，不再需要耗时的拷贝、移动、堆内存分配和释放。

例子说的是把map中的元素移动到另一个map。
```C++
std::map<int, string> src {{1, "one"}, {2, "two"}, {3, "buckle my shoe"}};
std::map<int, string> dst {{3, "three"}};
dst.insert(src.extract(src.find(1))); // 从`src`中删除{1, "one"}，并将其插入到`dst`，快如闪电。
dst.insert(src.extract(2)); // {2, "two"}, 从`src`到`dst`。
// dst == { { 1, "one" }, { 2, "two" }, { 3, "three" } };
```

插入一整个set。
```C++
std::set<int> src {1, 3, 5};
std::set<int> dst {2, 4, 5};
dst.merge(src);
// src == {5}
// dst == {1, 2, 3, 4, 5}
```

插入的元素并不在容器的作用域里。
```C++
auto elementFactory() {
    std::set<...> s;
    s.emplace(...);
    return s.extract(s.begin());
}
s2.insert(elementFactory());
```

改变map元素的key。
```C++
std::map<int, string> m {{1, "one"}, {2, "two"}, {3, "three"}};
auto e = m.extract(2);
e.key() = 4;
m.insert(std::move(e));
// m == {{1, "one"}, {3, "three"}, {4, "two"}}
```

### 并行算法
许多STL算法，比如`copy`，`find`，`sort`，开始支持*并行执行策略*：`seq`顺序执行，`par`并行执行，`par_unseq`并行非有序执行。

```C++
std::vector<int> longVector;
// 用并行执行策略查找元素
auto result1 = std::find(std::execution::par, std::begin(longVector), std::end(longVector), 2);
// 用顺序执行策略对vector排序
auto result2 = std::sort(std::execution:seq, std::begin(longVector), std::end(longVector));
```

## C++14 语言特性
