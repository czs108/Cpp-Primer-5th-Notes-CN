# 第2章 变量和基本类型

## 基本内置类型（Primitive Built-in Types）

### 算数类型（Arithmetic Types）

算数类型分为两类：整型（integral type）、浮点型（floating-point type）。 

![2-1](Image/2-1.png)

`bool`类型的取值是`true`或`false`。

一个`char`的大小和一个机器字节一样，确保可以存放机器基本字符集中任意字符对应的数字值。`wchar_t`确保可以存放机器最大扩展字符集中的任意一个字符。

在整型类型大小方面，C++规定`short` ≤ `int` ≤ `long` ≤ `long long`（`long long`是C++11定义的类型）。

浮点型可表示单精度（single-precision）、双精度（double-precision）和扩展精度（extended-precision）值，分别对应`float`、`double`和`long double`类型。

除去布尔型和扩展字符型，其他整型可以分为带符号（signed）和无符号（unsigned）两种。带符号类型可以表示正数、负数和0，无符号类型只能表示大于等于0的数值。类型`int`、`short`、`long`和`long long`都是带符号的，在类型名前面添加`unsigned`可以得到对应的无符号类型，如`unsigned int`。

字符型分为`char`、`signed char`和`unsigned char`三种，但是表现形式只有带符号和无符号两种。类型`char`和`signed char`并不一样， `char`的具体形式由编译器（compiler）决定。

如何选择算数类型：
- 当明确知晓数值不可能为负时，应该使用无符号类型。

- 使用`int`执行整数运算，如果数值超过了`int`的表示范围，应该使用`long long`类型。

- 在算数表达式中不要使用`char`和`bool`类型。如果需要使用一个不大的整数，应该明确指定它的类型是`signed char`还是`unsigned char`。

- 执行浮点数运算时建议使用`double`类型。

### 类型转换（Type Conversions）

进行类型转换时，类型所能表示的值的范围决定了转换的过程。

- 把非布尔类型的算术值赋给布尔类型时，初始值为0则结果为`false`，否则结果为`true`。
- 把布尔值赋给非布尔类型时，初始值为`false`则结果为0，初始值为`true`则结果为1。
- 把浮点数赋给整数类型时，进行近似处理，结果值仅保留浮点数中的整数部分。
- 把整数值赋给浮点类型时，小数部分记为0。如果该整数所占的空间超过了浮点类型的容量，精度可能有损失。
- 赋给无符号类型一个超出它表示范围的值时，结果是初始值对无符号类型表示数值总数（8比特大小的`unsigned char`能表示的数值总数是256）取模后的余数。
- 赋给带符号类型一个超出它表示范围的值时，结果是未定义的（undefined）。

避免无法预知和依赖于实现环境的行为。

无符号数不会小于0这一事实关系到循环的写法。

```C++
// WRONG: u can never be less than 0; the condition will always succeed
for (unsigned u = 10; u >= 0; --u)
    std::cout << u << std::endl;
```

当*u*等于0时，*--u*的结果将会是4294967295。一种解决办法是用`while`语句来代替`for`语句，前者可以在输出变量前先减去1。 

```c++
unsigned u = 11;    // start the loop one past the first element we want to print
while (u > 0) 
{
    --u;    // decrement first, so that the last iteration will print 0
    std::cout << u << std::endl;
}
```

不要混用带符号类型和无符号类型。 

### 字面值常量（Literals） 

以`0`开头的整数代表八进制（octal）数，以`0x`或`0X`开头的整数代表十六进制（hexadecimal）数。在C++14中，`0b`或`0B`开头的整数代表二进制（binary）数。

整型字面值具体的数据类型由它的值和符号决定。

C++14新增了单引号`'`形式的数字分隔符。数字分隔符不会影响数字的值，但可以通过分隔符将数字分组，使数值读写更容易。

```c++
// 按照书写形式，每3位分为一组
std::cout << 0B1'101;   // 输出"13"
std::cout << 1'100'000; // 输出"1100000"
```

浮点型字面值默认是一个`double`。

由单引号括起来的一个字符称为`char`型字面值，双引号括起来的零个或多个字符称为字符串字面值。

字符串字面值的类型是由常量字符构成的数组（array）。编译器在每个字符串的结尾处添加一个空字符`'\0'`，因此字符串字面值的实际长度要比它的内容多一位。 

转义序列： 

|      含义       | 转义字符 |
| :-------------: | :------: |
|     newline     |   `\n`   |
| horizontal tab  |   `\t`   |
|  alert (bell)   |   `\a`   |
|  vertical tab   |   `\v`   |
|    backspace    |   `\b`   |
|  double quote   |   `\"`   |
|    backslash    |   `\\`   |
|  question mark  |   `\?`   |
|  single quote   |   `\'`   |
| carriage return |   `\r`   |
|    formfeed     |   `\f`   |

```c++
std::cout << '\n';      // prints a newline
std::cout << "\tHi!\n"; // prints a tab followd by "Hi!" and a newline
```

泛化转义序列的形式是`\x`后紧跟1个或多个十六进制数字，或者`\`后紧跟1个、2个或3个八进制数字，其中数字部分表示字符对应的数值。如果`\`后面跟着的八进制数字超过3个，则只有前3个数字与`\`构成转义序列。相反，`\x`要用到后面跟着的所有数字。 

```c++
std::cout << "Hi \x4dO\115!\n"; // prints Hi MOM! followed by a newline
std::cout << '\115' << '\n';    // prints M followed by a newline
```

添加特定的前缀和后缀，可以改变整型、浮点型和字符型字面值的默认类型。 

![2-2](Image/2-2.png)

使用一个长整型字面值时，最好使用大写字母`L`进行标记，小写字母`l`和数字`1`容易混淆。

## 变量（Variables）

### 变量定义（Variable Definitions）

变量定义的基本形式：类型说明符（type specifier）后紧跟由一个或多个变量名组成的列表，其中变量名以逗号分隔，最后以分号结束。定义时可以为一个或多个变量赋初始值（初始化，initialization）。

初始化不等于赋值（assignment）。初始化的含义是创建变量时赋予其一个初始值，而赋值的含义是把对象的当前值擦除，再用一个新值来替代。

用花括号初始化变量称为列表初始化（list initialization）。当用于内置类型的变量时，如果使用了列表初始化并且初始值存在丢失信息的风险，则编译器会报错。

```c++
long double ld = 3.1415926536;
int a{ld}, b = {ld};    // error: narrowing conversion required
int c(ld), d = ld;      // ok: but value will be truncated
```

如果定义变量时未指定初值，则变量被默认初始化（default initialized）。

对于内置类型，定义于任何函数体之外的变量被初始化为0，函数体内部的变量将不被初始化（uninitialized）。

定义于函数体内的内置类型对象如果没有初始化，则其值未定义，使用该类值是一种错误的编程行为且很难调试。类的对象如果没有显式初始化，则其值由类确定。

建议初始化每一个内置类型的变量。

### 变量声明和定义的关系（Variable Declarations and Definitions）

声明（declaration）使得名字为程序所知。一个文件如果想使用其他地方定义的名字，则必须先包含对那个名字的声明。

定义（definition）负责创建与名字相关联的实体。

如果想声明一个变量而不定义它，就在变量名前添加关键字`extern`，并且不要显式地初始化变量。

```c++
extern int i; // declares but does not define i
int j;      // declares and defines j
```

`extern`语句如果包含了初始值就不再是声明了，而变成了定义。

变量能且只能被定义一次，但是可以被声明多次。

如果要在多个文件中使用同一个变量，就必须将声明和定义分开。此时变量的定义必须出现且只能出现在一个文件中，其他使用该变量的文件必须对其进行声明，但绝对不能重复定义。

### 标识符（Identifiers）

C++的标识符由字母、数字和下划线组成，其中必须以字母或下划线开头。标识符的长度没有限制，但是对大小写字母敏感。C++为标准库保留了一些名字。用户自定义的标识符不能连续出现两个下划线，也不能以下划线紧连大写字母开头。此外，定义在函数体外的标识符不能以下划线开头。

![2-3](Image/2-3.png)

### 名字的作用域（Scope of a Name）

定义在函数体之外的名字拥有全局作用域（global scope）。声明之后，该名字在整个程序范围内都可使用。

最好在第一次使用变量时再去定义它。这样做更容易找到变量的定义位置，并且也可以赋给它一个比较合理的初始值。

作用域中一旦声明了某个名字，在它所嵌套着的所有作用域中都能访问该名字。同时，允许在内层作用域中重新定义外层作用域已有的名字，此时内层作用域中新定义的名字将屏蔽外层作用域的名字。

可以用作用域操作符`::`来覆盖默认的作用域规则。因为全局作用域本身并没有名字，所以当作用域操作符的左侧为空时，会向全局作用域发出请求获取作用域操作符右侧名字对应的变量。

```c++
#include <iostream>
// Program for illustration purposes only: It is bad style for a function
// to use a global variable and also define a local variable with the same name
int reused = 42;    // reused has global scope
int main()
{
    int unique = 0; // unique has block scope
    // output #1: uses global reused; prints 42 0
    std::cout << reused << " " << unique << std::endl;
    int reused = 0; // new, local object named reused hides global reused
    // output #2: uses local reused; prints 0 0
    std::cout << reused << " " << unique << std::endl;
    // output #3: explicitly requests the global reused; prints 42 0
    std::cout << ::reused << " " << unique << std::endl;
    return 0;
}
```

如果函数有可能用到某个全局变量，则不宜再定义一个同名的局部变量。 

## 复合类型（Compound Type）

### 引用（References）

引用为对象起了另外一个名字，引用类型引用（refers to）另外一种类型。通过将声明符写成`&d`的形式来定义引用类型，其中*d*是变量名称。

```c++
int ival = 1024;
int &refVal = ival; // refVal refers to (is another name for) ival
int &refVal2;       // error: a reference must be initialized
```

定义引用时，程序把引用和它的初始值绑定（bind）在一起，而不是将初始值拷贝给引用。一旦初始化完成，将无法再令引用重新绑定到另一个对象，因此引用必须初始化。

引用不是对象，它只是为一个已经存在的对象所起的另外一个名字。

声明语句中引用的类型实际上被用于指定它所绑定的对象类型。大部分情况下，引用的类型要和与之绑定的对象严格匹配。

引用只能绑定在对象上，不能与字面值或某个表达式的计算结果绑定在一起。

### 指针（Pointer）

与引用类似，指针也实现了对其他对象的间接访问。

- 指针本身就是一个对象，允许对指针赋值和拷贝，而且在生命周期内它可以先后指向不同的对象。
- 指针无须在定义时赋初值。和其他内置类型一样，在块作用域内定义的指针如果没有被初始化，也将拥有一个不确定的值。

通过将声明符写成`*d`的形式来定义指针类型，其中*d*是变量名称。如果在一条语句中定义了多个指针变量，则每个量前都必须有符号`*`。

```c++
int *ip1, *ip2;     // both ip1 and ip2 are pointers to int
double dp, *dp2;    // dp2 is a pointer to double; dp is a double
```

指针存放某个对象的地址，要想获取对象的地址，需要使用取地址符`&`。 

```c++
int ival = 42;
int *p = &ival; // p holds the address of ival; p is a pointer to ival
```

因为引用不是对象，没有实际地址，所以不能定义指向引用的指针。

声明语句中指针的类型实际上被用于指定它所指向的对象类型。大部分情况下，指针的类型要和它指向的对象严格匹配。

指针的值（即地址）应属于下列状态之一：

- 指向一个对象。
- 指向紧邻对象所占空间的下一个位置。
- 空指针，即指针没有指向任何对象。
- 无效指针，即上述情况之外的其他值。

试图拷贝或以其他方式访问无效指针的值都会引发错误。

如果指针指向一个对象，可以使用解引用（dereference）符`*`来访问该对象。

```c++
int ival = 42;
int *p = &ival; // p holds the address of ival; p is a pointer to ival
cout << *p;     // * yields the object to which p points; prints 42
```

给解引用的结果赋值就是给指针所指向的对象赋值。

解引用操作仅适用于那些确实指向了某个对象的有效指针。

空指针（null pointer）不指向任何对象，在试图使用一个指针前代码可以先检查它是否为空。得到空指针最直接的办法是用字面值`nullptr`来初始化指针。

旧版本程序通常使用`NULL`（预处理变量，定义于头文件*cstdlib*中，值为0）给指针赋值，但在C++11中，最好使用`nullptr`初始化空指针。

```c++
int *p1 = nullptr;  // equivalent to int *p1 = 0;
int *p2 = 0;        // directly initializes p2 from the literal constant 0
// must #include cstdlib
int *p3 = NULL;     // equivalent to int *p3 = 0;
```

建议初始化所有指针。

`void*`是一种特殊的指针类型，可以存放任意对象的地址，但不能直接操作`void*`指针所指的对象。

### 理解复合类型的声明（Understanding Compound Type Declarations）

指向指针的指针（Pointers to Pointers）：

```c++
int ival = 1024;
int *pi = &ival;    // pi points to an int
int **ppi = &pi;    // ppi points to a pointer to an int
```

![2-4](Image/2-4.png)

指向指针的引用（References to Pointers）：

```C++
int i = 42;
int *p;         // p is a pointer to int
int *&r = p;    // r is a reference to the pointer p
r = &i;         // r refers to a pointer; assigning &i to r makes p point to i
*r = 0;         // dereferencing r yields i, the object to which p points; changes i to 0
```

面对一条比较复杂的指针或引用的声明语句时，从右向左阅读有助于弄清它的真实含义。 

## const限定符（Const Qualifier）

在变量类型前添加关键字`const`可以创建值不能被改变的对象。`const`变量必须被初始化。

```c++
const int bufSize = 512;    // input buffer size
bufSize = 512;      // error: attempt to write to const object
```

默认情况下，`const`对象被设定成仅在文件内有效。当多个文件中出现了同名的`const`变量时，其实等同于在不同文件中分别定义了独立的变量。

如果想在多个文件间共享`const`对象：

- 若`const`对象的值在编译时已经确定，则应该定义在头文件中。其他源文件包含该头文件时，不会产生重复定义错误。

- 若`const`对象的值直到运行时才能确定，则应该在头文件中声明，在源文件中定义。此时`const`变量的声明和定义前都应该添加`extern`关键字。

  ```c++
  // file_1.cc defines and initializes a const that is accessible to other files
  extern const int bufSize = fcn();
  // file_1.h
  extern const int bufSize;   // same bufSize as defined in file_1.cc
  ```

### const的引用（References to const）

把引用绑定在`const`对象上即为对常量的引用（reference to const）。对常量的引用不能被用作修改它所绑定的对象。

```c++
const int ci = 1024;
const int &r1 = ci;     // ok: both reference and underlying object are const
r1 = 42;        // error: r1 is a reference to const
int &r2 = ci;   // error: non const reference to a const object
```

大部分情况下，引用的类型要和与之绑定的对象严格匹配。但是有两个例外：

- 初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可。

  ```c++
  int i = 42;
  const int &r1 = i;      // we can bind a const int& to a plain int object
  const int &r2 = 42;     // ok: r1 is a reference to const
  const int &r3 = r1 * 2;     // ok: r3 is a reference to const
  int &r4 = r * 2;        // error: r4 is a plain, non const reference
  ```

- 允许为一个常量引用绑定非常量的对象、字面值或者一般表达式。

  ```c++
  double dval = 3.14;
  const int &ri = dval;
  ```

### 指针和const（Pointers and const）

指向常量的指针（pointer to const）不能用于修改其所指向的对象。常量对象的地址只能使用指向常量的指针来存放，但是指向常量的指针可以指向一个非常量对象。

```c++
const double pi = 3.14;     // pi is const; its value may not be changed
double *ptr = &pi;          // error: ptr is a plain pointer
const double *cptr = &pi;   // ok: cptr may point to a double that is const
*cptr = 42;         // error: cannot assign to *cptr
double dval = 3.14; // dval is a double; its value can be changed
cptr = &dval;       // ok: but can't change dval through cptr
```

定义语句中把`*`放在`const`之前用来说明指针本身是一个常量，常量指针（const pointer）必须初始化。

```c++
int errNumb = 0;
int *const curErr = &errNumb;   // curErr will always point to errNumb
const double pi = 3.14159;
const double *const pip = &pi;  // pip is a const pointer to a const object
```

指针本身是常量并不代表不能通过指针修改其所指向的对象的值，能否这样做完全依赖于其指向对象的类型。 

### 顶层const（Top-Level const）

顶层`const`表示指针本身是个常量，底层`const`（low-level const）表示指针所指的对象是一个常量。指针类型既可以是顶层`const`也可以是底层`const`。

```c++
int i = 0;
int *const p1 = &i;     // we can't change the value of p1; const is top-level
const int ci = 42;      // we cannot change ci; const is top-level
const int *p2 = &ci;    // we can change p2; const is low-level
const int *const p3 = p2; // right-most const is top-level, left-most is not
const int &r = ci;      // const in reference types is always low-level
```

当执行拷贝操作时，常量是顶层`const`还是底层`const`区别明显：

- 顶层`const`没有影响。拷贝操作不会改变被拷贝对象的值，因此拷入和拷出的对象是否是常量无关紧要。

  ```c++
  i = ci;     // ok: copying the value of ci; top-level const in ci is ignored
  p2 = p3;    // ok: pointed-to type matches; top-level const in p3 is ignored
  ```

- 拷入和拷出的对象必须具有相同的底层`const`资格。或者两个对象的数据类型可以相互转换。一般来说，非常量可以转换成常量，反之则不行。

  ```c++
  int *p = p3;    // error: p3 has a low-level const but p doesn't
  p2 = p3;        // ok: p2 has the same low-level const qualification as p3
  p2 = &i;        // ok: we can convert int* to const int*
  int &r = ci;    // error: can't bind an ordinary int& to a const int object
  const int &r2 = i;  // ok: can bind const int& to plain int
  ```

### constexpr和常量表达式（constexpr and Constant Expressions）

常量表达式（constant expressions）指值不会改变并且在编译过程就能得到计算结果的表达式。

一个对象是否为常量表达式由它的数据类型和初始值共同决定。

```c++
const int max_files = 20;           // max_files is a constant expression
const int limit = max_files + 1;    // limit is a constant expression
int staff_size = 27;        // staff_size is not a constant expression
const int sz = get_size();  // sz is not a constant expression
```

C++11允许将变量声明为`constexpr`类型以便由编译器来验证变量的值是否是一个常量表达式。

```c++
constexpr int mf = 20;          // 20 is a constant expression
constexpr int limit = mf + 1;   // mf + 1 is a constant expression
constexpr int sz = size();      // ok only if size is a constexpr function
```

指针和引用都能定义成`constexpr`，但是初始值受到严格限制。`constexpr`指针的初始值必须是0、`nullptr`或者是存储在某个固定地址中的对象。

函数体内定义的普通变量一般并非存放在固定地址中，因此`constexpr`指针不能指向这样的变量。相反，函数体外定义的变量地址固定不变，可以用来初始化`constexpr`指针。

在`constexpr`声明中如果定义了一个指针，限定符`constexpr`仅对指针本身有效，与指针所指的对象无关。`constexpr`把它所定义的对象置为了顶层`const`。

```c++
constexpr int *p = nullptr;     // p是指向int的const指针
constexpr int i = 0;
constexpr const int *cp = &i;   // cp是指向const int的const指针
```

`const`和`constexpr`限定的值都是常量。但`constexpr`对象的值必须在编译期间确定，而`const`对象的值可以延迟到运行期间确定。

建议使用`constexpr`修饰表示数组大小的对象，因为数组的大小必须在编译期间确定且不能改变。

## 处理类型（Dealing with Types）

### 类型别名（Type Aliases）

类型别名是某种类型的同义词，传统方法是使用关键字`typedef`定义类型别名。

```c++
typedef double wages;   // wages is a synonym for double
typedef wages base, *p; // base is a synonym for double, p for double*
```

C++11使用关键字`using`进行别名声明（alias declaration），作用是把等号左侧的名字规定成等号右侧类型的别名。

```c++
using SI = Sales_item; // SI is a synonym for Sales_item
```

### auto类型说明符（The auto Type Specifier）

C++11新增`auto`类型说明符，能让编译器自动分析表达式所属的类型。`auto`定义的变量必须有初始值。

```c++
// the type of item is deduced from the type of the result of adding val1 and val2
auto item = val1 + val2;    // item initialized to the result of val1 + val2
```

编译器推断出来的`auto`类型有时和初始值的类型并不完全一样。

- 当引用被用作初始值时，编译器以引用对象的类型作为`auto`的类型。

  ```c++
  int i = 0, &r = i;
  auto a = r;     // a is an int (r is an alias for i, which has type int)
  ```

- `auto`一般会忽略顶层`const`。 

  ```c++
  const int ci = i, &cr = ci;
  auto b = ci;    // b is an int (top-level const in ci is dropped)
  auto c = cr;    // c is an int (cr is an alias for ci whose const is top-level)
  auto d = &i;    // d is an int*(& of an int object is int*)
  auto e = &ci;   // e is const int*(& of a const object is low-level const)
  ```

  如果希望推断出的`auto`类型是一个顶层`const`，需要显式指定`const auto`。

  ```C++
  const auto f = ci;  // deduced type of ci is int; f has type const int
  ```


设置类型为`auto`的引用时，原来的初始化规则仍然适用，初始值中的顶层常量属性仍然保留。

```c++
auto &g = ci;   // g is a const int& that is bound to ci
auto &h = 42;   // error: we can't bind a plain reference to a literal
const auto &j = 42;     // ok: we can bind a const reference to a literal
```

### decltype类型指示符（The decltype Type Specifier）

C++11新增`decltype`类型指示符，作用是选择并返回操作数的数据类型，此过程中编译器不实际计算表达式的值。

```c++
decltype(f()) sum = x;  // sum has whatever type f returns
```

`decltype`处理顶层`const`和引用的方式与`auto`有些不同，如果`decltype`使用的表达式是一个变量，则`decltype`返回该变量的类型（包括顶层`const`和引用）。

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0;     // x has type const int
decltype(cj) y = x;     // y has type const int& and is bound to x
decltype(cj) z;     // error: z is a reference and must be initialized
```

如果`decltype`使用的表达式不是一个变量，则`decltype`返回表达式结果对应的类型。如果表达式的内容是解引用操作，则`decltype`将得到引用类型。如果`decltype`使用的是一个不加括号的变量，则得到的结果就是该变量的类型；如果给变量加上了一层或多层括号，则`decltype`会得到引用类型，因为变量是一种可以作为赋值语句左值的特殊表达式。

`decltype((var))`的结果永远是引用，而`decltype(var)`的结果只有当*var*本身是一个引用时才会是引用。

## 自定义数据结构（Defining Our Own Data Structures）

C++11规定可以为类的数据成员（data member）提供一个类内初始值（in-class initializer）。创建对象时，类内初始值将用于初始化数据成员，没有初始值的成员将被默认初始化。

类内初始值不能使用圆括号。

类定义的最后应该加上分号。

头文件（header file）通常包含那些只能被定义一次的实体，如类、`const`和`constexpr`变量。

头文件一旦改变，相关的源文件必须重新编译以获取更新之后的声明。

头文件保护符（header guard）依赖于预处理变量（preprocessor variable）。预处理变量有两种状态：已定义和未定义。`#define`指令把一个名字设定为预处理变量。`#ifdef`指令当且仅当变量已定义时为真，`#ifndef`指令当且仅当变量未定义时为真。一旦检查结果为真，则执行后续操作直至遇到`#endif`指令为止。

```c++
#ifndef SALES_DATA_H
#define SALES_DATA_H
#include <string>
struct Sales_data 
{
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
#endif
```

在高级版本的IDE环境中，可以直接使用`#pragma once`命令来防止头文件的重复包含。

预处理变量无视C++语言中关于作用域的规则。

整个程序中的预处理变量，包括头文件保护符必须唯一。预处理变量的名字一般均为大写。

头文件即使目前还没有被包含在任何其他头文件中，也应该设置保护符。