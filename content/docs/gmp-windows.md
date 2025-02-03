+++
title = 'Windows 安装 C++ GMP 大数运算库'
date = 2024-04-15T22:26:06+08:00
+++

## 使用 vcpkg + Visual Studio

### 安装 vcpkg

[通过 CMake 安装和使用包 | Microsoft Learn](https://learn.microsoft.com/zh-cn/vcpkg/get_started/get-started?pivots=shell-cmd)

1. 克隆存储库

   第一步是从 GitHub 克隆 vcpkg 存储库。 存储库包含用于获取 vcpkg 可执行文件的脚本，以及由 vcpkg 社区维护的特选开放源代码库的注册表。 若要执行此操作，请运行：

   ```shell
   git clone https://github.com/microsoft/vcpkg.git
   ```

   vcpkg 特选注册表是一组数量超过 2000 个的开源库。 这些库已通过 vcpkg 的持续集成管道进行验证，可以协同工作。 虽然 vcpkg 存储库不包含这些库的源代码，但它保存方案和元数据，以便在系统中生成和安装它们。

2. 运行启动脚本

   现在，你已经克隆了 vcpkg 存储库，请导航到 `vcpkg` 目录并执行启动脚本：

   ```console
   cd vcpkg && bootstrap-vcpkg.bat
   ```

   启动脚本执行先决条件检查并下载 vcpkg 可执行文件。

   就这么简单！ vcpkg 已安装并可供使用。

### 检查环境

安装好后运行下面这个命令看看

```shell
vcpkg search
```

### 使用 vcpkg 安装 GMP

如需下载第三方库，需运行命令：

```shell
vcpkg install <pkg>
```

`<pkg>` 就是你安装的库名，比如想安装大数运算库 GMP，那就运行：

```shell
vcpkg install gmp
```

### 在 VS 中测试代码

使用的是谢勰的 GMP 示例代码 [iterative_Fibonacci_GMP.cpp · xiexiexx/DSAD (github.com)](https://github.com/xiexiexx/DSAD/blob/main/second-edition/src/iterative_Fibonacci_GMP.cpp)。

```cpp
#include <iostream>
// 需要安装GMP库(https://gmplib.org).
#include "gmp.h"

int main()
{
  // F[i]表示Fibonacci序列中下标为i的项.
  // m2, m1, f分别代表F[n - 2], F[n - 1], F[n].
  mpz_t m2, m1, f;
  // 以字符串形式赋值m2和m1, 分别取十进制的0和1.
  mpz_init_set_str(m2, "0", 10);
  mpz_init_set_str(m1, "1", 10);
  // 初始化f(无任何值).
  mpz_init(f);
  size_t n = 2021;
  for (size_t i = 2; i < n; ++i)
  {
    // mpz_add可理解为将m2 + m1的结果赋予f.
    mpz_add(f, m2, m1);
    // mpz_set(x, y)是将y的值赋予x.
    mpz_set(m2, m1);
    mpz_set(m1, f);
  }
  mpz_add(f, m2, m1);
  std::cout << "Fibonacci " << n << std::endl;
  // 打印f的值.
  gmp_printf("%Zd\n", f);
  // 释放所有整数变量的空间, 注意列表要以NULL结尾.
  mpz_clears(m2, m1, f, NULL);
  return 0;
}
```

### Notice

- 编程环境要和安装的库**位数一致**。例如：安装的库是 64 位，则编译环境需要使用`x64`。
- 另外还可以在 VS Code 中使用 vcpkg：[在 Visual Studio Code 中使用 CMake 安装和管理包](https://learn.microsoft.com/zh-cn/vcpkg/get_started/get-started-vscode?pivots=shell-powershell)。
- 安装缓慢：[解决vcpkg下载缓慢的问题](https://blog.csdn.net/qq_39690181/article/details/82910610)。
- [一元负运算符应用于无符号类型，结果仍为无符号类型](https://blog.csdn.net/u013925378/article/details/104655923)。