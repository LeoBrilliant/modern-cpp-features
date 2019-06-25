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

### 二进制字面量
二进制字面量提供了一种便捷的表示二进制数的方式。
数位之间可以用`'`分隔。
```C++
0b110 // == 6
0b1111'1111 // == 255
```

### 泛型lambda表达式
C++14允许`auto`作为参数列表中的类型标识符，使多态lambda表达式成为现实。
```C++
auto identity = [](auto x) { return x; };
int three = identity(3); // == 3
std::string foo = identify("foo"); // == "foo"
```

### Lambda表达式带初始化的捕获
这就允许lambda表达式在创建时，捕获用任意表达式初始化的变量。捕获变量的名字不一定非要跟闭合作用域中的变量相关，无非是在lambda函数体中新增一个变量名。需要注意，初始化表达式是在lambda表达式_创建时_求值(并非在_调用时_)。
```C++
int factory(int i) { return i * 10; }
auto f = [x = factory(2)] { return x ;}; // return 20

auto generator = [x = 0]() mutable {
    // 如果不声明为`mutable`，编译会失败，因为每次调用都会修改x的值
    return x++;
};
auto a = generator(); // == 0
auto b = generator(); // == 1
auto c = generator(); // == 2
```
因为现在可以_move_(或_forward_)一个参数到lambda表达式里，之前只能是按值或者按引用捕获，我们现在可以按值捕获一些只允许move的类型。注意，在下面的实例中，`=`左手边的`task2`的捕获列表中的`p`，是lambda函数体的私有变量，并不是指向原来的`p`。
```C++
auto p = std::make_unique<int>(1);

auto task1 = [=] { *p = 5; }; // ERROR: std::unique_ptr无法被复制
// vs.
auto task2 = [p = std::move(p)] { *p = 5; }; // OK: p 移动构造到这个闭包对象
// task2创建之后，原来的p就为空了
```
如果是按引用捕获，变量可以用一个跟被引用变量不同的名字。
```C++
auto x = 1;
auto f = [&r = x, x = x * 10] {
    ++r;
    return r + x;
};
f(); // x为2，返回12
```

### 返回类型推导
C++14中使用`auto`作为函数的返回类型，编译器会尝试推导实际的返回类型。在lambda表达式而言，我们可以用`auto`声明返回类型，编译器会推导出实际返回类型，并且返回类型有可能是引用或rvalue引用。
```C++
// 返回类型推导为`int`
auto f(int i) {
    return i;
}
```
```C++
template <typename T>
auto& f(T& t) {
    return t;
}

// 返回对推导类型的引用
auto g = [](auto &x) -> auto& { return f(x); };
int y = 123;
int& z = g(y); // `y`的引用
```

### decltype(auto)
`decltype(auto)`类型标识符也可以推导实际类型。与`auto`相比，它可以推导返回类型，并且保持引用及cv限定符。
```C++
const int x = 0;
auto x1 = x; // int
decltype(auto) x2 = x; // const int
int y = 0;
int& y1 = y;
auto y2 = y1; // int
decltype(auto) y3 = y1; // int&
int&& z = 0;
auto z1 = std::move(z); // int
decltype(auto) z2 = std::move(z); // int&&
```
```C++
// 注意：对于泛型代码特别有用！

// 返回类型是`int`。
auto f(const int& i) {
    return i;
}

// 返回类型是`const int&`.
decltype(auto) g(const int& i) {
    return i;
}

int x = 123;
static_assert(std::is_same<const int&, decltype(f(x))>::value == 0);
static_assert(std::is_same<int, decltype(f(x))>::value == 1);
static_assert(std::is_same<const int&, decltype(g(x))>::value == 1);
```

另见: [`decltype`](#decltype)

### 给constexpr函数松绑
在C++11中，`constexpr`函数体只能包含一些非常有限的语法，包括（但不限于）：`typedef`, `using`, 和一个`return`语句。在C++14，`constexpr`函数已经允许像`if`语句，多个`return`，循环等常见的语法。
```C++
constexpr int factorial(int n) {
    if (n <= 1) {
        return 1;
    } else {
        return n * factorial(n - 1);
    }
}
factorial(5); // == 120
```

### 模板变量
C++14允许变量模板化：

```C++
template<class T>
constexpr T pi = T(3.1415926535897932385);
template<class T>
constexpr T e = T(2.7182818284590452353);
```

## C++14 Library 特性

### 用户为标准库类型定义字面量
用户为标准库类型定义的字面量，包括`chrono`和`basic_string`等内建字面量。如果带`constexpr`限定符，字面量可以在编译时使用，应用场景包括编译时整数解析，二进制字面量，虚数字面量等。
```C++
using namespace std::chrono_literals;
auto day = 24h;
day.count(); // == 24
std::chrono::duration_cast<std::chrono::minutes>(day)::count(); // == 1440
```

### 编译时整数序列
类模板`std::integer_sequence`代表编译时的整数序列。有一些helper接口是基于它而实现的：
* `std::make_integer_sequence<T, N...>` - 创建了一个序列，序列中的元素类型为`T`，序列的元素为`0, ..., N - 1`。
* `std::index_sequence_for<T...>` - 将模板参数包转换为一个整数序列。

将一个数组转换为tuple：
```C++
template<typename Array, std::size_t... I>
decltype(auto) a2t_impl(const Array& a, std::integer_sequence<std::size_t, I...>) {
    return std::make_tuple(a[I], ...);
}

template<typename T, std::size_t N, typename Indices = std::make_index_sequence<N>>
decltype(auto) a2t(const std::array<T, N>& a) {
    return a2t_impl(a, Indices());
}
```

### std::make_unique
如果我们想创建一个`std::unique_ptr`实例，推荐使用`std::make_unique`，因为：
* 避免使用`new`操作符。
* 避免重复类似的代码。为每个指针指明具体指向的类型，也很烦的。
* 最重要的是，这家伙是异常安全的。假如我们这样调用函数`foo`：
```C++
foo(std::unique_ptr<T>{new T{}}, function_that_throws(), std::unique_ptr<T>{new T{}});
```
编译器会调用`new T{}`，然后`function_that_throws()`，然后调用blabla。在第一次构造`T`对象时，我们是在堆上分配内存，并执行构造函数。遇到`function_that_throws`，异常了，然后就内存泄露了。如果用`std::make_unique`，则可以避免内存泄露:
```C++
foo(std::make_unique<T>(), function_that_throws(), std::make_unique<T>());
```

想了解关于`std::unique_ptr`和`std::shared_ptr`的更多信息，请移步[智能指针](#智能指针)。

## C++11 语言特性

### Move语义
Move语义主要是为了性能优化：移动一个对象，节省了昂贵的拷贝开销。拷贝和移动的区别在于：拷贝不会改变源对象，移动要么不改变源对象，要么让源对象成为一个完全不同的对象 -- 主要是看源对象是什么。对于平凡数据来说，移动和拷贝结果相同。

移动一个对象意味着将其管理的一些资源的所有权转移到另一个对象。你可以理解为改变指针所指的位置，移动对象A到对象B，此时有两个看不见的指针，PA指向A，PB指向B，移动之后，PB指向A，PA指向什么取决于A的类型；资源在内存中的位置没有发生变化。这种廉价的资源转移对`rvalue`极其有效。移动完成之后，再去改变源对象是一种危险的做法，甚至有些多余，因为源对象是一个临时对象，很可能无法方法。

移动也可以用在`std::unique_ptr`，[智能指针](#智能指针)等，这样就可以把一些非共享的对象从一个作用域转移到另一个作用域。

另见：[右值引用](#右值引用)，[定义移动成员函数](#移动语义下的特殊成员函数)，[`std::move`](#stdmove)，[`std::forward`](#stdforward)，[`转发引用`](#forwarding-references)。

### 右值引用
C++11引入了一种新的引用类型，叫做_右值引用_。`A&&`定义了一个`A`的右值引用，其中`A`不是模板，就是像`int`这样普通的类型，或者是一个普通的用户定义类型。注意，右值引用只能绑定到右值上。

左值和右值的类型推导：
```C++
int x = 0; // `x`是一个类型为`int`的左值
int& xl = x; // `xl`是类型为`int&`的左值
int&& xr = x; // 编译错误 -- `x`是左值
int&& xr2 = 0; // `xr2`是一个类型为`int&&`的左值 -- 绑定到一个临时右值，`0`
```

参见：[`std::move`](#stdmove)，[`std::forward`](#stdforward)，[`转发引用`](#转发引用)。

### 转发引用
还有一种非官方的称呼_通用引用_。转发引用也是用`T&&`这种语法创建，`T`是磨板类型参数，或者用`auto&&`。这样就激活了两大特性：移动语义和_完美转发_。不管是左值还是右值，完美转发都可以实现参数传递。

转发引用既可以引用左值，也可以引用右值，看具体的参数类型。转发引用遵循以下_引用塌缩_规则:
* `T& &` -> `T&`
* `T& &&` -> `T&`
* `T&& &` -> `T&`
* `T&& &&` -> `T&&`

左值和右值的`auto`类型推导：
```C++
int x = 0; // `x` 是左值
auto&& al = x; // `al`是`int&`类型的左值 -- 绑定到左值`x`
auto&& ar = 0; // `ar`是`int&&`类型的左值 -- 绑定到临时右值，`0`
```

左值和右值的模板参数类型推导：
```C++
// C++14及之后版本
void f(auto&& t){
    // ...
}

// C++11及之后版本
template <typename T>
void f(T&& t) {
    // ...
}

int x = 0;
f(0); // 调用f(int&&)
f(x); // 调用f(int&)
```

另见： [`std::move`](#stdmove), [`std::forward`](#stdforward),[`右值引用`](#右值引用)。

### 可变参数模板
`...`语法创建了一个_参数包_，或者展开一个参数包。模板_参数包_是一个模板参数，接受0个或更多的模板参数（无类型的参数，有类型的参数，或者是另一个模板）。包含至少一个参数包的模板称之为_可变参数模板_。
```C++
template <typename... T>
struct arity {
    constexpr static int value = sizeof...(T);
};
static_assert(arity<>::value == 0);
static_assertarity<char, short, int>::value == 3);
```

### 初始化列表
是一种轻量的容器，看上去像数组，但是用花括号{}的语法创建。比如，`{1, 2, 3}`创建了一个整数序列，类型是`std::initializer_list<int>`。用于取代向函数传递vector对象。
```C++
int sum(const std::initializer_list<int>& list) {
    int total = 0;
    for (auto& e : list) {
        total += e;
    }

    return total;
}

auto list = {1, 2, 3};
sum(list); // == 6
sum({1, 2, 3}); // == 6
sum({}); // == 0;
```

### 静态断言
编译时对断言求值。
```C++
constexpr int x = 0;
constexpr int y = 1;
static_assert(x == y, "x != y");
```

### auto
`auto`类型的变量，由编译器根据其初始化参数推导具体类型。
```C++
auto a = 3.14; // double
auto b = 1; // int
auto& c = b; // int&
auto d = {0}; // std::initializer_list<int>
auto&& e = 1; // int&&
auto&& f = b; // int&
auto  g = new auto(123); // int*
const auto h = 1; // const int
auto i = 1, j = 2, k = 3; // int, int, int
auto l = 1, m = true, n = 1.61; // error -- `l`为int，or`m`为bool，`n`为double
auto o; // error -- `o`需要一个初始化参数
```

极大的提升了可读性，尤其是对于比较复杂的类型：
```C++
std::vector<int> v = ...;
std::vector<int>::const_iterator cit = v.cbegin();
// vs.
auto cit = v.cbegin();
```

函数也可以将返回值类型声明为`auto`，从而由编译器进行推导。不过在C++11中，函数的`auto`返回类型需要配合`decltype`使用，要么就显式声明返回值类型：
```C++
template <typename X, typename Y>
auto add(X x, Y y) -> decltype(x + y) {
    return x + y;
}
add(1, 2); // == 3
add(1, 2.0); // == 3.0
add(1.5, 1,5); // == 3.0
```
`decltype`的具体用法请参见[`decltype`](#decltype)。结合上述实例，`x`是整型，`y`是double，则`decltype(x + y)`是double。上述函数将根据表达式`x + y`的结果的类型来决定返回值类型。注意，上述`decltype`可以获得函数的参数，有时甚至可以拿到`this`指针。

### Lambda表达式
`lambda`是一个未命名的函数对象，能够捕获其作用域中的变量。它的组成部分有：一个_捕获列表_；一组参数，可以为空；一个可选的尾部返回类型；一个函数体。捕获列表的实例如下：
* `[]` - 啥都不捕获。
* `[=]` - 按值捕获作用域中的局部对象（局部变量，参数）。
* `[&]` - 按引用捕获作用域中的局部对象（局部变量，参数）。
* `[this]` - 按值捕获`this`指针。
* `[a, &b]` - 按值捕获`a`，按引用捕获`b`。

```C++
int x = 1;

auto getX = [=] { return x; };
getX(); // == 1

auto addX = [=](int y) { return x + y; };
addX(1); // == 2

auto getXRef = [&]() -> int& { return x; };
getXRef(); // int& to `x`
```
缺省情况下，lambda函数体内无法修改按值捕获的参数，因为编译器将lambda函数标记为`const`。这个时候`mutable`关键字就可以发挥作用，允许函数体修改参数。`mutable`需要放在参数列表的圆括号之后，就算参数没有参数也要把圆括号留下。
```C++
int x = 1;

auto f1 = [&x] { x = 2; }; // 没毛病：x是按引用捕获，原来的值也被修改了。

auto f2 = [x] { x = 2; }; // ERROR：lambda对按值捕获的参数只能进行const操作。
// vs.
auto f3 = [x]() mutable { x = 2; }; // 没错：lambda对按值捕获的参数，可以想干什么就干什么。
```

### decltype
`decltype`是一个操作符，能够返回表达式的_声明类型_，并且还能保持cv限定符和引用性。见下示例：
```C++
int a = 1; // `a`声明为`int`型
decltype(a) b = a; // `decltype(a)`为`int`
const int& c = a; // `c`声明为`const int&`型
decltype(c) d = a; // `decltype(c)`为`const int&`
decltype(123) e = 123; // `decltype(123)`为`int`
int&& f = 1; // `f`声明为`int&&`型
decltype(f) g = 1; // `decltype(f)`为`int&&`
decltype((a)) h = g; // `decltype((a))`为`int&`
```
```C++
template <typename X, typename Y>
auto add(X x, Y y) -> decltype(x + y) {
    return x + y;
}
add(1, 2.0); // `decltype(x + y)` => `decltype(3.0)` => `double`
```

另见：[`decltype(auto)`](#decltypeauto).

### 模板别名
语义上跟`typedef`类似。但是，使用`using`声明的模板别名可读性更佳，并且和模板兼容。
```C++
template <typename T>
using Vec = std::vector<T>;
Vec<int> v; // std::vector<int>

using String = std::string
String s {"foo"};
```

### nullptr
C++11 引入了该空指针类型，用于取代C中的NULL宏。`nullptr`自身是`std::nullptr_t`类型，可以隐式转换成指针类型。与`NULL`不同的是，`nullptr`无法转换成除`bool`之外的整数类型。
```C++
void foo(int);
void foo(char*);
foo(NULL); // error -- 有歧义
foo(nullptr); // 调用foo(char*)
```

### 强类型枚举类型
类型安全的枚举类型可以解决一大堆C式枚举类型带来的问题：隐式转换，无法指定底层类型，作用域污染等等。
```C++
// 指定底层类型是`unsigned int`
enum class Color : unsigned int { Red = 0xff0000, Green = 0xf00, Blue = 0xff};
// `Alert`内部的`Red`/`Green`并不与`Color`中同名的枚举值冲突
enum class Alert : bool { Red, Green };
Color c = Color::Red;
```

### 属性
属性提供了一个通用语法，可以替代`__attribute__(...)`，`__declspec`，等等。
```C++
// `noreturn`属性说明`f`没有返回值。
[[noreturn]] void f() {
    throw "error";
}
```

### constexpr
常量表达式`constexpr`由编译器在编译时求值，而且只能执行简单的计算。`constexpr`可以声明变量、函数等为常量表达式。
```C++
constexpr int square(int x) {
    return x * x;
}

int square2(int x) {
    return x * x;
}

int a = square(2); // mov DWORD PTR [rbp - 4], 4

int b = square2(2); // mov edi, 2
                    // call square2(int)
                    // mov DWORD PTR [rbp - 8], eax
```

`constexpr`值可以在编译时确定：
```C++
const int x = 123;
constexpr const int& y = x; // 错误 -- constexpr变量`y`必须用一个常量表达式初始化。
```

类中的常量表达式：
```C++
<!-- struct Complex { -->
    constexpr Complex(double r, double i) : re(r), im(i) {}
    constexpr double real() { return re; }
    constexpr double imag() { return im; }

private:
    double re;
    double im;
}

constexpr Complex I(0, 1);
```

### 委托构造函数
构造函数可以其初始化列表中调用该类的其他构造函数。
```C++
struct Foo {
    int foo;
    Foo(int foo) : foo(foo) {}
    Foo() : Foo(0) {}
};

Foo foo;
foo.foo; // == 0
```

### 用户定义的字面量
用户定义的字面量允许用户扩展语言，增加其偏爱的语法。定义函数`T operator "" X(...) { ... }`，其返回值类型为`T`，函数名为`X`，即可创建一个字面量。注意，函数名定义了字面量的名称。如果字面量名称首字母不是下划线，则该字面量属于保留字段，编译器不会调用。用户定义字面量函数可以接受哪些参数，都是由字面量的使用场景决定的。

将摄氏温度转换成华氏温度：
```C++
// 整形字面量要求参数是`unsigned long long`类型。
long long operator "" _celsius(unsigned long long tempCelsius) {
    return std::llround(tempCelsius * 1.8 + 32);
}
24_celsius; // == 75
```

字符串转换成整数：
```CPP
// 要求参数为`const char*`和`std::size_t`
int operator "" _int(const char* str, std::size_t) {
    return std::stoi(str);
}

"123"_int; // == 123，整型
```

### 显式虚函数重写
指明一个虚函数重写了另一个虚函数。如果函数声明时含有`override`关键字，但是并没有重写父类的虚函数，编译时报错。
```CPP
struct A {
    virtual void foo();
    void bar();
};

struct B : A {
    void foo() override; // 正确 -- B::foo重写A::foo
    void bar() override; // 错误 -- A::bar非虚函数
    void baz() override; // 错误 -- 不存在A::baz
}
```

### Final关键字
指明虚函数无法被派生类重写，或者类无法被继承。
```CPP
strcut A {
    virtual void foo();
};

struct B : A {
    virtual void foo() final;
};

struct C : B {
    virtual void foo(); // 错误 -- `foo`重写了`final`函数
};
```

类无法被继承：
```CPP
struct A final {

};

struct B : A { // 错误 -- `A`已经被标记为`final`

};
```

### 默认函数
一种简洁优雅的函数默认实现方式，常用于构造函数。
```CPP
struct A {
    A() = default;
    A(int x) : x(x) {}
    int x {1};
};
A a; // a.x == 1
A a2 { 123 }; // a.x == 123
```

继承中的默认函数：
```CPP
struct B {
    B() : x(1) {}
    int x;
};

struct C : B {
    // 调用B::B
    C() = default;
};

C c; // c.x == 1
```

### 已删除函数
一种简洁优雅的删除函数实现的语法。在防止对象拷贝时很有用。
```CPP
class A {
    int x;

public:
    A(int x) : x(x) {}
    A(const A&) = delete;
    A& operator(const A&) = delete
};

A x { 123 };
A y = x; // 错误 -- 调用了拷贝构造，但是拷贝构造已删除。
y = x; // 错误 -- 删除赋值操作符
```

### 基于范围的for循环
遍历容器元素的语法糖。
```CPP
std::array<int, 5> a {1, 2, 3, 4, 5};
for(int& x : a) x *= 2;
// a == {2, 4, 6, 8, 10};
```

注意`int`和`int&`的区别：
```CPP
std::array<int, 5> a {1, 2, 3, 4, 5};
for(int x : a) x *= 2;
// a == {1, 2, 3, 4, 5};
```

### move语义下的特殊成员函数
