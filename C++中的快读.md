在 C++ 中，进行输入输出操作时有多种方式，如 `getchar`, `cin`, `cout`, `putchar`, `printf`, `scanf`, `endl` 和 `"\n"` 等。它们之间的速度、底层机制和使用场景存在差异。

---

##  一、概述

| 方法          | 类型        | 速度 | 缓冲    | 同步      | 可移植性 | 常见用途       |
| ----------- | --------- | -- | ----- | ------- | ---- | ---------- |
| `getchar()` | C 函数      | 快  | 有     | 不同步 C++ | 高    | 单字符输入      |
| `putchar()` | C 函数      | 快  | 有     | 不同步 C++ | 高    | 单字符输出      |
| `scanf()`   | C 函数      | 快  | 有     | 不同步 C++ | 高    | 格式化输入      |
| `printf()`  | C 函数      | 快  | 有     | 不同步 C++ | 高    | 格式化输出      |
| `cin`       | C++ 流     | 较慢 | 有     | 默认同步 C  | 高    | 格式化输入，安全   |
| `cout`      | C++ 流     | 较慢 | 有     | 默认同步 C  | 高    | 格式化输出，类型安全 |
| `endl`      | C++ 流     | 慢  | 刷新缓冲区 | 是       | 高    | 换行并立即刷新缓冲区 |
| `"\n"`      | C/C++ 字符串 | 快  | 不刷新缓冲 | 否       | 高    | 换行（不立即刷新）  |

---

##  二、性能比较（实测）

以下是对多次输入输出进行性能测试的示例（以百万次操作为例）：

```cpp
#include <iostream>
#include <cstdio>
#include <chrono>
#include <cstring>
using namespace std;
using namespace std::chrono;

int main() {
    int n = 1000000;
    auto start = high_resolution_clock::now();
    for (int i = 0; i < n; ++i)
        printf("%d\n", i);
    auto end = high_resolution_clock::now();
    cout << "printf time: " << duration_cast<milliseconds>(end - start).count() << "ms\n";

    start = high_resolution_clock::now();
    for (int i = 0; i < n; ++i)
        cout << i << "\n";
    end = high_resolution_clock::now();
    cout << "cout + \\n time: " << duration_cast<milliseconds>(end - start).count() << "ms\n";

    start = high_resolution_clock::now();
    for (int i = 0; i < n; ++i)
        cout << i << endl;
    end = high_resolution_clock::now();
    cout << "cout + endl time: " << duration_cast<milliseconds>(end - start).count() << "ms\n";
}
```

> **注意**：实际测试结果可能随编译器和系统不同而变化，但通常：
>
> * `printf` 更快于 `cout`（未关闭同步时）
> * `endl` 慢于 `"\n"`，因为它刷新缓冲区
> * 关闭同步 `cin.tie(0); ios::sync_with_stdio(false);` 后，`cin/cout` 会更接近 `scanf/printf` 的速度

---

##  三、同步控制优化（C++ I/O 性能优化）

```cpp
ios::sync_with_stdio(false);
cin.tie(0);
```

这两行可以显著提升 `cin` 和 `cout` 的速度，使其接近甚至超过 `scanf` 和 `printf`，因为它们关闭了与 C I/O 的同步，并断开了 `cin` 与 `cout` 的绑定（减少刷新）。

---

##  四、各方式的代码示例与比较

### 1. `getchar` / `putchar`

适用于**字符级输入输出**，速度非常快。

```cpp
#include <cstdio>

int main() {
    char c = getchar();  // 读取一个字符
    putchar(c);          // 输出该字符
}
```

### 2. `scanf` / `printf`

适合**格式化输入输出**，更快，C 风格。

```cpp
#include <cstdio>

int main() {
    int x;
    scanf("%d", &x);     // 输入一个整数
    printf("You entered: %d\n", x);
}
```

### 3. `cin` / `cout`

类型安全、C++ 风格，适合大多数 C++ 项目，便于重载。

```cpp
#include <iostream>
using namespace std;

int main() {
    int x;
    cin >> x;           // 输入
    cout << "You entered: " << x << "\n";
}
```

### 4. `endl` 与 `"\n"` 的区别

```cpp
cout << "Line with endl" << endl;  // 刷新缓冲区，慢
cout << "Line with \\n" << "\n";   // 不刷新，快
```

> 优化建议：在循环中频繁输出时，**避免使用 `endl`**，用 `"\n"` 更高效。

---

##  五、各方法优劣总结

| 方法          | 优势           | 劣势                  |
| ----------- | ------------ | ------------------- |
| `getchar()` | 快速、直接操作字符缓冲区 | 只能读一个字符，读整行或数字需循环处理 |
| `putchar()` | 输出字符快        | 不能输出字符串或格式化文本       |
| `scanf()`   | 快、适合格式化读取    | 类型安全性低，容易出错         |
| `printf()`  | 快、格式化灵活      | 不支持对象输出，不类型安全       |
| `cin`       | 类型安全、C++风格   | 默认速度慢，需关闭同步优化       |
| `cout`      | 支持重载、自定义类型输出 | `endl` 慢、默认刷新       |
| `endl`      | 自动刷新，适合日志等场景 | 慢，不适合频繁输出           |
| `"\n"`      | 快速换行，不刷新缓冲   | 不刷新有可能导致输出延迟（调试不方便） |

---

##  六、建议选择

| 使用场景          | 推荐方式                                        |
| ------------- | ------------------------------------------- |
| **竞赛、性能敏感**   | `scanf` / `printf` / `getchar`              |
| **现代 C++ 项目** | `cin` / `cout`（搭配 `sync_with_stdio(false)`） |
| **频繁输出换行**    | 使用 `"\n"` 替代 `endl`                         |
| **字符输入输出**    | `getchar` / `putchar`                       |
| **格式化复杂输出**   | `printf`（或 C++20 的 `std::format`）           |

---

## 七、对程序输入的优化

由此可见，我们可以使用getchar以达到输入更快的目的。

而 **快读** 是指在 C++ 竞赛或高性能计算中，为了加快程序的输入速度而使用的一种自定义输入方式，它绕过标准输入流 cin 和 scanf，直接从标准输入缓冲区中读取字符，从而提高读取速度。

### 快读的本质是：

1. 使用 `getchar()` / `fgetc(stdin)` 一次读取一个字符；
2. 跳过空格、换行等非数字字符；
3. 利用字符的 ASCII 值快速转换为整数；
4. 避免调用高开销的标准库解析函数（如 `scanf` 或 `cin >>`）。

### 快读适用场景

* **大数据量读取**（如读取 10⁶ \~ 10⁷ 的整数）
* **输入格式清晰**、错误容忍度低
* **算法竞赛或高性能批处理**

### 快读的变种

快读也可以用于读取：

* long long（用 `long long` 类型）
* 浮点数（需手动处理小数点）
* 字符串（用 `getchar` + buffer 或 `fgets`）

---

## 只关于 正整数 的快读 (basis)

``` c++

#include <iostream>
#include <cstring>

int n, m;

inline int read() {
    int x = 0; // 最终返回的值
    char c = getchar(); //字符串的输入

    // start:: 这里比较强调对应的顺序， 顺序错了那么就有可能导致错误

    while (c < '0' || c > '9') c = getchar(); // 如果是0 ~ 9 范围之外的，比如说空格之类的，则继续getchar;
    while (c >= '0' && c <= '9') c = getchar(), x = x * 10 + c - '0'; // 简单的char类型转化为int类型

    // end:: 这里比较强调对应的顺序， 顺序错了那么就有可能导致错误

    return x;
}

int main() {
    n = read(), m = read(); // 使用快读
}

```

## 若输入出现负数

``` c++

#include <iostream>
#include <cstring>

int n, m;

inline int read() {
    int x = 0, judge = 0;
    char c = getchar();

    // start:: 这里比较强调对应的顺序， 顺序错了那么就有可能导致错误

    while (c < '0' || c > '9') c = getchar();
    if ( c == '-' ) c = getchar(), judge = 1; //判断是否负数，一定要注意位置顺序
    while (c >= '0' && c <= '9') c = getchar(), x = x * 10 + c - '0';

    // end:: 这里比较强调对应的顺序， 顺序错了那么就有可能导致错误

    if ( judge == 1 ) return -x;
    return x;
}

int main() {
    n = read(), m = read(); // 使用快读
}

```

## 若出现字母
在某些题中，对于字符串处理，（其实本人更倾向于用 `cin` （关闭同步流）调试会方便

``` c++
#include <iostream>
#include <cstring>

std::string s1, s2;

int main() {
    std::ios_base::sync_with_stdio(false);
    std::cin.tie(0);
    cin >> s1 >> s2;
}
```
其中经典模版 P3370 【模板】字符串哈希 https://www.luogu.com.cn/problem/P3370
这种推荐用cin或者可以写个getline(cin, 变量);

其实都大差不差, 若是输入字母，`getline` 可以解决，或者单字符串直接使用`getchar`

## 浮点数
小数点单独判断扩展
``` c++
#include <iostream>
#include <cstdio>
#include <cctype>
using namespace std;

inline double fastReadDouble() {
    double x = 0, frac = 0;
    int f = 1, div = 1;
    char ch = getchar();

    while (!isdigit(ch) && ch != '-' && ch != '.') {
        ch = getchar();
    }

    if (ch == '-') {
        f = -1;
        ch = getchar();
    }

    while (isdigit(ch)) {
        x = x * 10 + (ch - '0');
        ch = getchar();
    }

    if (ch == '.') {
        ch = getchar();
        while (isdigit(ch)) {
            frac = frac * 10 + (ch - '0');
            div *= 10;
            ch = getchar();
        }
    }

    return f * (x + frac / div);
}

int main() {
    double n = fastReadDouble();
    cout << n;
    return 0;
}
```


# ！注意
在一开始若考虑选择快读，请认真的测试一遍你的 **快速** 是否 是否 是否！ 有问题， 这也是大多数初学者会犯的错误。


##  一、快读要解决的核心问题

1. **跳过非数字字符（如空格、换行）**
2. **正确识别正负号**
3. **高效读取连续字符**
4. **正确转换为数值类型**
5. **边界处理**
6. **若是字母是否考虑使用getchar**

---

##  二、快读常见注意事项（通用）

| 项   | 说明                                                                    | 示例 / 说明                      |   |                       |
| --- | --------------------------------------------------------------------- | ---------------------------- | - | --------------------- |
| 1️⃣ | **跳过空白字符**                                                            | 使用 \`while(c<'0'             |   | c>'9')\` 跳过空格、换行、制表符等 |
| 2️⃣ | **处理负数和正号**                                                           | 判断是否是 `'-'`（或 `'+'`，如需要）     |   |                       |
| 3️⃣ | **防止死循环**                                                             | 保证输入流始终能向前推进，`getchar()` 不能漏 |   |                       |
| 4️⃣ | **使用 `register` 或 `inline` 函数提升性能**                                   | 如 `inline int fastRead()`    |   |                       |
| 5️⃣ | **读取结束检测**                                                            | 根据 EOF (`c == EOF`) 判断文件结尾   |   |                       |
| 6️⃣ | **避免使用 cin/cout 混合**                                                  | 会导致同步问题，影响性能                 |   |                       |
| 7️⃣ | **使用 `ios::sync_with_stdio(false);` + `cin.tie(0);`（如果用 `cin/cout`）** | 关闭与 C 标准库同步                  |   |                       |
| 8️⃣ | **缓存读写（高级用法）**                                                        | 读大块数据到缓冲区再处理（用于极限数据量）        |   |                       |



### 一些小补充
大多数平台背后运行的几乎都是 **Linux** 系统
所以我们可以利用这个使用 `getchar_unlocked` 替换 `getchar` 一定程度下输入速度也会提高

`getchar_unlocked`是非线程安全的，仅在竞赛/单线程环境中使用。

``` c++

#include <iostream>
#include <cstring>

int n, m;

inline int read() {
    #ifdef __linux__
        #define getchar getchar_unlocked
    #endif
        int x = 0;
        char c = getchar(); 
        while (c < '0' || c > '9') c = getchar();
        while (c >= '0' && c <= '9') c = getchar(), x = x * 10 + c - '0'; 
        return x;
}

int main() {
    n = read(), m = read(); 
}

```
