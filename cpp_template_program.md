# 模板参数

## 类型参数

使用模板参数可以为不同类型的输入编写同一套函数，让编译器自己根据模板函数自己生成对应参数的函数，以加法函数为例：

```C++
template <typename T>
T addFunction(const T& a, const T& b) {
    return a + b;
}
```

这样的话，我们就可以在主函数去调用该函数，输入不同类型的参数，生成不同的函数

```C++
int main(int argc, char** argv) {
    std::cout << addFunction(1, 2) << std::endl;  // int 版本的函数
    std::cout << addFunction(1.1, 2.1) << std::endl;  // float 版本的函数
}
```

同时，除了C++内置的类型，我们也自定义一些对象，生成对应的函数版本。当然，必须保证在生成对应函数时不会出错，在本例中，保证我们自定义的类型支持加法运算。

```C++
class Account {
public:
    double money; 
    Acount(double _money = 0) :money(_money){}
    Account operator+(const Account& account) {
        Account res;
        res.money = this.money + account.money;
        return res;
    }
};

int main(int argc, char** argv) {
    Account account1(100.0);
    Account account2(111.111);
    Account account3 = addFunction(account1, account2); // 生成 Account版本的函数
}

```

另外，模板除了可以应用在函数上，也可以应用在类上，以一个自定义数组为例。

```C++
template <class T> // class 和 typename 在这里每区别
class ArrayList {
public:
    ArrayList(const size_t iCap = 10):cap(iCap){ptr = static_cast<T*>(malloc(cap * sizeof(T)); size = 0;}
    T& at(size_t x) {
        if (size <= x) throw "index out of range" ;
        return ptr[x];
    }
    // ...
private:
    T* ptr;
    size_t size;
    size_t cap;
};
```

在应用的时候可以单独使用，也可以嵌套使用。

```C++
int main(int argc, char** argv) {
    ArrayList<int> a1;
    ArrayList<ArrayList<int>> a2;
}
```



## 非类型参数

除了定义类型参数外，我们也可以定义非类型模板参数，即数值。

下面以一个二次函数为例

```C++
template<typename T, int a = 0, int b = 0, int c = 0>
T quadraticEquation(const T& x) {
    return a*x^2 + b*x + c;
}
int main(int argc, char** argv) {
    std::cout << quadraticEquation<int,1,2,3>(1) << std::endl;
}
```

但是，一般来说，模板的非类型参数只能是**整型常量（包括枚举），或者指向外部链接的指针（包括函数指针，类的成员函数指针）**。 到目前为止还不支持浮点数，对于字符串常量也不支持，但是可以支持具有外部链接的字符串常量指针。



同时非类型参数也可以应用在类模板中，同样以自定义数组为例。

```C++
template <class T, size_t DEFAULT = 10> // class 和 typename 在这里每区别
class ArrayList {
public:
    ArrayList():cap(iCap){ptr = static_cast<T*>(malloc(DEFAULT * sizeof(T)); size = 0;}
    T& at(size_t x) {
        if (size <= x) throw "index out of range" ;
        return ptr[x];
    }
    // ...
private:
    T* ptr;
    size_t size;
    size_t cap;
};
                                                
                                                
int main(int argc, char** argv) {
    ArrayList<int, 10> a1;
    ArrayList<ArrayList<int, 10>, 30> a2;
}
```



## 模板模板参数



# 模板特化和偏特化



## 模板特化

有时为了需要，针对特定的类型，需要对模板进行特化，也就是所谓的特殊处理。比如有以下的一段代码：

```C++
template <typename T>
T addFunction(const T& a, const T& b) {
    return a + b;
}

template<>
int addFunction<int>(const int& a, const int& b) {
    std::cout << "specialization" << std::endl;
    return a + b;
}
```

```C++
int main(int argc, char** argv) {
    std::cout << addFunction(1,1) << std::endl; // int 的 特化版本
}
```

类模板同理：

```C++
template <class T> 
class ArrayList {
public:
    ArrayList(const size_t iCap = 10):cap(iCap){ptr = static_cast<T*>(malloc(cap * sizeof(T)); size = 0;}
    T& at(size_t x) {
        if (size <= x) throw "index out of range" ;
        return ptr[x];
    }
    // ...
private:
    T* ptr;
    size_t size;
    size_t cap;
};
                                                                      
template <> // class 和 typename 在这里每区别
class ArrayList<int> {
public:
    ArrayList(const size_t iCap = 10):cap(iCap){ptr = static_cast<int*>(malloc(cap * sizeof(int)); size = 0;}
    int& at(size_t x) {
        if (size <= x) throw "index out of range" ;
        return ptr[x];
    }
    // ...
private:
    int* ptr;
    size_t size;
    size_t cap;
};
                                                                        
int main(int argc, char** argv) {
    ArrayList<int> a1; // int 特化版本
}
```



## 偏特化

### 类模板的偏特化

**模板偏特化（template partitial specialization）**是模板特化的一种特殊情况，指指定模板参数而非全部模板参数，或者模板参数的一部分而非全部特性，也称为模板部分特化。也就是说，针对template参数更进一步的条件限制所设计出来的一个特化版本，而其本身仍为**templatized**.

模板的偏特化是不支持函数模板的，如下例：

```C++
template <typename T>
T addFunction(const T& a, const T& b) {
    return a + b;
}

template <typename T>
T addFunction<T*>(const T* a, const T* b) {
    std::cout << "Point Version" << std::endl;
    return (*a) + (*b);
}
```

> 编译报错： error: non-class, non-variable partial specialization ‘addFunction<T*>’ is not allowed

如果真的有实现函数偏特化的需求，可以使用类模板重载()方式，如下所示：

```C++
template <typename T>
class addClass{
public:
    addClass(const T& a, const T& b) : m_a(a), m_b(b) {}
    T operator()(){
        return m_a + m_b;
    }
private:
    T m_a;
    T m_b;
};


template <typename T>
class addClass<T*>{
public:
    addClass(const T* a, const T* b) : m_a(a), m_b(b) {}
    T operator()(){
        std::cout << "partial specialization" << std::endl;
        return (*m_a) + (*m_b);
    }
private:
    T* m_a;
    T* m_b;
};

int main(int argc, char** argv) {
	std::cout << addClass<int*>(&a, &b)() << std::endl; // 偏特化版本
}
```

### 函数重载实现函数偏特化

C++标准虽然不支持函数模板的偏特化，但函数的重载显然是支持的。使用标签分发(Tag Dispatch)的方案就是通过函数实现不同的函数重载实现，根据不同实参类型选择具体的函数实现，以达到函数模板偏特化的实现。

```C++
// 标签分发demo
struct NormalVersionTag {};
struct IntPartialVersionTag {};

template<class T> struct TagDispatchTrait { 
    using Tag = NormalVersionTag; 
};

template<> 
struct TagDispatchTrait<int> { 
    using Tag = IntPartialVersionTag; 
};

template <typename A, typename B>
inline void internal_f(A a, B b, NormalVersionTag) {
    std::cout << "Normal version." << std::endl;
}

template <typename A, typename B>
inline void internal_f(A a, B b, IntPartialVersionTag) {
    std::cout << "Partial version." << std::endl;
}

template <typename A, typename B>
void f(A a, B b) {
    return internal_f(a, b, typename TagDispatchTrait<B>::Tag {}); // typename 另外一个作用为：使用嵌套依赖类型(nested depended name)
}
int main(int argc, char** argv) {
    // 测试代码
	int a = 10;
	double b = 12;
	f(a, b);
	f(a, a);  // int 的偏特化
}
```

---

对于模板、模板的特化和模板的偏特化都存在的情况下，编译器在编译阶段进行匹配时，是如何抉择的呢？从哲学的角度来说，应该先照顾最特殊的，然后才是次特殊的，最后才是最普通的。编译器进行抉择也是尊从的这个道理。

# 模板元编程

模板元编程（Template Metaprogramming，TMP）是**编写生成或操纵程序的程序**，也是一种复杂且功能强大的编程范式（Programming Paradigm）。C++模板给C++提供了元编程的能力，但大部分用户对 C++ 模板的使用并不是很频繁，大致限于泛型编程，在一些系统级的代码，尤其是对通用性、性能要求极高的基础库（如 STL、Boost）几乎不可避免在大量地使用 C++ 模板以及模板元编程。. 模版元编程完全不同于普通的运行期程序，因为模版元程序的执行完全是在编译期，并且模版元程序操纵的数据不能是运行时变量，只能是编译期常量，不可修改。

## 可变模板参数

C++11的新特性--**可变模版参数（variadic templates）**是C++11新增的最强大的特性之一，它对参数进行了高度泛化，它能表示*0到任意个数、任意类型的参数*。 相比C++98/03，类模版和函数模版中只能含固定数量的模版参数，可变模版参数无疑是一个巨大的改进。

基本形式：

```C++
template <typename ...Args>
void function(Args... args){}
```

下面是一个打印参数个数的例子

```C++
template <typename... Args>
void printNumOfArgs(Args... args) {
    std::cout << sizeof...(args) << std::endl;
}
```

下面以多数之和的例子介绍两种可变模板参数的展开方式

### 递归的方式展开可变模板参数

通过递归的方式本质上是通过模板的特化和模板实例化的优先级完成的，在下例中，addCore函数如果输入参数只有一个的话，就会匹配第一个模板，如果有多个参数的话，就会匹配第二个模板。其中第一个参数映射到T，其余参数打包之可变参数包中。

```C++
template <typename T>
T addCore(const T& a) {
    return a;
}

template <typename T, typename... Args>
T addCore(const T& a, Args... args) {
    return a + addCore(args...);
}

template <typename T, typename... Args>
T addCoupleNum(Args... args) {
    return addCore(args...);
}
```

在模板元编程中，我们可以使用**if constexpr(C++17标准)**作简单的条件判断以便简化递归，如下例所示：

```C++
template <typename T, typename... Args>
T addCore(const T& a, Args... args) {
    if constexpr (sizeof...(args) == 0) {
        return a
    } else {
    	return a + addCore(args...);
    }
}

template <typename T, typename... Args>
T addCoupleNum(Args... args) {
    return addCore(args...);
}
```

### 通过折叠表达式展开可变模板参数

C++17的折叠表达式总共有四种，即一元右展开，一元左展开，二元右展开， 二元左展开

> 一元右折叠(unary right fold)

**( pack op ... )**  展开之后变为 **E1 op (... op (EN-1 op EN))**

> 一元左折叠(unary left fold)

**( ... op pack ) **展开之后变为  **((E1 op E2) op ...) op EN**

> 二元右折叠(binary right fold)

**( pack op ... op init )**展开之后变为 **E1 op (... op (EN−1 op (EN op Init)))**

> 二元左折叠(binary left fold)

**( init op ... op pack )**展开之后变为 **(((Init op E1) op E2) op ...) op EN**

语法形式中的**op**代表运算符，**pack**代表参数包，**init**代表初始值。

折叠表达式支持 32 个操作符: `+`, `-`, `*`, `/`, `%`, `^`, `&`, `|`, `=`, `<`,
`>`, `<<`, `>>`, `+=`, `-=`, `*=`, `/=`, `%=`, `^=`, `&=`, `|=`, `<<=`,
`>>=`,`==`, `!=`, `<=`, `>=`, `&&`, `||`, `,`, `.*`, `->*`. 

初始值在右边的为右折叠，展开之后从右边开始折叠。而初始值在左边的为左折叠，展开之后从左边开始折叠。

不指定初始值的为一元折叠表达式，而指定初始值的为二元折叠表达式。

当一元折叠表达式中的参数包为空时，只有三个运算符（&& || 以及逗号）有缺省值，其中&&的缺省值为true,||的缺省值为false，逗号的缺省值为void()。

同样是多数之和的例子

```C++
// 多数之和
template <typename T, typename... Args>
T addCoupleNum(Args... args) {
    return (args + ... + 0); //二元右展开 , 括号不要忘记！！！
}
// 多数绝对值的和
template <typename T, typename... Args>
T addCoupleNumAbs(Args... args) {
    return (abs(args) + ... + 0); //二元右展开 , 括号不要忘记！！！
}
```

## 模板递归

上述在可变参数模板中已经介绍过模板递归了，下面再以求n！的例子加深影响。在模板元编程中，所有的计算都是在编译器完成的，所以使用的变量都是编译期可以确定的，内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。

```C++
template<unsigned int n>
class Factorial {
public:
    // C++标准只允许static constant intergral或者枚举类型的数据在类内进行初始化
    static const unsigned int value = n * Factorial<n-1>::value;
};

template<>
class Factorial<0> {
public:
    static const unsigned int value = 1;
};

template<>
class Factorial<1> {
public:
    static const unsigned int value = 1 * Factorial<0>::value;
};
```

**if constexpr**简化

> C++标准只允许static constant intergral或者枚举类型的数据在类内进行初始化

```C++
template<unsigned int n>
class Factorial {
public:
    unsigned int value = 0;
    Factorial(){
    	if constexpr (n == 0 || n == 1) {
        	value = 1;
    	} else {
            Factorial<n-1> f;
        	value = n * f.value;
    	}
    }   
};
```

> 缺点：无法在编译器完成计算

## 完美转发

std::forward被称为**完美转发**，它的作用是保持原来的`值`属性不变。啥意思呢？通俗的讲就是，如果原来的值是左值，经std::forward处理后该值还是左值；如果原来的值是右值，经std::forward处理后它还是右值。如下所示：

```C++
template<typename T>
void print(T & t){
    std::cout << "左值" << std::endl;
}

template<typename T>
void print(T && t){
    std::cout << "右值" << std::endl;
}

template<typename T>
void testForward(T && v){
    print(v);
    print(std::forward<T>(v));
    print(std::move(v));
}

int main(int argc, char * argv[])
{
    testForward(1);

    std::cout << "======================" << std::endl;

    int x = 1;
    testFoward(x);
}
```

同时完美转发，减少拷贝的性能开销，见下例：

```C++
template <typename T>
void handleValue(T&& t) {std::cout << t << std::endl;}

void handleeeImp() {cout << "calls succeeful" << endl;}

template<typename T, typename... Types>
void handleeeImp(T&& t, Types&&... args) {
    handleValue(std::forward<T>(t)); // 完美转发，如果T类的所占空间很大，这么操作的话，就可以省去拷贝的开销
    handleeeImp(std::forward<Types>(args)...); // 折叠表达式，无操作符
}

template<typename... Types>
void handle(Types&&... args) {
    std::cout << "arg's size = " << sizeof ...(args) << std::endl;
    handleeeImp(args...);
}

```

## 混入模式 - Mixin类

混入模式是元模板编程中，灵活利用继承完成功能组合的一种方式，见下述代码：

```C++
class ReadInterface{
public:
    void Read() {
        std::cout << "Read" << std::endl;
    }
};
class WriteInterface{
public:
    void Write() {
        std::cout << "Write" << std::endl;
    }
};
class ZipInterface{
public:
    void Zip() {
        std::cout << "Zip" << std::endl;
    }
};
class PrintInterface{
public:
    void Print() {
        std::cout << "Print" << std::endl;
    }
};
// 可灵活的决定要继承的父类
template<typename ... Mixins>
class MyInterface : public Mixins... 
{
public:
    MyInterface() {}
    MyInterface(const Mixins& ... mixin) : Mixins(mixin) ... {}
};


// 继承 ReadInterface、WriteInterface
MyInterface<ReadInterface, WriteInterface> interface;
```

**Mixin**是一种设计思想，用于将相互独立的类的功能组合在一起，以一种安全的方式来模拟多重继承。而c++中mixin的实现通常采用Template Parameters as Base Classes的方法，既实现了多个不交叉的类的功能组合，又避免了多重继承的问题

## 整数序列

std::index_sequence、 index_sequence_for

```C++
template<typename Tuple, size_t... Indices>
void tuple_print_helper(const Tuple& t, 
                        std::index_sequence<Indices...>){
    ((std::cout << std::get<Indices>(t) << std::endl), ...); 
}

template <typename... Args>
void tuple_print(const std::tuple<Args...>& t) {
    tuple_print_helper(t, std::index_sequence_for<Args...>()); // 生成对应的整数队列
}
auto tp = std::make_tuple(24, "234", "Friden", 1.23);
tuple_print(tp);
```



## trait

- type_traits是C++11提供的模板元基础库。
- type_traits可实现在编译期计算、判断、转换、查询等等功能。
- type_traits提供了编译期的true和false。

**is_void**

**is_integral**

**is_reference**

**is_object**

**is_floating_point**

**is_pointer**

**is_const**

**is_same**

**is_base_of**

**is_convertible**

```C++
#include <iostream>
#include <type_traits>
using namespace std;

int main()
{
	// 判断int和const int类型
	std::cout << "int: " << std::is_const<int>::value << std::endl;
	std::cout << "const int:" << std::is_const<const int>::value << std::endl;

	// 判断类型是否相同
	std::cout << std::is_same<int, int>::value << "\n";
	std::cout << std::is_same<int, unsigned int>::value << "\n";

	// 添加/移除const
	std::cout << std::is_same<const int, add_const<int>::type>::value << std::endl;
	std::cout << std::is_same<int, remove_const<const int>::type>::value << std::endl;

	//添加引用
	std::cout << std::is_same<int&, add_lvalue_reference<int>::type>::value << std::endl;
	std::cout << std::is_same<int&&, add_rvalue_reference<int>::type>::value << std::endl;

	// 取公共类型
	typedef std::common_type<unsigned char, short, int>::type NumericType;
	std::cout << std::is_same<int, NumericType>::value << std::endl;
	return 0;
}
```

## enable_if

 **std::enable_if**，它利用SFINAE(substitude failuer is not an error)特性，根据条件选择重载函数

`enable_if` 利用了模板匹配的技巧和`struct`结构, 巧妙的将条件匹配分割成两种情况,
一种是true的情况: 为结构绑定一个type
一种是false的情况: 采取留空策略

```C++
// 原型 ，在判断条件B为真时，函数才有效
template<bool B, class T = void> struct enable_if;
// 例子
typename std::endable_if<std::is_arithmetic<T>::value, T>::type foo(T t)
{
    return t;
}

auto numberic = foo(1);    // 返回整数1
auto str = foo("test");    // 编译失败

// 例 相当于函数重载，通过enable_if和条件判断式，将入参分为两类(数字和非数字)
template<class T>
typename std::enable_if<std::is_arithmetic<T>::value, int>::type fool(T t)
{
    cout<< t << endl;
    return 0;
}

template<class T>
typename std::enable_if<!std::is_arithmetic<T>::value, int>::type fool(T& t) foo(T t)
{
    cout << t << endl;
    return 1;
}

```



**enable_if_t**它基于模板的 [SFINAE](https://www.jianshu.com/p/dae1976239da) 和 [匿名类型参数](https://www.jianshu.com/p/3deec1ac6430) 的基础概念上进行了简洁且完美的封装(落地).`enable_if_t` 强制使用 `enable_if` 的 `::type` 来触发 `SFINAE` 规则, 如果失败则跳过当前匹配进入下一个匹配.

```C++
class IsDoable {
public:
    void doit() const {
        std::cout << "parent call doit " << std::endl;
    }
};

class Derived : public IsDoable {
public:
    void doit() const {
        std::cout << "derived call doit " << std::endl;
    }
};


template <typename T>  
std::enable_if_t<std::is_base_of_v<IsDoable, T>, void> call_doit(const T& t) {  // 只有继承 IsDoable 的类型才会使模板实例化
    t.doit();
}


template <typename T> 
std::enable_if_t<!std::is_base_of_v<IsDoable, T>, void> call_doit(const T& t) { // 只有不继承 IsDoable 的类型才会使模板实例化
    std::cout << "you can not have doit interdace " << std::endl;
}
```

