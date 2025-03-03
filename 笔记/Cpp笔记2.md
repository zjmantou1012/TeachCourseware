---
author: zjmantou
title: Cpp笔记2
time: 2025-03-03 周一
tags:
  - cpp
  - 笔记
---
# 容器

## 线性容器

- std::array：大小固定、提供empty和size、sort的api
- std::vector：自动扩容，删除需要手动运行`shrink_to_fit()`释放内存
- std::list： 双向链表
- std::forward_list：单向链表，唯一不提供size方法的容器；

std::array兼容C风格接口

```cpp
void foo(int *p, int len) {
    return;
}

std::array<int, 4> arr = {1,2,3,4};

// C 风格接口传参
// foo(arr, arr.size()); // 非法, 无法隐式转换
foo(&arr[0], arr.size());
foo(arr.data(), arr.size());

// 使用 `std::sort`
std::sort(arr.begin(), arr.end());
```

## 有序容器

- std::map
- std::set

## 无序容器

- std::unordered_map/std::unordered_multimap
- std::unordered_set/std::unordered_multiset

# 元组

1. std::make_tuple：构造元组
2. std::get：获得元组某个位置的值
3. std::tie：元组拆包
```cpp
#include <tuple>
#include <iostream>

auto get_student(int id)
{
    // 返回类型被推断为 std::tuple<double, char, std::string>

    if (id == 0)
        return std::make_tuple(3.8, 'A', "张三");
    if (id == 1)
        return std::make_tuple(2.9, 'C', "李四");
    if (id == 2)
        return std::make_tuple(1.7, 'D', "王五");
    return std::make_tuple(0.0, 'D', "null");
    // 如果只写 0 会出现推断错误, 编译失败
}

int main()
{
    auto student = get_student(0);
    std::cout << "ID: 0, "
    << "GPA: " << std::get<0>(student) << ", "
    << "成绩: " << std::get<1>(student) << ", "
    << "姓名: " << std::get<2>(student) << '\n';

    double gpa;
    char grade;
    std::string name;

    // 元组进行拆包
    std::tie(gpa, grade, name) = get_student(1);
    std::cout << "ID: 1, "
    << "GPA: " << gpa << ", "
    << "成绩: " << grade << ", "
    << "姓名: " << name << '\n';
}

```

## 合并

std::tuple_cat

```cpp
auto new_tuple = std::tuple_cat(get_student(1), std::move(t));
```

## 遍历

获取一个元组的长度:
```cpp
template <typename T>
auto tuple_len(T &tpl) {
    return std::tuple_size<T>::value;
}
```

迭代：
```cpp
// 迭代
for(int i = 0; i != tuple_len(new_tuple); ++i)
    // 运行期索引
    std::cout << tuple_index(new_tuple, i) << std::endl;
```