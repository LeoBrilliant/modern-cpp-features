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
