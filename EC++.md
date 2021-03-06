### 条款01：视C++为一个语言联邦

C++是个多重泛型编程语言，一个同时支持**过程式编程、面向对象编程、函数形式、泛型形式、元编程形式的语言**。

元编程：**元编程** *(metaprogramming)* 通过操作 **程序实体** *(program entity)*，在 **编译时** *(compile time)* 计算出 **运行时** *(runtime)* 需要的常数、类型、代码的方法。

一般的编程是通过直接编写 **程序** *(program)*，通过编译器 **编译** *(compile)*，产生目标代码，并用于 **运行时** 执行。与普通的编程不同，元编程则是借助语言提供的 **模板** *(template)* 机制，通过编译器 **推导** *(deduce)*，在 **编译时** 生成程序。元编程经过编译器推导得到的程序，再进一步通过编译器编译，产生最终的目标代码。

因此，元编程又被成为 **两级编程** *(two-level programming)*，**生成式编程** *(generative programming)* 或 **模板元编程** *(template metaprogramming)*。

（我的理解就是，一般的程序都是编写好了然后编译运行；但是元编程我只写了怎么生成代码，编译的时候会又编译器生成这些代码，然后才执行；也就是说，前者所有程序都是我写的，后者我写了一部分，编译器写了一部分。**其实模板就是这样的一类，模板不产生任何函数定义，除非你进行显式实例化。编译的时候需要用到的时候，编译器才会依照模板产生这样的函数。**）

> C++ = C语言的超集 + 面向对象编程 + 模板编程 + 标准库

C++较C语言新增了**模板、异常、重载、面向对象、STL...**；



### 条款2：尽量以const、enum、inline替换#define

①宏的作用就是**直接替换**，例如你定义了一个宏

#define   a   10

a  =  20；

由于编译的时候a变成了10，因此报错的时候大概会提示**右值10不能被赋值，这时候你并不会直到原来是a出错了，只知道替换之后的10出错了，debug会比较难；**

解决方法是，使用一个**const常量代替宏**：

const int  a = 10；

②**指针中的const**：

const int \*p表示这是一个**指向const int的指针**，所以我不能通过指针改变它的值，但是我可以指向别的const int 类型的变量；

int \* const p表示这是一个**指向int的const指针**，所以我不能改变指针指向谁，但是我可以用过指针改变变量的值；

③**类成员变量既是static又是const。**这种情况下你可以直接在**头文件赋值**，但是并没有定义，实现文件还是要提一下，但是不用赋值：

class XX{

public：

static const int a = 5；//头文件

}

const int XX：：a；//实现文件

当然头文件没赋值就必须在实现文件赋值了。

④**重视enum**。首先enum跟#define比较像，你可以对一个const整型取地址，但是不能读#define和enum取地址，此外，enum也能实现一个替换的功能。而且，enum广泛应用在模板元编程中。

⑤#define宏定义函数十分危险，例如：

#define  CALL_WITH_MAX（a，b）  f（（a）>（b）？（a）：（b））

int a = 5，b = 0；

CALL_WITH_MAX（++a，b）

就会变成f（（++a）>（b）？（++a）：（b）），如果**++a比b要大，就会返回++a，因此a自增了两次！如果++a比b小，返回b，这时候正常。**

解决方法是使用模板。

**建议：**

1. 对于单纯常量，最好用const代替#define；
2. 对于形似函数的宏，最好用inline替换#define；

### 条款3：尽可能使用const

①**为什么使用返回值为const的函数？**因为如果返回的不是const，有可能会被错误地使用而不知道：

if（a \* b = c）其实就是想比较a\*b和c，但是漏了一个=。此时c的值赋予了a\*b，所以没有报错，而且c不是0还会判断为true。

②**const用于类成员函数**

将const用于成员函数，目的是**确认该成员函数可作用于const对象。**好处在于：

1. 使得函数容易理解，直到哪些函数可以修改对象内容而哪些不可以；
2. 使得操作const对象成为可能，**这可以提升C++程序效率；**

我们直到，**仅仅相差一个const的可以重载，**最常见的就是[]运算符：

const char& operator=（size_t pos）const；

char& operator=（size_t pos）；

如果这个时候对象a是const的，你就不能完成以下操作：

a[0] = 'x'，因为返回的是const，不能改变；但是如果对象a不是const，就可以用来赋值了；

③我们直到，如果成员函数后面带有const，说明这个函数内不能修改成员变量，如果我非要呢？

可以使用mutable关键字，如下：

class XX{

private：

mutable int a；//即使在const成员函数里，依然可以修改a的值；

}

**建议：**

1. const可以用于对象、函数参数、函数返回类型、成员函数本体等，声明为const有助于编译的时候发现错误；
2. 有时候概念上的常量更重要，计算机的const并不完全周密；
3. **只能在non-const调用const函数避免代码重复！**



### 条款4：确定对象使用前已被初始化

①最核心的观点就是**构造函数要使用初始化列表，**在构造函数体内进行的**都是赋值**，赋值都是对象创建以后发生的；**初始化是对象创建之前发生的；**

**建议**：

1. 对内置对象进行手工初始化，因为C++对于这种对象不承诺初始化；

2. 使用成员初始化列表；

3. 使用类内static的好处在于，我可以通过成员函数返回这个变量来确保这个变量已经存在，全局的static在多文件项目中并不保证；

   

### 条款5：了解C++默默编写了什么函数

①每一个类，C++都会默认提供**构造函数、析构函数、拷贝构造函数、赋值运算符、移动拷贝构造函数（C++11）、移动赋值运算符（C++11）；**

class XX；

XX a；

XX b（a）；//拷贝构造函数

XX c；

c = a；//赋值运算符

区分赋值运算符和赋值构造函数的最简单方法就是看**这一行有没有生成新的类对象，有就是复制，没有就是赋值。**



### 条款6：如果不需要自动生成，主动拒绝

①如果要禁止某一个类的复制构造函数和赋值运算符，可以参考boost：：noncopyable

或者将赋值构造函数和赋值运算符写在类成员函数里，但是不去实现。



### 条款7：为多态基类声明virtual析构函数

第一，如果没有virtual关键字，会根据调用该方法的**指针或者引用的类型**来决定使用哪一个方法；如果有virtual关键字，就会根据调用的对象的类型来决定使用哪一个方法；

第二，A是基类B是派生类，构造的时候先构造A再构造B，析构的时候先析构B再析构A；为了保证析构顺序正确，基类的析构函数应该是virtual的；

第三，声明一个virtual析构函数就说明，这个类可以继承；如果析构函数不是virtual，说明我并不希望这个类被继承；



### 条款8：别让异常逃离析构函数

C++中，虽然析构函数可以出现异常，但是析构函数出现异常会带来诸多问题。例如在析构函数close某一个fd，就有可能出现异常。

解决方法一：使用

XX：：~XX{

​	try{close（fd）；}

​	catch（...）{

​		abort（）；

​	}

}

异常规范来约束这样的行为，可以重新定义处理方法，如果catch为空就是忽略错误。

更推荐的解决方法二：**提供一个函数让客户主动关闭。**虽然让客户来决定了是否关闭这个fd会比较麻烦，但是这给了客户一个机会去处理错误。

**建议：**

1. 析构函数不要吐出异常；
2. 可预料的会产生错误的操作应该提供一个函数去检查是否出错，而不是将该操作留在析构函数内；



### 条款9：构造和析构函数中不要调用virtual函数

例如有一个类：

class A{

public：

​	A（）；

​	virtual void funcA（） const = 0；//纯虚函数，只能作为基类，不能产生对象

}

A：：A（）{

​	funcA（）；

}

class B：public A{

public：

​	virtual void funcA（） const；

}

此时我的愿望是**构造B的时候自动调用我在B内重定义的funcA**，但是真的会这样吗？

我们知道，构造派生类之前必须先构造基类，这个funcA是基类构造函数的其中一行，也就是说，**这个时候并没有派生类，但是你却要调用派生类版本的funcA？**这不太现实，因为**这个时候这个对象的类型是基类**，派生类还没创建呢，虚函数根据调用对象的类型来决定使用哪一个方法，那此时必然是使用基类版本的funcA。

同理，在析构函数那里也一样，会有一些情况下，虚函数不能保证你能调用你想调用的那个函数。

解决方法是：基类构造函数不要调用virtual的成员函数，首先将其改成non-virtual的；然后由派生类将其所需要的参数放进基类构造函数参数列表，然后基类再调用。

**建议：**

1. 构造和析构的时候不要调用virtual的函数，因为不能确定能调用到派生类的函数；



### 条款10：赋值运算符应该返回一个\*this的引用

例如：

int x，y，z；

x = y = z = 5；被解析为（x = （y = （z = 5））），所以可以连续赋值。只有赋值运算符返回左边的值的引用，才可能把5赋予y。



### 条款11：在operator=中处理“自我赋值”

对于一些潜在的自我赋值，例如：

class A{

private：

int \*a；

}

A \*p，\*q；如果有p = q，你怎么知道p和q是不是一个对象呢？

但是一般你会这样实现赋值运算符：

A& A：：operator=（const A& obj）{

​	delete  a；

​	a = new int（\*obj.a）；//对obj的a变量解除指针，返回obj.a的值

​	return \*this；

}

万一自我赋值，那这就相当于删除了自己，再把自己赋予自己，将会报错。

解决方法一：增加一个**证同测试**：

A& A：：operator=（const A& obj）{

​	if（this == &obj）

​		return \*this；

​	delete  a；

​	a = new int（\*obj.a）；

​	return \*this；

}

这样确实可以解决自我赋值的问题，但是new的操作不一定会成功的啊。因此有了第二个版本：

A& A：：operator=（const A& obj）{

​	A \*temp = a；//将自己的变量保存起来

​	a = new int（\*obj.a）；//如果没错才会删除原来的a

​	delete  temp；

​	return \*this；

}

但是这样就没有证同测试了，没关系你可以加上，但是这样的代码效率会非常低。因此出现了swap技术：

A& A：：operator=（const A& obj）{

​	A temp（obj）；//拷贝一份obj副本

​	swap（temp）；//this和副本交换

​	return \*this；

}

这样就实现了赋值，同时传入的对象又没有变化，因为只是复制一份给对方而已，利用这个复制，会生成超级简介方案：

A& A：：operator=（A obj）{

​	swap（obj）；

​	return \*this；

}

注意，**这里是传值，**我们知道，如果传值就会**复制一份副本**传入函数，跟上面的方法是一样的，本体不会受到任何影响；

**建议**：

1. 赋值运算符必须解决自我赋值、为了避免发生错误，最好还是使用swap的版本；
2. 确定任何函数操作多个对象，其中多个对象是同一个的时候，仍然正确；



### 条款12：赋值对象的时候不要忘记任何成分

我们在给**派生类**对象编写复制构造函数或者赋值运算符的时候，不要忘了，基类对象也要构造或者赋值的。例如：

class A{

private：

​	int a；

}

class B：public A{

private：

​	int b；

}

那么在B的复制构造函数内，必须在初始化列表内构造基类：

B::B（const B &obj）：A（obj），b（obj.b）{}

注意了，**可以直接将派生类对象传入基类的构造函数，因为基类指针或引用可以指向派生类对象**，此时只会访问基类包含的成员变量，然后构造基类；

对于B的赋值运算符，要调用基类的赋值运算符：

B& B::operator=（const B &obj）{

​	A::operator=（obj）；//**显示调用基类的赋值运算符**

​	b = obj.b；

​	return \*this；

}

**注意：**

①**复制构造函数不要调用赋值运算符**，因为你没有构造完成，哪来的对象让你复制呢？

②赋值运算符不要调用复制构造函数，复制构造函数是用来构造对象的，调用赋值运算符的时候肯定对象都已经存在了，再构造对象没有任何意义；

**建议**：

1. 确保基类的成员也完整地复制了；
2. 如果要实现代码重用，应该将代码放在第三个函数内，然后复制构造函数和赋值运算符都调用它，而不是混搭复制构造函数和赋值运算符；



### 条款13：以对象管理资源

C++最常见的需要管理的资源包括**内存、文件描述符fd、互斥锁、数据库连接、网络socket等。**

如果每次都是new然后delete，可能容易忘记delete；此外，在以后维护代码的时候可能在new和delete中间插入了一个return，那这块内存就泄漏了；因此，出现了“以对象管理资源”的观念，称为**RAII（resource acquisition is initialization）资源获取时即初始化**。也就是我们常说的C++11中的智能指针，**智能指针不是指针，而是一种行为类型指针的模板类，**智能指针的好处是指针过期的时候会析构指向的对象，保证内存不会泄露。

①auto_ptr，最初始的智能指针，为了防止多个指针指向同一个对象而发生析构多次的错误，**auto_ptr对对象有独占权，但是并不阻止对象所有权转移。**例如：

std::auto_ptr<int> p1（new int（5））；

std::auto_ptr<int> p2；

p2 = p1；//此时p2独占对象，p1指向null

针对这个缺点，出现了unique_ptr，这个指针也是独占的，但是它**不允许转移对象所有权**。因此，**严禁在STL里面使用auto_ptr**，因为其对象不可复制。但是可以调用reset（）方法或者move强制转移对象所有权。

此外，最常用的应该是shared_ptr，这是一个共享对象所有权的**计数型智能指针**，其内部有一个成员变量记录指向该对象的shared_ptr的数量，只有数量归零的时候才会调用对应的析构函数。但是存在一个**环状引用**的问题，这个问题出现在两个对象的内部含有一个shared_ptr指向对方，指向两个对象的shared_ptr已经不用了，但是两个对象内部都有一个shared_ptr指向对方，两个都不会析构，但是你又无法使用那两个shared_ptr，内存就泄露了。解决方法是**weak_ptr**，这个指针可以指向shared_ptr指向的对象，但是不参与计数（有一个计算weak_ptr的成员变量），因此即使有weak_ptr指向对象，依然会直接析构。

关于weak_ptr，shared_ptr可以直接赋予weak_ptr，但是反过来不行，必须调用weak_ptr.lock（）方法，例如：

weak_ptr<int> wp1（new int（5））；

shared_ptr<int> sp1（wp1.lock（））；

此外，注意了：**shared_ptr只会调用delete而不会调用delete[]，要使用delete[]可以使用unique_ptr。**如果你要用shared_ptr指向一个数组，建议使用vector；

**建议**：

1. 为了防止内存泄漏，建议使用RAII；
2. 最好使用shared_ptr，auto_ptr有些地方太不友好了；



### 条款14：在资源管理类中小心copy行为

