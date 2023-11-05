# 第7章 类

类的基本思想是数据抽象（data abstraction）和封装（encapsulation）。数据抽象是一种依赖于接口（interface）和实现（implementation）分离的编程及设计技术。类的接口包括用户所能执行的操作；类的实现包括类的数据成员、负责接口实现的函数体以及其他私有函数。

## 定义抽象数据类型（Defining Abstract Data Types）

### 设计`Sales_data`类（Designing the `Sales_data` Class）

类的用户是程序员，而非应用程序的最终使用者。

### 定义改进的`Sales_data`类（Defining the Revised `Sales_data` Class）

成员函数（member function）的声明必须在类的内部，定义则既可以在类的内部也可以在类的外部。定义在类内部的函数是隐式的内联函数。

```c++
struct Sales_data
{
    // new members: operations on Sales_data objects
    std::string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data&);
    double avg_price() const;

    // data members
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

成员函数通过一个名为`this`的隐式额外参数来访问调用它的对象。`this`参数是一个常量指针，被初始化为调用该函数的对象地址。在函数体内可以显式使用`this`指针。

```c++
total.isbn()
// pseudo-code illustration of how a call to a member function is translated
Sales_data::isbn(&total)

std::string isbn() const { return this->bookNo; }
std::string isbn() const { return bookNo; }
```

默认情况下，`this`的类型是指向类类型非常量版本的常量指针。`this`也遵循初始化规则，所以默认不能把`this`绑定到一个常量对象上，即不能在常量对象上调用普通的成员函数。

C++允许在成员函数的参数列表后面添加关键字`const`，表示`this`是一个指向常量的指针。使用关键字`const`的成员函数被称作常量成员函数（const member function）。

```c++
// pseudo-code illustration of how the implicit this pointer is used
// this code is illegal: we may not explicitly define the this pointer ourselves
// note that this is a pointer to const because isbn is a const member
std::string Sales_data::isbn(const Sales_data *const this)
{
    return this->isbn;
}
```

常量对象和指向常量对象的引用或指针都只能调用常量成员函数。

类本身就是一个作用域，成员函数的定义嵌套在类的作用域之内。编译器处理类时，会先编译成员声明，再编译成员函数体（如果有的话），因此成员函数可以随意使用类的其他成员而无须在意这些成员的出现顺序。

在类的外部定义成员函数时，成员函数的定义必须与它的声明相匹配。如果成员函数被声明为常量成员函数，那么它的定义也必须在参数列表后面指定`const`属性。同时，类外部定义的成员名字必须包含它所属的类名。

```c++
double Sales_data::avg_price() const
{
    if (units_sold)
        return revenue / units_sold;
    else
        return 0;
}
```

可以定义返回`this`对象的成员函数。

```c++
Sales_data& Sales_data::combine(const Sales_data &rhs)
{
    units_sold += rhs.units_sold;   // add the members of rhs into
    revenue += rhs.revenue;     // the members of 'this' object
    return *this;       // return the object on which the function was called
}
```

### 定义类相关的非成员函数（Defining Nonmember Class-Related Functions）

类的作者通常会定义一些辅助函数，尽管这些函数从概念上来说属于类接口的组成部分，但实际上它们并不属于类本身。

```c++
// input transactions contain ISBN, number of copies sold, and sales price
istream &read(istream &is, Sales_data &item)
{
    double price = 0;
    is >> item.bookNo >> item.units_sold >> price;
    item.revenue = price * item.units_sold;
    return is;
}

ostream &print(ostream &os, const Sales_data &item)
{
    os << item.isbn() << " " << item.units_sold << " "
        << item.revenue << " " << item.avg_price();
    return os;
}
```

如果非成员函数是类接口的组成部分，则这些函数的声明应该与类放在同一个头文件中。

一般来说，执行输出任务的函数应该尽量减少对格式的控制。

### 构造函数（Constructors）

类通过一个或几个特殊的成员函数来控制其对象的初始化操作，这些函数被称作构造函数。只要类的对象被创建，就会执行构造函数。

构造函数的名字和类名相同，没有返回类型，且不能被声明为`const`函数。构造函数在`const`对象的构造过程中可以向其写值。

```c++
struct Sales_data
{
    // constructors added
    Sales_data() = default;
    Sales_data(const std::string &s): bookNo(s) { }
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    Sales_data(std::istream &);
    // other members as before
};
```

类通过默认构造函数（default constructor）来控制默认初始化过程，默认构造函数无须任何实参。

如果类没有显式地定义构造函数，则编译器会为类隐式地定义一个默认构造函数，该构造函数也被称为合成的默认构造函数（synthesized default constructor）。对于大多数类来说，合成的默认构造函数初始化数据成员的规则如下：

- 如果存在类内初始值，则用它来初始化成员。

- 否则默认初始化该成员。

某些类不能依赖于合成的默认构造函数。

- 只有当类没有声明任何构造函数时，编译器才会自动生成默认构造函数。一旦类定义了其他构造函数，那么除非再显式地定义一个默认的构造函数，否则类将没有默认构造函数。

- 如果类包含内置类型或者复合类型的成员，则只有当这些成员全部存在类内初始值时，这个类才适合使用合成的默认构造函数。否则用户在创建类的对象时就可能得到未定义的值。

- 编译器不能为某些类合成默认构造函数。例如类中包含一个其他类类型的成员，且该类型没有默认构造函数，那么编译器将无法初始化该成员。

在C++11中，如果类需要默认的函数行为，可以通过在参数列表后面添加`=default`来要求编译器生成构造函数。其中`=default`既可以和函数声明一起出现在类的内部，也可以作为定义出现在类的外部。和其他函数一样，如果`=default`在类的内部，则默认构造函数是内联的。

```c++
Sales_data() = default;
```
构造函数初始值列表（constructor initializer list）负责为新创建对象的一个或几个数据成员赋初始值。形式是每个成员名字后面紧跟括号括起来的（或者在花括号内的）成员初始值，不同成员的初始值通过逗号分隔。

```c++
Sales_data(const std::string &s): bookNo(s) { }
Sales_data(const std::string &s, unsigned n, double p):
    bookNo(s), units_sold(n), revenue(p*n) { }
```
当某个数据成员被构造函数初始值列表忽略时，它会以与合成默认构造函数相同的方式隐式初始化。

```c++
// has the same behavior as the original constructor defined above
Sales_data(const std::string &s):
    bookNo(s), units_sold(0), revenue(0) { }
```

构造函数不应该轻易覆盖掉类内初始值，除非新值与原值不同。如果编译器不支持类内初始值，则所有构造函数都应该显式初始化每个内置类型的成员。

### 拷贝、赋值和析构（Copy、Assignment，and Destruction）

编译器能合成拷贝、赋值和析构函数，但是对于某些类来说合成的版本无法正常工作。特别是当类需要分配类对象之外的资源时，合成的版本通常会失效。

## 访问控制与封装（Access Control and Encapsulation）

使用访问说明符（access specifier）可以加强类的封装性：

- 定义在`public`说明符之后的成员在整个程序内都可以被访问。`public`成员定义类的接口。

- 定义在`private`说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问。`private`部分封装了类的实现细节。

```c++
class Sales_data
{
public: // access specifier added
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
    bookNo(s), units_sold(n), revenue(p*n) { }
    Sales_data(const std::string &s): bookNo(s) { }
    Sales_data(std::istream&);
    std::string isbn() const { return bookNo; }
    Sales_data &combine(const Sales_data&);

private: // access specifier added
    double avg_price() const { return units_sold ? revenue/units_sold : 0; }
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

一个类可以包含零或多个访问说明符，每个访问说明符指定了接下来的成员的访问级别，其有效范围到出现下一个访问说明符或类的结尾处为止。

使用关键字`struct`定义类时，定义在第一个访问说明符之前的成员是`public`的；而使用关键字`class`时，这些成员是`private`的。二者唯一的区别就是默认访问权限不同。

### 友元（Friends）

类可以允许其他类或函数访问它的非公有成员，方法是使用关键字`friend`将其他类或函数声明为它的友元。

```C++
class Sales_data
{
    // friend declarations for nonmember Sales_data operations added
    friend Sales_data add(const Sales_data&, const Sales_data&);
    friend std::istream &read(std::istream&, Sales_data&);
    friend std::ostream &print(std::ostream&, const Sales_data&);

    // other members and access specifiers as before
public:
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
    bookNo(s), units_sold(n), revenue(p*n) { }
    Sales_data(const std::string &s): bookNo(s) { }
    Sales_data(std::istream&);
    std::string isbn() const { return bookNo; }
    Sales_data &combine(const Sales_data&);

private:
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

// declarations for nonmember parts of the Sales_data interface
Sales_data add(const Sales_data&, const Sales_data&);
std::istream &read(std::istream&, Sales_data&);
std::ostream &print(std::ostream&, const Sales_data&);
```

友元声明只能出现在类定义的内部，具体位置不限。友元不是类的成员，也不受它所在区域访问级别的约束。

通常情况下，最好在类定义开始或结束前的位置集中声明友元。

封装的好处：

- 确保用户代码不会无意间破坏封装对象的状态。

- 被封装的类的具体实现细节可以随时改变，而无须调整用户级别的代码。

友元声明仅仅指定了访问权限，而并非一个通常意义上的函数声明。如果希望类的用户能调用某个友元函数，就必须在友元声明之外再专门对函数进行一次声明（部分编译器没有该限制）。

为了使友元对类的用户可见，通常会把友元的声明（类的外部）与类本身放在同一个头文件中。

## 类的其他特性（Additional Class Features）

### 类成员再探（Class Members Revisited）

由类定义的类型名字和其他成员一样存在访问限制，可以是`public`或`private`中的一种。

```c++
class Screen
{
public:
    // alternative way to declare a type member using a type alias
    using pos = std::string::size_type;
    // other members as before
};
```

与普通成员不同，用来定义类型的成员必须先定义后使用。类型成员通常位于类起始处。

定义在类内部的成员函数是自动内联的。

如果需要显式声明内联成员函数，建议只在类外部定义的位置说明`inline`。

`inline`成员函数该与类定义在同一个头文件中。

使用关键字`mutable`可以声明可变数据成员（mutable data member）。可变数据成员永远不会是`const`的，即使它在`const`对象内。因此`const`成员函数可以修改可变成员的值。

```c++
class Screen
{
public:
    void some_member() const;
private:
    mutable size_t access_ctr;  // may change even in a const object
    // other members as before
};

void Screen::some_member() const
{
    ++access_ctr;   // keep a count of the calls to any member function
    // whatever other work this member needs to do
}
```

提供类内初始值时，必须使用`=`或花括号形式。

### 返回`*this`的成员函数（Functions That Return `*this`）

`const`成员函数如果以引用形式返回`*this`，则返回类型是常量引用。

通过区分成员函数是否为`const`的，可以对其进行重载。在常量对象上只能调用`const`版本的函数；在非常量对象上，尽管两个版本都能调用，但会选择非常量版本。

```c++
class Screen
{
public:
    // display overloaded on whether the object is const or not
    Screen &display(std::ostream &os)
    { do_display(os); return *this; }
    const Screen &display(std::ostream &os) const
    { do_display(os); return *this; }

private:
    // function to do the work of displaying a Screen
    void do_display(std::ostream &os) const
    { os << contents; }
    // other members as before
};

Screen myScreen(5,3);
const Screen blank(5, 3);
myScreen.set('#').display(cout);    // calls non const version
blank.display(cout);    // calls const version
```

### 类类型（Class Types）

每个类定义了唯一的类型。即使两个类的成员列表完全一致，它们也是不同的类型。

可以仅仅声明一个类而暂时不定义它。这种声明被称作前向声明（forward declaration），用于引入类的名字。在类声明之后定义之前都是一个不完全类型（incomplete type）。

```c++
class Screen;   // declaration of the Screen class
```

可以定义指向不完全类型的指针或引用，也可以声明（不能定义）以不完全类型作为参数或返回类型的函数。

只有当类全部完成后才算被定义，所以一个类的成员类型不能是该类本身。但是一旦类的名字出现，就可以被认为是声明过了，因此类可以包含指向它自身类型的引用或指针。

```c++
class Link_screen
{
    Screen window;
    Link_screen *next;
    Link_screen *prev;
};
```

### 友元再探（Friendship Revisited）

除了普通函数，类还可以把其他类或其他类的成员函数声明为友元。友元类的成员函数可以访问此类包括非公有成员在内的所有成员。

```c++
class Screen
{
    // Window_mgr members can access the private parts of class Screen
    friend class Window_mgr;
    // ... rest of the Screen class
};
```

友元函数可以直接定义在类的内部，这种函数是隐式内联的。但是必须在类外部提供相应声明令函数可见。

```c++
struct X
{
    friend void f() { /* friend function can be defined in the class body */ }
    X() { f(); }   // error: no declaration for f
    void g();
    void h();
};

void X::g() { return f(); }     // error: f hasn't been declared
void f();   // declares the function defined inside X
void X::h() { return f(); }     // ok: declaration for f is now in scope
```

友元关系不存在传递性。

把其他类的成员函数声明为友元时，必须明确指定该函数所属的类名。

```c++
class Screen
{
    // Window_mgr::clear must have been declared before class Screen
    friend void Window_mgr::clear(ScreenIndex);
    // ... rest of the Screen class
};
```

如果类想把一组重载函数声明为友元，需要对这组函数中的每一个分别声明。

## 类的作用域（Class Scope）

当成员函数定义在类外时，返回类型中使用的名字位于类的作用域之外，此时返回类型必须指明它是哪个类的成员。

```c++
class Window_mgr
{
public:
    // add a Screen to the window and returns its index
    ScreenIndex addScreen(const Screen&);
    // other members as before
};

// return type is seen before we're in the scope of Window_mgr
Window_mgr::ScreenIndex Window_mgr::addScreen(const Screen &s)
{
    screens.push_back(s);
    return screens.size() - 1;
}
```

### 名字查找与作用域（Name Lookup and Class Scope）

成员函数体直到整个类可见后才会被处理，因此它能使用类中定义的任何名字。

声明中使用的名字，包括返回类型或参数列表，都必须确保使用前可见。

如果类的成员使用了外层作用域的某个名字，而该名字表示一种类型，则类不能在之后重新定义该名字。

```c++
typedef double Money;
class Account
{
public:
    Money balance() { return bal; } // uses Money from the outer scop
private:
    typedef double Money; // error: cannot redefine Money
    Money bal;
    // ...
};
```

类型名定义通常出现在类起始处，这样能确保所有使用该类型的成员都位于类型名定义之后。

成员函数中名字的解析顺序：

- 在成员函数内查找该名字的声明，只有在函数使用之前出现的声明才会被考虑。

- 如果在成员函数内没有找到，则会在类内继续查找，这时会考虑类的所有成员。

- 如果类内也没有找到，会在成员函数定义之前的作用域查找。

```c++
// it is generally a bad idea to use the same name for a parameter and a member
int height;   // defines a name subsequently used inside Screen
class Screen
{
public:
    typedef std::string::size_type pos;
    void dummy_fcn(pos height)
    {
        cursor = width * height;  // which height? the parameter
    }

private:
    pos cursor = 0;
    pos height = 0, width = 0;
};
```

可以通过作用域运算符`::`或显式`this`指针来强制访问被隐藏的类成员。

```c++
// bad practice: names local to member functions shouldn't hide member names
void Screen::dummy_fcn(pos height)
{
    cursor = width * this->height;  // member height
    // alternative way to indicate the member
    cursor = width * Screen::height;  // member height
}

// good practice: don't use a member name for a parameter or other local variable
void Screen::dummy_fcn(pos ht)
{
    cursor = width * height;  // member height
}
```

## 构造函数再探（Constructors Revisited）

### 构造函数初始值列表（Constructor Initializer List）

如果没有在构造函数初始值列表中显式初始化成员，该成员会在构造函数体之前执行默认初始化。

如果成员是`const`、引用，或者是某种未定义默认构造函数的类类型，必须在初始值列表中将其初始化。

```c++
class ConstRef
{
public:
    ConstRef(int ii);
private:
    int i;
    const int ci;
    int &ri;
};

// ok: explicitly initialize reference and const members
ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(i) { }
```

最好令构造函数初始值的顺序与成员声明的顺序一致，并且尽量避免使用某些成员初始化其他成员。

如果一个构造函数为所有参数都提供了默认实参，则它实际上也定义了默认构造函数。

### 委托构造函数（Delegating Constructors）

C++11扩展了构造函数初始值功能，可以定义委托构造函数。委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程。

```c++
class Sales_data
{
public:
    // defines the default constructor as well as one that takes a string argument
    Sales_data(std::string s = ""): bookNo(s) { }
    // remaining constructors unchanged
    Sales_data(std::string s, unsigned cnt, double rev):
        bookNo(s), units_sold(cnt), revenue(rev*cnt) { }
    Sales_data(std::istream &is) { read(is, *this); }
    // remaining members as before
}
```

### 默认构造函数的作用（The Role of the Default Constructor）

当对象被默认初始化或值初始化时会自动执行默认构造函数。

默认初始化的发生情况：

- 在块作用域内不使用初始值定义非静态变量或数组。

- 类本身含有类类型的成员且使用合成默认构造函数。

- 类类型的成员没有在构造函数初始值列表中显式初始化。

值初始化的发生情况：

- 数组初始化时提供的初始值数量少于数组大小。

- 不使用初始值定义局部静态变量。

- 通过`T()`形式（`T`为类型）的表达式显式地请求值初始化。

类必须包含一个默认构造函数。

如果想定义一个使用默认构造函数进行初始化的对象，应该去掉对象名后的空括号对。

```c++
Sales_data obj();   // oops! declares a function, not an object
Sales_data obj2;    // ok: obj2 is an object, not a function
```

### 隐式的类类型转换（Implicit Class-Type Conversions）

如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制。这种构造函数被称为转换构造函数（converting constructor）。

```c++
string null_book = "9-999-99999-9";
// constructs a temporary Sales_data object
// with units_sold and revenue equal to 0 and bookNo equal to null_book
item.combine(null_book);
```

编译器只会自动执行一步类型转换。

```c++
// error: requires two user-defined conversions:
//   (1) convert "9-999-99999-9" to string
//   (2) convert that (temporary) string to Sales_data
item.combine("9-999-99999-9");
// ok: explicit conversion to string, implicit conversion to Sales_data
item.combine(string("9-999-99999-9"));
// ok: implicit conversion to string, explicit conversion to Sales_data
item.combine(Sales_data("9-999-99999-9"));
```

在要求隐式转换的程序上下文中，可以通过将构造函数声明为`explicit`的加以阻止。

```c++
class Sales_data
{
public:
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    explicit Sales_data(const std::string &s): bookNo(s) { }
    explicit Sales_data(std::istream&);
    // remaining members as before
};
```

`explicit`关键字只对接受一个实参的构造函数有效。

只能在类内声明构造函数时使用`explicit`关键字，在类外定义时不能重复。

执行拷贝初始化时（使用`=`）会发生隐式转换，所以`explicit`构造函数只能用于直接初始化。

```c++
Sales_data item1 (null_book);   // ok: direct initialization
// error: cannot use the copy form of initialization with an explicit constructor
Sales_data item2 = null_book;
```

可以使用`explicit`构造函数显式地强制转换类型。

```c++
// ok: the argument is an explicitly constructed Sales_data object
item.combine(Sales_data(null_book));
// ok: static_cast can use an explicit constructor
item.combine(static_cast<Sales_data>(cin));
```

### 聚合类（Aggregate Classes）

聚合类满足如下条件：

- 所有成员都是`public`的。

- 没有定义任何构造函数。

- 没有类内初始值。

- 没有基类。

- 没有虚函数。

```c++
struct Data
{
    int ival;
    string s;
};
```

可以使用一个用花括号包围的成员初始值列表初始化聚合类的数据成员。初始值顺序必须与声明顺序一致。如果初始值列表中的元素个数少于类的成员个数，则靠后的成员被值初始化。

```c++
// val1.ival = 0; val1.s = string("Anna")
Data val1 = { 0, "Anna" };
```

### 字面值常量类（Literal Classes）

数据成员都是字面值类型的聚合类是字面值常量类。或者一个类不是聚合类，但符合下列条件，则也是字面值常量类：

- 数据成员都是字面值类型。

- 类至少含有一个`constexpr`构造函数。

- 如果数据成员含有类内初始值，则内置类型成员的初始值必须是常量表达式。如果成员属于类类型，则初始值必须使用成员自己的`constexpr`构造函数。

- 类必须使用析构函数的默认定义。

`constexpr`构造函数用于生成`constexpr`对象以及`constexpr`函数的参数或返回类型。

`constexpr`构造函数必须初始化所有数据成员，初始值使用`constexpr`构造函数或常量表达式。

## 类的静态成员（static Class Members）

使用关键字`static`可以声明类的静态成员。静态成员存在于任何对象之外，对象中不包含与静态成员相关的数据。

```c++
class Account
{
public:
    void calculate() { amount += amount * interestRate; }
    static double rate() { return interestRate; }
    static void rate(double);

private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```

由于静态成员不与任何对象绑定，因此静态成员函数不能声明为`const`的，也不能在静态成员函数内使用`this`指针。

用户代码可以使用作用域运算符访问静态成员，也可以通过类对象、引用或指针访问。类的成员函数可以直接访问静态成员。

```c++
double r;
r = Account::rate(); // access a static member using the scope operator

Account ac1;
Account *ac2 = &ac1;
// equivalent ways to call the static member rate function
r = ac1.rate(); // through an Account object or reference
r = ac2->rate(); // through a pointer to an Account object

class Account
{
public:
    void calculate() { amount += amount * interestRate; }
private:
    static double interestRate;
    // remaining members as before
};
```

在类外部定义静态成员时，不能重复`static`关键字，其只能用于类内部的声明语句。

由于静态数据成员不属于类的任何一个对象，因此它们并不是在创建类对象时被定义的。通常情况下，不应该在类内部初始化静态成员。而必须在类外部定义并初始化每个静态成员。一个静态成员只能被定义一次。一旦它被定义，就会一直存在于程序的整个生命周期中。

```c++
// define and initialize a static class member
double Account::interestRate = initRate();
```

建议把静态数据成员的定义与其他非内联函数的定义放在同一个源文件中，这样可以确保对象只被定义一次。

尽管在通常情况下，不应该在类内部初始化静态成员。但是可以为静态成员提供`const`整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的`constexpr`。初始值必须是常量表达式。

```c++
class Account
{
public:
    static double rate() { return interestRate; }
    static void rate(double);
private:
    static constexpr int period = 30;  // period is a constant
    double daily_tbl[period];
};
```

静态数据成员的类型可以是它所属的类类型。

```c++
class Bar
{
    static Bar mem1;   // ok: static member can have incomplete type
    Bar *mem2;    // ok: pointer member can have incomplete type
    Bar mem3;   // error: data members must have complete type
}
```

可以使用静态成员作为函数的默认实参。

```c++
class Screen
{
public:
    // bkground refers to the static member
    // declared later in the class definition
    Screen& clear(char = bkground);
private:
    static const char bkground;
};
```