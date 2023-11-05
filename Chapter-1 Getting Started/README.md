# 第1章 开始

最简单的`main`函数：

```c++
int main()
{
    return 0;
}
```

C++包含两种注释，注释界定符`/**/`通常用于多行注释，而双斜杠`//`通常用于单行或半行注释。

```c++
#include <iostream>
/*
* Simple main function:
* Read two numbers and write their sum
*/
int main()
{
    int sum = 0, val = 1;
    // keep executing the while as long as val is less than or equal to 10
    while (val <= 10)
    {
        sum += val;  // assigns sum + val to sum
        ++val;       // add 1 to val
    }
    std::cout << "Sum of 1 to 10 inclusive is "<< sum << std::endl;
    return 0;
}
```