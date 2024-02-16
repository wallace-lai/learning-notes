# 【C/C++】vector中的reserve和resize的区别

作者：wallace-lai </br>
发布：2024-02-16 </br>
更新：2024-02-16 </br>

不知道有没有人也犯过类似以下代码中的错误：对vector容器进行reserve后直接用下标访问了！

```cpp
vector<int> result;
result.reserve(4);

for (int i = 0; i < 4; i++) {
    result[i] = i;
}
```

对上述代码进行Debug后发现result的length仍然是0，达不到对result进行赋初值的作用。原因何在？先看一下文档对reserve的解释：

```cpp
void reserve (size_type n);
```

**Request a change in capacity**

Requests that the vector capacity be at least enough to contain n elements.

If n is greater than the current vector capacity, the function causes the container to reallocate its storage increasing its capacity to n (or greater).

In all other cases, the function call does not cause a reallocation and the vector capacity is not affected.

This function has no effect on the vector size and cannot alter its elements.

从文档中可以得出以下几点信息：

（1）出于性能考虑，不要多次reserve以避免产生reallocate操作

（2）reserve操作只会修改底层的capacity，不修改length。换句话说以下对result的下标操作是**未定义行为**，而未定义行为是BUG的“温床”

```cpp
for (int i = 0; i < 4; i++) {
    result[i] = i;
}
```

如果要在修改capacity的同时同时修改length，可以使用resize，文档如下：

```cpp
void resize (size_type n);
void resize (size_type n, const value_type& val);
```

**Change size**

Resizes the container so that it contains n elements.

If n is smaller than the current container size, the content is reduced to its first n elements, removing those beyond (and destroying them).

If n is greater than the current container size, the content is expanded by inserting at the end as many elements as needed to reach a size of n. If val is specified, the new elements are initialized as copies of val, otherwise, they are value-initialized.

If n is also greater than the current container capacity, an automatic reallocation of the allocated storage space takes place.

Notice that this function changes the actual content of the container by inserting or erasing elements from it.

在上述代码中使用resize替代reserve，问题解决！

```cpp
vector<int> result;
result.resize(4);

for (int i = 0; i < 4; i++) {
    result[i] = i;
}
```