---
title: some C++ new Parts in 11/14/17/20
description: some new important parts in cpp11/14/17/20
slug: some-cpp-new-parts
image:
draft: false
categories:
  - Programming
date: 2025-10-05 16:36:14+08:00
tags:
  - CPP
weight: 0
---
# Intro
前几天被朋友问到 C++11中引入了哪些新特性。自己竟然直接语塞答不出来，发现自己虽然知道一些似乎高大上的名词，譬如CTAD，concepts，三五定律等等，但对c++整体的了解存在不足，于是有了这篇文章。当然这篇文章的水平也十分低下，笔者对于所提到的知识的细节事实上非常不足，许多概念都需要借助LLM为我讲解，如有错误，还请批评指正。

> Reference:  
>  [Curated Learning C++ Playlist](https://www.youtube.com/playlist?list=PLs3KjaCtOwSY34fFKyhOFovFlB7LikDwe)

# C++11

## auto

## ranged-for loops

```C++
for(const auto &i : vec){
	//do something
}
```
## lambda

## unique_ptr

## constexpr

## variadic templates

# C++14
对C++11的小修小补
## auto
函数返回类型可以是 `auto`
## lambda优化
lambda可以使用auto作为参数类型，自动捕获不同类型的变量。
增强了捕获机制，可以使用移动捕获`unique_ptr`，或者在捕获时创建一个新变量（你甚至可以在捕获里再写一个lambda）。
## `make_unique`
现在可以使用`make_unique` 而不是 `new` 来初始化指针了（我们整个程序都可以不使用`new` 和 `delete`了）。
## constexpr
强化了`constexpr` 的功能。比如循环，分支等都可以使用`constexpr`了
# C++17
## Copy/Move Elison（强制复制消除）
在C++14中
```C++

#include <iostream>

struct MyType {
    MyType() { std::cout << "Default constructor\n"; }
    MyType(const MyType&) { std::cout << "Copy constructor\n"; }
    MyType(MyType&&) noexcept { std::cout << "Move constructor\n"; }
    ~MyType() { std::cout << "Destructor\n"; }
};

MyType create_object() {
    return MyType(); // 返回一个临时对象
}

int main() {
    MyType obj = create_object();
    return 0;
}
```
会输出Default constructor，（会消除掉一个移动），但是这是建议进行的，是由编译器进行的优化，并且要求复制构造/移动构造是可访问的。在17中，这要求成为强制执行。所以下面这段代码能在C++17而不能在C++14通过编译。
```C++
#include <iostream>

struct NoCopyMove {
  NoCopyMove() { std::cout << "Default constructor\n"; }
  NoCopyMove(const NoCopyMove&) = delete; // 删除拷贝构造
  NoCopyMove(NoCopyMove&&) = delete; // 删除移动构造
  ~NoCopyMove() { std::cout << "Destructor\n"; }
};

  
NoCopyMove create_object() {
  return NoCopyMove(); // 这是一个纯右值 (prvalue)
}

int main() {
  NoCopyMove obj = create_object(); // 初始化 obj
}
```
## STL 和 lambda 中 开始加入 `constexpr` 的支持


## `string_view`
一个非拥有的，轻量的只读的字符串工具，除非有修改原始字符串的需求，否则大部分情况都可以使用`string_view` 。不过需要注意指向对象先于 `string_view`的生命周期前被销毁造成空悬引用。

## Class Template Argument Deduction
类模板参数类型推导，CTAD，如
```C++
#include <array>

std::array arr{1,2,3,4,5};
```
不需要显式写出`array`的参数`<int,5>`。

## fold expression

过去的模板处理多个参数需要进行递归，如
```C++

#include <iostream>

// 递归的终止条件（基础情况）
template<typename T>
T sum(T t) {
    return t;
}

// 递归的模板定义
template<typename T, typename... Args>
T sum(T t, Args... args) {
    return t + sum(args...); // 递归调用，将当前参数与剩余参数的和相加
}

int main() {
    std::cout << sum(1, 2, 3, 4, 5) << std::endl; // 输出 15
}

```
而在C++17中，
```C++

#include <iostream>

template<typename... Args>
auto sum(Args... args) {
    return (args + ...); // 这就是折叠表达式！
}

int main() {
    std::cout << sum(1, 2, 3, 4, 5) << std::endl; // 输出 15
}

```
## structured bindings

可以将 `std::pair`这样类型的两个成员分别绑定到不同的变量上，并且不需要提前声明（即使用`std::tie`)
```C++

#include <iostream>
#include <string>
#include <utility>
  
std::pair<std::string, int> get_person() {
	return {"Alice", 30};
}

int main() {
	auto [name, age] = get_person(); // 一行代码完成解构和声明
	//name type : basic_string<char>
	//age type : int
	std::cout << "Name: " << name << ", Age: " << age << std::endl;
}
```
## if-init expressions
C++17 可以在 if 语句的条件部分，直接声明并初始化一个变量。这个变量的作用域被严格限制在 if 及其对应的 else if / else 块内部。避免了作用域的污染。
```C++

#include <iostream>
#include <map>
#include <string>

std::map<std::string, int> user_ages = {{"Alice", 30}, {"Bob", 25}};

void check_user_cpp17(const std::string& name) {
    // 初始化和条件检查在一行内完成
    if (auto it = user_ages.find(name); it != user_ages.end()) {
        // it 的作用域从这里开始
        std::cout << name << " is " << it->second << " years old." << std::endl;
        // it 在 if 块内可见
    } else {
        std::cout << name << " not found." << std::endl;
        // it 在 else 块内也可见 (此时 it == user_ages.end())
    }
    // it 的作用域在这里结束！
    // 在这里访问 it 会导致编译错误，这是我们期望的行为！
}
```

# C++20
我们引流狗最爱的C++20，永远遥遥无期的 `module` 喜欢吗（
## Designated Initializers（指定初始化器）

```C++
struct S{
	int i;
	int j;
	int k;
};

S s{.i = 1, .j = 2, .k = 3};
```
可以指定成员初始化（其余成员默认初始化）。
有如下要求
- 必须是聚合类型，即：没有自定义构造函数，没有`private` 或  `protected` 类型的成员，没有基类，没有虚函数。（感觉基本上就是适用于一些`struct` 初始化）
- 初始化时成员不允许乱序，不允许多次出现。
## 太空船运算符（三向比较运算符）
~~这东西真有人用吗（）~~

在 `<compare>` 中定义了 `std::strong_ordering` （强排序） `std::weak_ordering` （弱排序） `std::partial_ordering` （偏排序） 
### `std::strong_ordering`
`std::strong_ordering` 存在的关系分别为 `less` `equal` `greater` ，当两者关系为`equal` 时，两者可交换。  
通常用于 `int` ，标准库容器等。
### `std::weak_ordering`
相较于 `std::strong_ordering` ，`std::weak_ordering` 的关系为`less` `equivalent` `greater` ，其中的 `equivalent` 不是严格的相等关系，而是等价关系 ：如 `"hello"` 和 `"HELLO"` 。两个关系为 `equivalent` 的对象不可以交换,但是`==` 运算符为真。
### `std::partial_ordering` 
存在关系为`less` `equivalent` `greater`  `unordered` ，其中 `unordered` 类型 意思是无法比较，此时：`A<B`,`A>B`,`A==B`,`A<=B`,`A>=B` 全部为`false` . 如一个浮点数和 `NaN` 比较结果就是`std::partial_ordering::unordered` 。 

---

- 编译器会根据返回的比较结果，自动生成出 `<` `>` `==` `!=` `<=` `>=` 的bool值。
- 比较过程是字典式的，比如说，在第一个成员中就可以判断出 **小于** 的关系，那么即使之后的每一个成员都是 **大于** 也是返回小于。
```C++
#import <iostream>

struct S{
int i;
int j;
int k;

constexpr auto operator<=>(const S&) const = default;
};

int main(){
S s1{.i =1, .j = 5, .k = 5};
S s2{.i =2, .j = 2, .k = 2};
  
bool res = s1 < s2;
std::cout << res << std::endl;
//输出结果为 1
return 0;

}
```
## module
不清楚具体实现，听朋友说是预编译，可以不导入头文件改为导入模块  

## coroutines
不清楚,目测和协程有关

## concepts
可以对模板进行约束，提高了模板编程的质量。  

譬如 我之前在写一个自己的矩阵类，初始化时需要满足一个初始化列表中有且仅有16个元素，朋友跟我说可以使用 `requires` 关键词约束。

## `<format>`
感觉不如。。。`fmt`  
简单来说，可以使用类似于 `("info {}:{}", id, description)` 这种语法来灵活的输出，并且，这种可以在 `{}` 输入对应的索引达到乱序的目的。

## `<source_location>`
可以在获得源文件的信息，如文件名，行号，函数名（这不是我们的`__FILE__`, `__LINE__`, `__func__`吗），并且是在编译时获取的，在运行时开销非常小。

## 万年历的功能增强

## `<ranges>`

emm我也不会讲怎么形容它，总之就是对于容器的数据有了更方便、更安全的处理方式？  
这里有一个来自gemini的示例。顺带一提 管道运算符`|` 和命名空间别名似乎也是C++20才引入的。
```C++
#import <iostream>
#import <vector>
#import <ranges>

int main(){
  
std::vector numbers{1,2,3,4,5,6,7,8};
  
namespace views = std::views;
  
auto process_view = numbers | views::filter([](int n ){return n%2 == 0 ;})
			| views::transform([](int n){return n*n;});

return 0;
}
```

## `constexpr` 
1. 虚函数可被声明为`constexpr` ，可在编译时进行多态调用
2. `constexpr` 可以被用于内存分配和释放了，所以`std::vector` 等容器可以在编译期求值。
3. `constexpr` 函数可以使用try-catch了
4. 新关键词 `consteval` ,被声明的函数必须在编译器执行，而 `constexpr` 可以在运行时运行。
5. 新关键词 `constinit` ，变量必须静态初始化。

## `<span>`
`span`是一个轻量级的，安全的，高效的用于操作连续内存队列的一部分或全部（如 `std::vector` ， `std::array` 或者C-style数组 ）

```C++
#include <span>
#include <vector>
#include <array>
#include <iostream>

// 函数接受一个 span，可以处理所有连续容器
void print_elements(std::span<const int> data) {
    std::cout << "Size: " << data.size() << std::endl;
    for (int n : data) {
        std::cout << n << " ";
    }
    std::cout << std::endl;
}

void test_span() {
    // 1. 原生数组
    int arr[] = {1, 2, 3, 4, 5};
    print_elements(arr); 

    // 2. std::vector
    std::vector<int> vec = {10, 20, 30};
    print_elements(vec);

    // 3. std::array
    std::array<int, 2> arr20 = {55, 66};
    print_elements(arr20);

    // 4. 作为子序列视图
    // span 可以通过构造函数或 subspan() 方法表示原序列的子集
    std::span<int> vec_span(vec); 
    auto sub_span = vec_span.subspan(1, 1); // 从索引 1 开始，长度为 1
    
    // 通过 span 修改原始数据
    sub_span[0] = 999; 
    
    std::cout << "Modified vec[1]: " << vec[1] << std::endl; // 输出 999
}
```

## thread updates
我不知道