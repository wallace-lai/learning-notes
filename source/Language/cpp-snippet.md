# 【C/C++】C/C++常用代码片段

作者：wallace-lai </br>
发布：2024-02-19 </br>
更新：2024-02-19 </br>

## 一、C常用代码片段

## 二、C++常用代码片段

### 提取字符中所有由分隔符隔开的数字

```cpp
string input = "1*2*3";
vector<string> vec;

string value;
stringstream ss(input);
while (getline(ss, value, '*')) {
    vec.push_back(value);
}
```

注意：**如果左侧有多余的分隔符，则结果中会出现多余的空字符串**，如下所示

（1）`1*2*3`

输出：`vec = { "1", "2", "3" }`

（2）`*1*2*3`

输出：`vec = { "", "1", "2", "3" }`

（3）`1*2*3*`

输出：`vec = { "1", "2", "3" }`

（4）`*1*2*3*`

输出：`vec = { "", "1", "2", "3" }`

