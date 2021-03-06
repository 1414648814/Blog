# 学习笔记

1. explicit，可以阻止它们被用来执行隐式类型转换，但它们仍可以被用来进行显示类型转换；

## 2. 尽量以const，enum，inline替换#define

或许改成这样：宁可编译器替换预处理器，总之，尽量少用预处理。

预处理是指以#号开头的代码行。预处理过程扫描源代码，对其进行初步的转换，产生新的源代码提供给编译器。检查包含预处理指令的语句和宏定义(define)，并对源代码进行相应的转换。预处理过程还会删除程序中的注释和多余的空白字符。可见预处理过程先于编译器对源代码进行处理。

**总而言之：**
- 对于单纯变量
        const 代替宏变量有助于编译器理解；
        
        enum 更像define，不消耗内存，无法取得地址，因为取得一个enum的地址是不合法的；

- 对于形似函数的宏
        inline 宏函数尽量用inline代替，因为宏并不会招致函数调用带来的额外的开销，而是一种简单的替换；

## 3. 尽可能使用const

- 如果关键字const出现在*左边，表示所指对象为常量，其中包括const char* p和char const *p，这两种写法是一样的;
- 如果关键字const出现在星号右边，表示指针为常量；

```cpp
std::vector<int> vec;
const std::vector<int>::iterator iter = vec.begin();
*iter = 10;
++iter; // Error!!!

std::vector<int>::const_iterator cIter = vec.begin();
*cIter = 10; // Error!!!
++cIter;

```

前者错的原因在于，作用相当于int *const，指针为常量，累加自然会错误；后者错的原因在于const int* ，表示所指对象为常量，重新赋值常量自然会报错；

两个成员函数如果只是常量性不同，是可以被重载的。成员函数是const就意味着bitwise constness和logical constness。const成员函数是不可以更改对象内任何not-static成员变量，但是可以使用mutable（在变量定义的时候加上）来摆脱这个约束。

const 和 non-const 成员函数的重复问题是无法通过mutalbe来解决的。但是可以通过强制类型转换来结局。

总而言之：

**1.将某些东西声明为const可以帮助编译器侦测出错误用法，const可以被施加到任何作用域内的对象，函数参数，函数返回类型，成员函数本体；**

2.编译器强制实施 bitwise constness，但应该要使用conceptual constness，这是两个流派，前者简直表里如一，使用了const就不应该加其他的东西，但是毕竟人无完人，可以使用类似 mutable, const_cast, static_cast等；

3.const 只能修饰输入参数，不能修饰输出参数；对于非内部数据类型（内部数据类型包括int...）的输入参数，应该将值传递改成 const 引用传递，用来提高效率，因为函数体内部将产生某类型的临时对象用于复制参数，而临时对象的构造、复制、析构过程都将消耗时间；

4.当const和非const成员函数有着实质等价的实现时，令non-const版本调用const版本避免代码重复；

参考文章如下：

[文章一](http://blog.csdn.net/zcf1002797280/article/details/7816977)

[文章二](http://www.cppblog.com/note-of-justin/archive/2009/12/15/103256.aspx)

## 4. 确定对象在使用前已经被初始化

永远在使用对象之前先进行初始化，确保每个构造函数都将对象的每一个成员初始化；

需要分清楚复制和初始化的区别，对象的成员变量的初始化动作发生在进入构造函数本体之前，所以应该将成员变量的初始化置于构造函数的初始化列表里（也就是在 : 后面的），但是如果放在构造函数中，那就是复制操作；

C++的成员初始化次序是固定的，base classes早于derived classes被初始化，而class的成员变量总是按照其声明次序(注意声明式)初始化，即使在初始化列表中出现的次序不同。

总而言之：

- 为内置对象进行手工初始化，因为C++不保证初始化它们；
- 构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。初始化列表列出的成员变量，其排列次序应该和它们在类中的声明次序相同；
- 为免除“跨编译单元之初始化次序”问题，请以local static对象替换non-local static对象


##5. 了解C++默默编写并调用哪些函数

**如果没有声明，那么编译器就会为类声明一个拷贝构造函数，一个拷贝赋值函数和一个析函数（都是编译器的版本）。如果没声明任何构造函数，则会声明一个默认构造函数，**而且这些都是 public 且 inline的；这些函数只有在被调用的时候，才会被编译器创建出来；

default构造函数和析构函数主要是给编译器一个地方用来放置“幕后的代码”，比如说调用 base classes 和 non-static 成员变量的成员函数和析构函数。

注意:

编译器产生的析构函数是个non-vritual，除非这个类的基类自身声明有virtual析构函数；

## 6. 若不想使用编译器自动生成的函数，就该明确拒绝

通常不希望class支持某一项特定的功能，只要不声明对应的函数即可，但是这个策略对拷贝构造函数和拷贝赋值函数并不起作用，编译器仍会自动声明；

因此，如果我们想让这些函数失效，可以将该成员函数声明为private并且故意不去实现它，可以设计一个专门阻止copying动作的base class类，再用一个子类去继承父类；但是这种方法会造成多重继承；

可以尝试着使用，创建一个宏，并将之放到每一个独一无二对象的private中，google推荐使用这种方法：

```cpp
#define DISALLOW_COPY_AND_ASSIGN(TypeName) \
TypeName(const TypeName&); \
void operator=(const TypeName&)

```

## 7. 为多态基类声明 virtual 析构函数

当基类的指针指向派生类的对象的时候，当我们使用完，对其调用delete的时候，其结果将是未有定义——基类成分通常会被销毁，而派生类的充分可能还留在堆里。这会造成资源泄漏、败坏之数据结构、在调试器上浪费许多时间。

解决办法是给基类加上一个virtual析构函数，任何类只要带有virtual函数都几乎确定应该也会有virtual析构函数；

总而言之：

- **带有多态性质的基类应该声明一个virtual析构函数，如果一个类带有任何virtual函数，他就应该拥有一个virtual析构函数**；
- 一个类的设计目的，如果不是作为基类或者为了多态性使用，就不应该使用virtual析构函数，否则需要额外的开销（指向虚函数表的指针vptr）；

## 8. 别让异常逃离析构函数

析构函数吐出异常就是危险，总会带来“过早结束程序”或者“发生不明确的行为”的危险；

如果某个操作可能在失败时抛出异常，而又存在某种需要必须处理该异常，那么这个异常必须来自析构函数以外的某个函数。
 
### 总结：
 
- 析构函数绝对不要吐出异常，如果一个被析构函数调用的函数可能会抛出异常，析构函数应该要捕捉异常，或则是吞下它们而不进行传播或者结束程序；
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么类应该提供一个普通函数（而非在析构函数中）执行该操作；

## 9. 绝不在构造和析构过程中调用virtual 函数

如果在父类的构造函数或者析构函数中使用虚构函数，首先，父类的构造函数会首先被调用，
**在derived class对象的base class构造期间，当前对象的类型是base class（父类）而不是derived class（子类）。不只virtual函数会被编译器解析到base class，如果要使用运行期类型信息(例如dynamic_cast和typeid)，也会把对象视为base class类型。**

因为无法使用virtual函数从base classes向下调用，将相应的函数改成non-virtual，在构造期间，可以将derived classes将必要的信息向上传递给base class构造函数，这样就保证了确定你的构造函数和析构函数都没有调用虚函数，而它们调用的所有函数也不应该调用虚函数。

### 总结
    
    在构造和析构期间不用调用virtual函数，因为这类调用从不下降至子类中（比起当前执行构造函数和析构函数那一层）
    
## 10. 令operator= 返回一个reference to *this

```cpp
class Widget {
public:
    ...
    Widget& operator=(const Widget* rhs) {
        ...
        return *this;
    }
};

```

因为赋值是采用右结合律的，所以赋值操作符必须要返回一个reference指向操作符的左侧实参。同时也适用于+=，-=，*=等；








[文章1](http://blog.csdn.net/john_cdy/article/details/45331957)
[文章2](http://www.cnblogs.com/fanzhidongyzby/archive/2012/11/18/2775603.html)
[文章3](http://blog.csdn.net/shenzi/article/details/5601038)
[文章4](http://dongxicheng.org/cpp/effective-cpp-part1/)
