# 第16章 模板与泛型编程

## 定义模板（Defining a Template）

### 函数模板（Function Templates）

函数模板可以用来生成针对特定类型的函数版本。

模板定义以关键字`template`开始，后跟一个模板参数列表（template parameter list）。模板参数列表以尖括号`<>`包围，内含用逗号分隔的一个或多个模板参数（template parameter）。

```c++
template <typename T>
int compare(const T &v1, const T &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
```

定义模板时，模板参数列表不能为空。

模板参数表示在类或函数定义中用到的类型或值。当使用模板时，需要显式或隐式地指定模板实参（template argument），并将其绑定到模板参数上。

使用函数模板时，编译器用推断出的模板参数来实例化（instantiate）一个特定版本的函数，这些生成的函数通常被称为模板的实例（instantiation）。

```c++
// instantiates int compare(const int&, const int&)
cout << compare(1, 0) << endl;    // T is int
// instantiates int compare(const vector<int>&, const vector<int>&)
vector<int> vec1{1, 2, 3}, vec2{4, 5, 6};
cout << compare(vec1, vec2) << endl;    // T is vector<int>
```

模板类型参数（type parameter）可以用来指定函数的返回类型或参数类型，以及在函数体内用于变量声明和类型转换。类型参数前必须使用关键字`class`或`typename`。

```c++
// ok: same type used for the return type and parameter
template <typename T>
T foo(T* p)
{
    T tmp = *p; // tmp will have the type to which p points
    // ...
    return tmp;
}

// error: must precede U with either typename or class
template <typename T, U> T calc(const T&, const U&);
// ok: no distinction between typename and class in a template parameter list
template <typename T, class U> calc (const T&, const U&);
```

建议使用typename而不是class来指定模板类型参数，这样更加直观。

模板非类型参数（nontype parameter）需要用特定的类型名来指定，表示一个值而非一个类型。非类型参数可以是整型、指向对象或函数类型的指针或左值引用。

```c++
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M])
{
    return strcmp(p1, p2);
}

int compare(const char (&p1)[3], const char (&p2)[4]);
```

绑定到整型非类型参数的实参必须是一个常量表达式。绑定到指针或引用非类型参数的实参必须具有静态的生存期，不能用普通局部变量或动态对象作为指针或引用非类型参数的实参。

函数模板也可以声明为inline或constexpr的，说明符放在模板参数列表之后，返回类型之前。

```c++
// ok: inline specifier follows the template parameter list
template <typename T> inline T min(const T&, const T&);
// error: incorrect placement of the inline specifier
inline template <typename T> T min(const T&, const T&);
```

模板程序应该尽量减少对实参类型的要求。

```c++
// expected comparison
if (v1 < v2) return -1;
if (v1 > v2) return 1;
return 0;

// version of compare that will be correct even if used on pointers
template <typename T>
int compare(const T &v1, const T &v2)
{
    if (less<T>()(v1, v2)) return -1;
    if (less<T>()(v2, v1)) return 1;
    return 0;
}
```

只有当模板的一个特定版本被实例化时，编译器才会生成代码。此时编译器需要掌握生成代码所需的信息，因此函数模板和类模板成员函数的定义通常放在头文件中。

使用模板时，所有不依赖于模板参数的名字都必须是可见的，这是由模板的设计者来保证的。模板设计者应该提供一个头文件，包含模板定义以及在类模板或成员定义中用到的所有名字的声明。

调用者负责保证传递给模板的实参能正确支持模板所要求的操作。

### 类模板（Class Templates）

使用一个类模板时，必须提供显式模板实参（explicit template argument）列表，编译器使用这些模板实参来实例化出特定的类。

```c++
template <typename T>
class Blob
{
public:
    Blob();
    Blob(std::initializer_list<T> il);
    void push_back(const T &t) { data->push_back(t); }
    void push_back(T &&t) { data->push_back(std::move(t)); }
    // ...
    
private:
    std::shared_ptr<std::vector<T>> data;
};

Blob<int> ia;   // empty Blob<int>
Blob<int> ia2 = { 0, 1, 2, 3, 4 };    // Blob<int> with five elements
// these definitions instantiate two distinct Blob types
Blob<string> names;     // Blob that holds strings
Blob<double> prices;    // different element type
```

一个类模板的每个实例都形成一个独立的类，相互之间没有关联。

如果一个类模板中的代码使用了另一个模板，通常不会将一个实际类型（或值）的名字用作其模板实参，而是将模板自己的参数用作被使用模板的实参。

类模板的成员函数具有和类模板相同的模板参数，因此定义在类模板外的成员函数必须以关键字template开始，后跟类模板参数列表。

```c++
template <typename T>
ret-type Blob<T>::member-name(parm-list)
```

默认情况下，一个类模板的成员函数只有当程序用到它时才进行实例化。

在类模板自己的作用域内，可以直接使用模板名而不用提供模板实参。

```c++
template <typename T>
class BlobPtr
{
public:
    // 类模板作用域内不需要写成BlobPtr<T>形式
    BlobPtr& operator++();
}

// 类外定义时需要提供模板实参
template <typename T>
BlobPtr<T>& BlobPtr<T>::operator++()
{
    // 进入类模板作用域
    BlobPtr Ret = *this;
}
```

当一个类包含一个友元声明时，类与友元各自是否是模板并无关联。如果一个类模板包含一个非模板友元，则友元可以访问所有类模板实例。如果友元自身是模板，则类可以给所有友元模板实例授予访问权限，也可以只授权给特定实例。

- 一对一友元关系

  为了引用模板的一个特定实例，必须首先声明模板自身。模板声明包括模板参数列表。

  ```c++
  // forward declarations needed for friend declarations in Blob
  template <typename> class BlobPtr;
  template <typename> class Blob;    // needed for parameters in operator==
  
  template <typename T>
  bool operator==(const Blob<T>&, const Blob<T>&);
  
  template <typename T>
  class Blob
  {
      // each instantiation of Blob grants access to the version of
      // BlobPtr and the equality operator instantiated with the same type
      friend class BlobPtr<T>;
      friend bool operator==<T>(const Blob<T>&, const Blob<T>&);
  };
  ```

- 通用和特定的模板友元关系

  为了让模板的所有实例成为友元，友元声明中必须使用与类模板本身不同的模板参数。

  ```c++
  // forward declaration necessary to befriend a specific instantiation of a template
  template <typename T> class Pal;
  
  class C
  { // C is an ordinary, nontemplate class
      friend class Pal<C>;    // Pal instantiated with class C is a friend to C
      // all instances of Pal2 are friends to C;
      // no forward declaration required when we befriend all instantiations
      template <typename T> friend class Pal2;
  };
  
  template <typename T>
  class C2
  { // C2 is itself a class template
      // each instantiation of C2 has the same instance of Pal as a friend
      friend class Pal<T>;    // a template declaration for Pal must be in scope
      // all instances of Pal2 are friends of each instance of C2, prior declaration needed
      template <typename X> friend class Pal2;
      // Pal3 is a nontemplate class that is a friend of every instance of C2
      friend class Pal3;      // prior declaration for Pal3 not needed
  };
  ```

C++11中，类模板可以将模板类型参数声明为友元。

```c++
template <typename Type>
class Bar
{
    friend Type;   // grants access to the type used to instantiate Bar
    // ...
};
```

C++11允许使用using为类模板定义类型别名。

```c++
template<typename T> using twin = pair<T, T>;
twin<string> authors;   // authors is a pair<string, string>
```

类模板可以声明static成员。

```c++
template <typename T>
class Foo
{
public:
    static std::size_t count() { return ctr; }
    
private:
    static std::size_t ctr;
};

// instantiates static members Foo<string>::ctr and Foo<string>::count
Foo<string> fs;
// all three objects share the same Foo<int>::ctr and Foo<int>::count members
Foo<int> fi, fi2, fi3;
```

类模板的每个实例都有一个独有的static对象，而每个static成员必须有且只有一个定义。因此与定义模板的成员函数类似，static成员也应该定义成模板。

```c++
template <typename T>
size_t Foo<T>::ctr = 0;    // define and initialize ctr
```

### 模板参数（Template Parameters）

模板参数遵循普通的作用域规则。与其他任何名字一样，模板参数会隐藏外层作用域中声明的相同名字。但是在模板内不能重用模板参数名。

```c++
typedef double A;
template <typename A, typename B>
void f(A a, B b)
{
    A tmp = a;   // tmp has same type as the template parameter A, not double
    double B;    // error: redeclares template parameter B
}
```

由于模板参数名不能重用，所以一个名字在一个特定模板参数列表中只能出现一次。

与函数参数一样，声明中模板参数的名字不必与定义中的相同。

一个特定文件所需要的所有模板声明通常一起放置在文件开始位置，出现在任何使用这些模板的代码之前。

模板中的代码使用作用域运算符`::`时，编译器无法确定其访问的名字是类型还是static成员。

默认情况下，C++假定模板中通过作用域运算符访问的名字是static成员。因此，如果需要使用一个模板类型参数的类型成员，就必须使用关键字`typename`显式地告知编译器该名字是一个类型。

```c++
template <typename T>
typename T::value_type top(const T& c)
{
    if (!c.empty())
        return c.back();
    else
        return typename T::value_type();
}
```

C++11允许为函数和类模板提供默认实参。

```c++
// compare has a default template argument, less<T>
// and a default function argument, F()
template <typename T, typename F = less<T>>
int compare(const T &v1, const T &v2, F f = F())
{
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
    return 0;
}
```

如果一个类模板为其所有模板参数都提供了默认实参，在使用这些默认实参时，必须在模板名后面跟一个空尖括号对`<>`。

```c++
template <class T = int>
class Numbers
{ // by default T is int
public:
    Numbers(T v = 0): val(v) { }
    // various operations on numbers
private:
    T val;
};

Numbers<long double> lots_of_precision;
Numbers<> average_precision;    // empty <> says we want the default type
```

### 成员模板（Member Templates）

一个类（无论是普通类还是模板类）可以包含本身是模板的成员函数，这种成员被称为成员模板。成员模板不能是虚函数。

```c++
class DebugDelete
{
public:
    DebugDelete(std::ostream &s = std::cerr): os(s) { }
    // as with any function template, the type of T is deduced by the compiler
    template <typename T>
    void operator()(T *p) const
    { 
        os << "deleting unique_ptr" << std::endl;
        delete p;
    }
    
private:
    std::ostream &os;
};
```

在类模板外定义一个成员模板时，必须同时为类模板和成员模板提供模板参数列表。

```c++
template <typename T>
class Blob
{
    template <typename It>
    Blob(It b, It e);
};

template <typename T>   // type parameter for the class
template <typename It>  // type parameter for the constructor
Blob<T>::Blob(It b, It e):
    data(std::make_shared<std::vector<T>>(b, e))
    { }
```

为了实例化一个类模板的成员模板，必须同时提供类和函数模板的实参。

### 控制实例化（Controlling Instantiations）

因为模板在使用时才会进行实例化，所以相同的实例可能出现在多个对象文件中。当两个或多个独立编译的源文件使用了相同的模板，并提供了相同的模板参数时，每个文件中都会有该模板的一个实例。

在大型程序中，多个文件实例化相同模板的额外开销可能非常严重。C++11允许通过显式实例化（explicit instantiation）来避免这种开销。

显式实例化的形式如下：

```c++
extern template declaration;    // instantiation declaration
template declaration;           // instantiation definition
```

*declaration*是一个类或函数声明，其中所有模板参数已被替换为模板实参。当编译器遇到extern模板声明时，它不会在本文件中生成实例化代码。对于一个给定的实例化版本，可能有多个extern声明，但必须只有一个定义。

```c++
// templateBuild.cc
// instantiation file must provide a (nonextern) definition for every
// type and function that other files declare as extern
template int compare(const int&, const int&);
template class Blob<string>;    // instantiates all members of the class template

// Application.cc
// these template types must be instantiated elsewhere in the program
extern template class Blob<string>;
extern template int compare(const int&, const int&);
Blob<string> sa1, sa2;    // instantiation will appear elsewhere
// Blob<int> and its initializer_list constructor instantiated in this file
Blob<int> a1 = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
Blob<int> a2(a1);    // copy constructor instantiated in this file
int i = compare(a1[0], a2[0]);    // instantiation will appear elsewhere
```

当编译器遇到类模板的实例化定义时，它不清楚程序会使用哪些成员函数。和处理类模板的普通实例化不同，编译器会实例化该模板的所有成员，包括内联的成员函数。因此，用来显式实例化类模板的类型必须能用于模板的所有成员。

### 效率与灵活性（Efficiency and Flexibility）

unique_ptr在编译时绑定删除器，避免了间接调用删除器的运行时开销。shared_ptr在运行时绑定删除器，使用户重载删除器的操作更加简便。

## 模板实参推断（Template Argument Deduction）

对于函数模板，编译器通过调用的函数实参来确定其模板参数。这个过程被称作模板实参推断。

### 类型转换与模板类型参数（Conversions and Template Type Parameters）

与非模板函数一样，调用函数模板时传递的实参被用来初始化函数的形参。如果一个函数形参的类型使用了模板类型参数，则会采用特殊的初始化规则，只有有限的几种类型转换会自动地应用于这些实参。编译器通常会生成新的模板实例而不是对实参进行类型转换。

有3种类型转换可以在调用中应用于函数模板：

- 顶层const会被忽略。
- 可以将一个非const对象的引用或指针传递给一个const引用或指针形参。
- 如果函数形参不是引用类型，则可以对数组或函数类型的实参应用正常的指针转换。数组实参可以转换为指向其首元素的指针。函数实参可以转换为该函数类型的指针。

其他的类型转换，如算术转换、派生类向基类的转换以及用户定义的转换，都不能应用于函数模板。

 一个模板类型参数可以作为多个函数形参的类型。由于允许的类型转换有限，因此传递给这些形参的实参必须具有相同的类型，否则调用失败。

```c++
long lng;
compare(lng, 1024);   // error: cannot instantiate compare(long, int)
```

如果想增强函数的兼容性，可以使用两个类型参数定义函数模板。

```c++
// argument types can differ but must be compatible
template <typename A, typename B>
int flexibleCompare(const A& v1, const B& v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}

long lng;
flexibleCompare(lng, 1024);   // ok: calls flexibleCompare(long, int)
```

函数模板中使用普通类型定义的参数可以进行正常的类型转换。

```c++
template <typename T>
ostream &print(ostream &os, const T &obj)
{
    return os << obj;
}

print(cout, 42);   // instantiates print(ostream&, int)
ofstream f("output");
print(f, 10);      // uses print(ostream&, int); converts f to ostream&
```

### 函数模板显式实参（Function-Template Explicit Arguments）

某些情况下，编译器无法推断出模板实参的类型。

```c++
// T1 cannot be deduced: it doesn't appear in the function parameter list
template <typename T1, typename T2, typename T3>
T1 sum(T2, T3);
```

显式模板实参（explicit template argument）可以让用户自己控制模板的实例化。提供显式模板实参的方式与定义类模板实例的方式相同。显式模板实参在尖括号`<>`中指定，位于函数名之后，实参列表之前。

```c++
// T1 is explicitly specified; T2 and T3 are inferred from the argument types
auto val3 = sum<long long>(i, lng);   // long long sum(int, long)
```

显式模板实参按照从左到右的顺序与对应的模板参数匹配，只有尾部参数的显式模板实参才可以忽略，而且前提是它们可以从函数参数推断出来。

```c++
// poor design: users must explicitly specify all three template parameters
template <typename T1, typename T2, typename T3>
T3 alternative_sum(T2, T1);
// error: can't infer initial template parameters
auto val3 = alternative_sum<long long>(i, lng);
// ok: all three parameters are explicitly specified
auto val2 = alternative_sum<long long, int, long>(i, lng);
```

对于模板类型参数已经显式指定了的函数实参，可以进行正常的类型转换。

```c++
long lng;
compare(lng, 1024);         // error: template parameters don't match
compare<long>(lng, 1024);   // ok: instantiates compare(long, long)
compare<int>(lng, 1024);    // ok: instantiates compare(int, int)
```

### 尾置返回类型与类型转换（Trailing Return Types and Type Transformation）

由于尾置返回出现在函数列表之后，因此它可以使用函数参数来声明返回类型。

```c++
// a trailing return lets us declare the return type after the parameter list is seen
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
    // process the range
    return *beg;   // return a reference to an element from the range
}
```

标准库在头文件*type_traits*中定义了类型转换模板，这些模板常用于模板元程序设计。其中每个模板都有一个名为`type`的公有类型成员，表示一个类型。此类型与模板自身的模板类型参数相关。如果不可能（或不必要）转换模板参数，则type成员就是模板参数类型本身。

![16-1](Image/16-1.png)

使用`remove_reference`可以获得引用对象的元素类型，如果用一个引用类型实例化remove_reference，则type表示被引用的类型。因为type是一个类的类型成员，所以在模板中必须使用关键字typename来告知编译器其表示一个类型。

```c++
// must use typename to use a type member of a template parameter
template <typename It>
auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type
{
    // process the range
    return *beg;  // return a copy of an element from the range
}
```

### 函数指针和实参推断（Function Pointers and Argument Deduction）

使用函数模板初始化函数指针或为函数指针赋值时，编译器用指针的类型来推断模板实参。

```c++
template <typename T> int compare(const T&, const T&);
// pf1 points to the instantiation int compare(const int&, const int&)
int (*pf1)(const int&, const int&) = compare;
```

如果编译器不能从函数指针类型确定模板实参，则会产生错误。使用显式模板实参可以消除调用歧义。

```c++
// overloaded versions of func; each takes a different function pointer type
void func(int(*)(const string&, const string&));
void func(int(*)(const int&, const int&));
func(compare);     // error: which instantiation of compare?
// ok: explicitly specify which version of compare to instantiate
func(compare<int>);    // passing compare(const int&, const int&)
```

### 模板实参推断和引用（Template Argument Deduction and References）

当一个函数参数是模板类型参数的普通（左值）引用（形如*T&*）时，只能传递给它一个左值（如一个变量或一个返回引用类型的表达式）。*T*被推断为实参所引用的类型，如果实参是const的，则*T*也为const类型。

```c++
template <typename T> void f1(T&);    // argument must be an lvalue
// calls to f1 use the referred-to type of the argument as the template parameter type
f1(i);     // i is an int; template parameter T is int
f1(ci);    // ci is a const int; template parameter T is const int
f1(5);     // error: argument to a & parameter must be an lvalue
```

当一个函数参数是模板类型参数的常量引用（形如*const T&*）时，可以传递给它任何类型的实参。函数参数本身是const时，*T*的类型推断结果不会是const类型。const已经是函数参数类型的一部分了，因此不会再是模板参数类型的一部分。

```c++
template <typename T> void f2(const T&);    // can take an rvalue
// parameter in f2 is const &; const in the argument is irrelevant
// in each of these three calls, f2's function parameter is inferred as const int&
f2(i);     // i is an int; template parameter T is int
f2(ci);    // ci is a const int, but template parameter T is int
f2(5);     // a const & parameter can be bound to an rvalue; T is int
```

当一个函数参数是模板类型参数的右值引用（形如*T&&*）时，如果传递给它一个右值，类型推断过程类似普通左值引用函数参数的推断过程，推断出的*T*类型是该右值实参的类型。

```c++
template <typename T> void f3(T&&);
f3(42);    // argument is an rvalue of type int; template parameter T is int
```

模板参数绑定的两个例外规则：

- 如果将一个左值传递给函数的右值引用参数，且此右值引用指向模板类型参数时，编译器推断模板类型参数为实参的左值引用类型。

- 如果间接创建了一个引用的引用（通过类型别名或者模板类型参数间接定义），则这些引用会被“折叠”。右值引用的右值引用会被折叠为右值引用。其他情况下，引用都被折叠为普通左值引用。

  |          折叠前          | 折叠后 |
  | :----------------------: | :----: |
  | *T& &*、*T& &&*、*T&& &* |  *T&*  |
  |         *T&& &&*         | *T&&*  |

```c++
f3(i);    // argument is an lvalue; template parameter T is int&
f3(ci);   // argument is an lvalue; template parameter T is const int&

// invalid code, for illustration purposes only
void f3<int&>(int& &&);    // when T is int&, function parameter is int& &&
void f3<int&>(int&);       // when T is int&, function parameter collapses to int&
```

模板参数绑定的两个例外规则导致了两个结果：

- 如果一个函数参数是指向模板类型参数的右值引用，则可以传递给它任意类型的实参。
- 如果将一个左值传递给这样的参数，则函数参数被实例化为一个普通的左值引用。

当代码中涉及的类型可能是普通（非引用）类型，也可能是引用类型时，编写正确的代码就变得异常困难。

```c++
template <typename T>
void f3(T&& val)
{
    T t = val;     // copy or binding a reference?
    t = fcn(t);    // does the assignment change only t or val and t?
    if (val == t) { /* ... */ }    // always true if T is a reference type
}
```

实际编程中，模板的右值引用参数通常用于两种情况：模板转发其实参或者模板被重载。函数模板的常用重载形式如下：

```c++
template <typename T> void f(T&&);         // binds to nonconst rvalues
template <typename T> void f(const T&);    // lvalues and const rvalues
```

### 理解std::move（Understanding std::move）

std::move的定义如下：

```c++
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
}
```

std::move的工作过程：

```c++
string s1("hi!"), s2;
s2 = std::move(string("bye!"));     // ok: moving from an rvalue
s2 = std::move(s1);     // ok: but after the assigment s1 has indeterminate value
```

- 在`std::move(string("bye!"))`中传递的是右值。

- - 推断出的*T*类型为string。
  - remove_reference用string进行实例化。
  - remove_reference\<string\>的type成员是string。
  - move的返回类型是string&&。
  - move的函数参数t的类型为string&&。

- 在`std::move(s1)`中传递的是左值。

- - 推断出的*T*类型为string&。
  - remove_reference用string&进行实例化。
  - remove_reference\<string&\>的type成员是string。
  - move的返回类型是string&&。
  - move的函数参数t的类型为string& &&，会折叠成string&。

可以使用static_cast显式地将一个左值转换为一个右值引用。

### 转发（Forwarding）

某些函数需要将其一个或多个实参连同类型不变地转发给其他函数。在这种情况下，需要保持被转发实参的所有性质，包括实参的const属性以及左值/右值属性。

```c++
template <typename F, typename T1, typename T2>
void flip1(F f, T1 t1, T2 t2)
{
    f(t2, t1);
}

void f(int v1, int &v2)   // note v2 is a reference
{
    cout << v1 << " " << ++v2 << endl;
}

f(42, i);   // f changes its argument i
flip1(f, j, 42);    // f called through flip1 leaves j unchanged
```

将函数参数定义为指向模板类型参数的右值引用（形如*T&&*），通过引用折叠，可以保持翻转实参的左值/右值属性。并且引用参数（无论是左值还是右值）可以保持实参的const属性，因为在引用类型中的const是底层的。

```c++
template <typename F, typename T1, typename T2>
void flip2(F f, T1 &&t1, T2 &&t2)
{
    f(t2, t1);
}
```

函数参数与其他变量一样，都是左值表达式。所以即使是指向模板类型的右值引用参数也只能传递给接受左值引用的函数，不能传递给接受右值引用的函数。

```c++
void g(int &&i, int& j)
{
    cout << i << " " << j << endl;
}

// error: can't initialize int&& from an lvalue
flip2(g, i, 42);
```

C++11在头文件*utility*中定义了`forward`。与move不同，forward必须通过显式模板实参调用，返回该显式实参类型的右值引用。即forward\<*T*\>返回类型*T&&*。

通常情况下，可以使用forward传递定义为指向模板类型参数的右值引用函数参数。通过其返回类型上的引用折叠，forward可以保持给定实参的左值/右值属性。

```c++
template <typename Type>
intermediary(Type &&arg)
{
    finalFcn(std::forward<Type>(arg));
    // ...
}

template <typename F, typename T1, typename T2>
void flip(F f, T1 &&t1, T2 &&t2)
{
    f(std::forward<T2>(t2), std::forward<T1>(t1));
}
```

与std::move一样，对std::forward也不应该使用using声明。

## 重载与模板（Overloading and Templates）

函数模板可以被另一个模板或普通非模板函数重载。

如果重载涉及函数模板，则函数匹配规则会受到一些影响：

- 对于一个调用，其候选函数包括所有模板实参推断成功的函数模板实例。

- 候选的函数模板都是可行的，因为模板实参推断会排除任何不可行的模板。

- 和往常一样，可行函数（模板与非模板）按照类型转换（如果需要的话）来排序。但是可以用于函数模板调用的类型转换非常有限。

- 和往常一样，如果恰有一个函数提供比其他任何函数都更好的匹配，则选择此函数。但是如果多个函数都提供相同级别的匹配，则：

- - 如果同级别的函数中只有一个是非模板函数，则选择此函数。
  - 如果同级别的函数中没有非模板函数，而有多个函数模板，且其中一个模板比其他模板更特例化，则选择此模板。
  - 否则该调用有歧义。

通常，如果使用了一个没有声明的函数，代码将无法编译。但对于重载函数模板的函数而言，如果编译器可以从模板实例化出与调用匹配的版本，则缺少的声明就不再重要了。

```c++
template <typename T> string debug_rep(const T &t);
template <typename T> string debug_rep(T *p);
// the following declaration must be in scope
// for the definition of debug_rep(char*) to do the right thing
string debug_rep(const string &);
string debug_rep(char *p)
{
    // if the declaration for the version that takes a const string& is not in scope
    // the return will call debug_rep(const T&) with T instantiated to string
    return debug_rep(string(p));
}
```

在定义任何函数之前，应该声明所有重载的函数版本。这样编译器就不会因为未遇到你希望调用的函数而实例化一个并非你所需要的版本。

## 可变参数模板（Variadic Templates）

可变参数模板指可以接受可变数量参数的模板函数或模板类。可变数量的参数被称为参数包（parameter pack），分为两种：

- 模板参数包（template parameter pack），表示零个或多个模板参数。
- 函数参数包（function parameter pack），表示零个或多个函数参数。

用一个省略号`…`来指出模板参数或函数参数表示一个包。在一个模板参数列表中，`class…`或`typename…`指出接下来的参数表示零个或多个类型的列表；一个类型名后面跟一个省略号表示零个或多个给定类型的非类型参数列表。在函数参数列表中，如果一个参数的类型是模板参数包，则此参数也是函数参数包。

```C++
// Args is a template parameter pack; rest is a function parameter pack
// Args represents zero or more template type parameters
// rest represents zero or more function parameters
template <typename T, typename... Args>
void foo(const T &t, const Args& ... rest);
```

对于一个可变参数模板，编译器会推断模板参数类型和参数数量。

可以使用`sizeof…`运算符获取参数包中的元素数量。类似sizeof，sizeof…也返回一个常量表达式，而且不会对其实参求值。

```c++
template<typename ... Args>
void g(Args ... args)
{
    cout << sizeof...(Args) << endl;    // number of type parameters
    cout << sizeof...(args) << endl;    // number of function parameters
}
```

### 编写可变参数函数模板（Writing a Variadic Function Template）

可变参数函数通常是递归的，第一步调用参数包中的第一个实参，然后用剩余实参调用自身。为了终止递归，还需要定义一个非可变参数的函数。

```c++
// function to end the recursion and print the last element
// this function must be declared before the variadic version of print is defined
template<typename T>
ostream &print(ostream &os, const T &t)
{
    return os << t;   // no separator after the last element in the pack
}

// this version of print will be called for all but the last element in the pack
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args&... rest)
{
    os << t << ", ";    // print the first argument
    return print(os, rest...);   // recursive call; print the other arguments
}
```

|         Call          |      t      |   rest...   |
| :-------------------: | :---------: | :---------: |
| print(cout, i, s, 42) |      i      |    s, 42    |
|  print(cout, s, 42)   |      s      |     42      |
|    print(cout, 42)    | nonvariadic | nonvariadic |

### 包扩展（Pack Expansion）

对于一个参数包，除了获取其大小外，唯一能对它做的事情就是扩展。当扩展一个包时，需要提供用于每个扩展元素的模式（pattern）。扩展一个包就是将其分解为构成的元素，对每个元素应用模式，获得扩展后的列表。通过在模式右边添加一个省略号`…`来触发扩展操作。

包扩展工作过程：

```c++
template <typename T, typename... Args>
ostream& print(ostream &os, const T &t, const Args&... rest)   // expand Args
{
    os << t << ", ";
    return print(os, rest...);   // expand rest
}
```

- 第一个扩展操作扩展模板参数包，为print生成函数参数列表。编译器将模式*const Args&*应用到模板参数包*Args*中的每个元素上。因此该模式的扩展结果是一个以逗号分隔的零个或多个类型的列表，每个类型都形如*const type&*。

  ```c++
  print(cout, i, s, 42);   // two parameters in the pack
  ostream& print(ostream&, const int&, const string&, const int&);
  ```

- 第二个扩展操作扩展函数参数包，模式是函数参数包的名字。扩展结果是一个由包中元素组成、以逗号分隔的列表。

  ```c++
  print(os, s, 42);
  ```

扩展操作中的模式会独立地应用于包中的每个元素。

```c++
// call debug_rep on each argument in the call to print
template <typename... Args>
ostream &errorMsg(ostream &os, const Args&... rest)
{
    // print(os, debug_rep(a1), debug_rep(a2), ..., debug_rep(an)
    return print(os, debug_rep(rest)...);
}

// passes the pack to debug_rep; print(os, debug_rep(a1, a2, ..., an))
print(os, debug_rep(rest...));   // error: no matching function to call
```

### 转发参数包（Forwarding Parameter Packs）

在C++11中，可以组合使用可变参数模板和forward机制来编写函数，实现将其实参不变地传递给其他函数。

```c++
// fun has zero or more parameters each of which is
// an rvalue reference to a template parameter type
template<typename... Args>
void fun(Args&&... args)    // expands Args as a list of rvalue references
{
    // the argument to work expands both Args and args
    work(std::forward<Args>(args)...);
}
```

## 模板特例化（Template Specializations）

在某些情况下，通用模板的定义对特定类型是不合适的，可能编译失败或者操作不正确。如果不希望或不能使用模板版本时，可以定义类或函数模板的特例化版本。一个特例化版本就是模板的一个独立定义，其中的一个或多个模板参数被指定为特定类型。

```c++
// first version; can compare any two types
template <typename T> int compare(const T&, const T&);
// second version to handle string literals
template<size_t N, size_t M>
int compare(const char (&)[N], const char (&)[M]);

const char *p1 = "hi", *p2 = "mom";
compare(p1, p2);        // calls the first template
compare("hi", "mom");   // calls the template with two nontype parameters

// special version of compare to handle pointers to character arrays
template <>
int compare(const char* const &p1, const char* const &p2)
{
    return strcmp(p1, p2);
}
```

特例化一个函数模板时，必须为模板中的每个模板参数都提供实参。为了指明我们正在实例化一个模板，应该在关键字template后面添加一个空尖括号对`<>`。

特例化版本的参数类型必须与一个先前声明的模板中对应的类型相匹配。

定义特例化函数版本本质上是接管编译器的工作，为模板的一个特殊实例提供了定义。特例化并非重载，因此不影响函数匹配。

将一个特殊版本的函数定义为特例化模板还是独立的非模板函数会影响到重载函数匹配。

模板特例化遵循普通作用域规则。为了特例化一个模板，原模板的声明必须在作用域中。而使用模板实例时，也必须先包含特例化版本的声明。

通常，模板及其特例化版本应该声明在同一个头文件中。所有同名模板的声明放在文件开头，后面是这些模板的特例化版本。

类模板也可以特例化。与函数模板不同，类模板的特例化不必为所有模板参数提供实参，可以只指定一部分模板参数。一个类模板的部分特例化（partial specialization）版本本身还是一个模板，用户使用时必须为那些未指定的模板参数提供实参。

只能部分特例化类模板，不能部分特例化函数模板。

由于类模板的部分特例化版本是一个模板，所以需要定义模板参数。对于每个未完全确定类型的模板参数，在特例化版本的模板参数列表中都有一项与之对应。在类名之后，需要为特例化的模板参数指定实参，这些实参位于模板名之后的尖括号中，与原始模板中的参数按位置相对应。

```c++
// 通用版本
template <typename T>
struct remove_reference
{
    typedef T type;
};

// 部分特例化版本
template <typename T>
struct remove_reference<T &>   // 左值引用
{
    typedef T type;
};

template <typename T>
struct remove_reference<T &&>  // 右值引用
{
    typedef T type;
};
```

类模板部分特例化版本的模板参数列表是原始模板参数列表的一个子集或特例化版本。

可以只特例化类模板的指定成员函数，而不用特例化整个模板。

```c++
template <typename T>
struct Foo
{
    Foo(const T &t = T()): mem(t) { }
    void Bar() { /* ... */ }
    T mem;
    // other members of Foo
};

template<>      // we're specializing a template
void Foo<int>::Bar()    // we're specializing the Bar member of Foo<int>
{
    // do whatever specialized processing that applies to ints
}

Foo<string> fs;     // instantiates Foo<string>::Foo()
fs.Bar();    // instantiates Foo<string>::Bar()
Foo<int> fi;    // instantiates Foo<int>::Foo()
fi.Bar();    // uses our specialization of Foo<int>::Bar()
```

