(前三章看完)

从右向左阅读，判断对象类型(类型修饰符从右向左绑定)

对于数组，由内向外阅读，判断类型(c++ primer—103)

c++程序尽量使用vector和迭代器，避免使用内置指针和数组，尽量使用string类型

c++程序建议使用引用类型的形参代替指针

::全局作用域运算符

## c++11标准

### 范围for

```c++
/*
expression表示一个序列：花括号括起来的初始值列表、数组、vector和string等(拥有能返回迭代器的begin和end)
declaration定义一个变量，序列中的所有元素都得能转换成该类型，故从简可以设置成auto类型
如果对序列中的元素需要进行写操作(改动序列元素)，则declaration必须声明成引用类型
每次迭代，declaration都会重新定义，初始化为expression中的下一个值
*/
for(declaration : expression)
    statement

```

**对于写操作和引用的深入探究**：多维数组的应用

```c++
//传统for循环初始化
constexpr size_t row_cnt = 3, col_cnt = 4;
int ia[row_cnt][col_cnt];
for(size_t i = 0;i < row_cnt;i++) {
    for(size_t j = 0;j < col_cnt;j++) {
        ia[i][j] = i*col_cnt + j;
    }
}
```

```c++
//应用范围for进行初始化
constexpr size_t row_cnt = 3, col_cnt = 4;
size_t cnt = 0;
int ia[row_cnt][col_cnt];
for(auto &row: ia) //第一个for循环遍历ia所有元素，则row的类型是大小为4的整数数组的引用
    for(auto &col: row) { //第二个for遍历3个数组中的其中一个(按顺序)，则col类型是整数的引用
        col = cnt;
        cnt++;
    }
```

要使用范围for处理多维数组，如果最内层不需要写操作，则**最内层**的控制变量**可以不声明**为引用类型，外层循环的控制变量必须声明成整数类型

```c++
for((const) auto &row1: ia) {
    for((const) auto col1: row1) {
        cout << col1 << endl;
    }
}

//如果外层变量不是引用类型
for(auto row2: ia) 
    for(auto col2: row2)
/*
第一个for循环还是遍历ia所有元素，还是3个数组；由于没有声明成引用类型，初始化row的时候，会将这些元素转换成指向数组首元素的指针，则row就是int *类型，不能够遍历
*/
```



## extern关键字

### 变量的声明和定义

变量可以被声明很多次，但是定义只能一次

- 作用1：在变量类型前面加上extern关键字，且不初始化，则是声明

  ```c++
  extern int t1;
  ```

  

- 作用2：**分离编译**。只在一个文件中定义const，在其他多个文件中声明并引用

  ```c++
  //const变量不管是声明还是定义都添加extern关键字
  //在ta.h中声明，指明该变量并非本文件独有，定义会在别处出现
  extern const int buf_size;
  //在te.cpp中定义，该常量能被其他文件访问
  extern const int buf_size = 1024;
  ```

  

- 3

- 4

## 头文件

头文件通常包含只能**被一次定义**的实体，如类、const和constexpr变量等

### #include

**#include< >**

#include< > 引用的是**编译器的类库路径**里面的头文件。

假如你编译器定义的自带头文件引用在 C:\Keil\c51\INC\ 下面，则 #include<stdio.h> 引用的就是 C:\Keil\c51\INC\stdio.h 这个头文件，不管你的项目在什么目录里， C:\Keil\c51\INC\stdio.h 这个路径就定下来了，一般是引用自带的一些头文件，如： stdio.h、conio.h、string.h、stdlib.h 等等。

**#include" "**

#include" " 引用的是你**程序目录的相对路径**中的头文件。

假如你的项目目录是在 D:\Projects\tmp\ ，则 #include"my.h" 引用的就是 D:\Projects\tmp\my.h 这个头文件，一般是用来引用自己写的一些头文件。如果使用 #include" " ，它是会先在你项目的当前目录查找是否有对应头文件，如果没有，它还是会在对应的引用目录里面查找对应的头文件。例如，使用 #include "stdio.h" 如果在你项目目录里面，没有 stdio.h 这个头文件，它还是会定位到 C:\Keil\c51\INC\stdio.h 这个头文件的。

### 预处理器和文件保护符

预处理器是一些指令，指示编译器在实际编译之前所需完成的预处理

```c++
#include
#define //宏
//条件编译
#ifdef
#ifndef
#endif
```

**文件保护符：**

```c++
/*
aaa.h头文件内容如下：
class A{}

bbb.h头文件内容如下：
#include <aaa.h>
class B{ }
则bbb.h相当于是如下内容：
class A{}
class B{}

如果我还有一个ccc.h的头文件，定义如下：
#include <aaa.h>
#include <bbb.h>
那么ccc.h中的内容相当于：
class A{}
class A{}
class B{}
那么这样A类被定义了两次，我们要如何避免这种情况呢
*/

//aaa.h头文件定义：
#ifndef AAA_H //如果没有定义AAA_H这个宏，就执行下面的操作，如果有，就跳过下面一段
#define AAA_H
class{}
#endif
//bbb.h头文件定义：
#ifndef BBB_H
#define BBB_H
#include<aaa.h>
class{}
#endif
//ccc.h头文件定义：
#ifndef CCC_H
#define CCC_H
#include<aaa.h>
#include<bbb.h>
class{}
#endif
```

## 数组

```c++
int *p1[10]; //包含十个指针的数组
int (*p2)[10]; //指向含有十个整数的数组
int (&p3)[10]; //引用含有十个整数的数组

int &p4[10] = {...}; //错误，数组的元素应为对象，故不存在引用的数组
```

### 数组和指针

在大多数表达式中，使用数组类型的对象其实是一个指向该**数组首元素**的指针

**指针作为迭代器(用法和容器的迭代器相同)**

```c++
int arr[10];
int *p = arr;
for(int i=1;i<10;i++)
    arr[i] = i;
p++; //指向arr[1]
```

c++11中引入begin()和end()函数，功能类似容器的begin和end

```c++
//由于数组不是类类型，故begin和end也不是成员函数，使用时要将数组作为参数传入
#include <iterator>
int arr[10];
for(int i=1;i<10;i++)
    arr[i] = i;
int *init = begin(arr);
int *last = end(arr);
```

### 和C风格字符串的接口

C语言没有存储字符串的变量类型，故字符串都存储在char类型的数组中，末尾需要空字符'\0'

```c++
char c[4] = {'I', 'You', 'He', '\0'};
```

```c++
//CPP中字符串的赋值和初始化(允许使用字符串字面值初始化)
string cpp = "ssss";
string cpp("ssss");
```

**混用string对象和c风格字符串**

任何出现字符串字面值的地方都可以用以**空字符结束的字符数组**代替

```c++
//允许用空字符结束的字符数组来初始化string对象或为string对象赋值
char c[4] = {'I', 'Y', 'H', '\0'};
string cpp = c;

//在string对象的加法运算中允许以空字符结束的字符数组作为其中一个运算对象(不能两个都是)
char c[4] = {'I', 'Y', 'H', '\0'};
string cpp = c;
string cpp2;
cpp2 = cpp + c;

//在string对象的复合赋值运算中允许以空字符结束的字符数组作为右侧的运算对象
char c[4] = {'I', 'Y', 'H', '\0'};
string cpp;
cpp += c;

//不能用string对象直接初始化指向字符的指针，需要c_str()成员函数
string cpp = "ssss";
const char *p = cpp.c_str(); //const char *c_str()返回的是一个临时指针，不能对其进行操作
//更好的做法
char p[5]; //用char *p会报Segmentation Fault错误，即超出所给内存空间
string cpp = "ssss";
strcpy(p, cpp.c_str());
```

**使用数组初始化vector对象**

```c++
//指明拷贝区域的首位地址即可
int ia[4] = {1,2,3,4};
vector<int> vec(ia, ia+4);
vector<int> vec2(begin(ia), end(ia));
```

### 多维数组(数组的数组)

**二维数组和矩阵**：第一个维度为行，第二个维度为列

#### 多维数组的下标引用

表达式中，下标数量=数组的维数，则结果是最内层数组所存元素；下标数量<数组的维数，则给定结果是给定索引处的内层数组

#### 指针和多维数组

多维数组名转换的指针指向第一个内层数组

```c++
int ia[3][4];
int (*p)[4] = ia; //指向ia的第一个元素(即包含4个整数的数组) 注意：指向的元素是数组，不是内层数组的第一个元素
p = &ia[2]; //指向ia的尾元素
```

根据c++11的新标准，可以用auto或者decltype声明变量，避免像上述操作一样在数组前面加上一个指针类型

```c++
for(auto p = ia;p != ia+3;p++) {
    for(auto q = *p;q != *p+4;q++) //*p是一个含有4个整数的数组，q指向该数组的首元素
        cout << *q << endl;
}

for(auto p = ia;p != ia+3;p++) {
    for(auto q = p;q != p+4;q++) //错误，把p的指向错误理解成指向内层数组的第一个元素
        cout << *q << endl;
}
```



## 指针和引用

引用是一个对象的别名，本身并非对象；指针本身是一个对象

引用的类型必须和其引用的对象一致；指针的类型必须和其引用的对象一致；**但是有两个例外**

**例外1(详见下一小结const)：**

- 引用：对常量的引用(**简称**常量引用)，常量引用可以指向非常量对象；初始化常量引用时允许用**任意**表达式作为初始值，只要该表达式能够**转换**成引用类型

- 指针：指向常量的指针，可以指向非常量对象(**区别**常量指针)

  ```c++
  //指向常量的指针
  const int pi = 3;
  const int *p = &pi; //底层const(所指对象是一个const)
  //常量指针
  int pi = 3;
  int *const p = &pi; //顶层const(指针是一个const)
  ```

  

**例外2：**

## const

### const的引用

对常量的引用不能修改其绑定的对象，也不能让一个非常量引用绑定常量对象

```c++
const int ci = 1;
const int &r1 = ci; //引用和其绑定对象都是常量
r1 = 42; //错误
int &r2 = r1; //错误
```

**初始化常量引用**

初始化常量引用时允许用**任意**表达式作为初始值，只要该**表达式**能够**转换**成引用类型

非常量引用初始化必须是**左值**，且不能用常量引用初始化

常量引用可以绑定非常量对象

```c++
int size = 1;
const int &r1 = size;
const int &r2 = 2;
const int &r4 = r1*2;
int &r3 = r1; //错误
```

**当常量引用绑定了一个非const对象时**

```c++
int i = 1;
int &r1 = i;
const int &r2 = i;
r1 = 0; // 正确
r2 = 0; // 错误
```

常量引用**仅对引用可参与的操作做了限定**，对于引用的对象本身是不是常量未做限定

### const和指针

指针是否是常量以及所指对象是否是常量，两者相互独立(**但**是普通指针不能指向常量对象，同常量引用)

**指向常量的指针**(底层)

```c++
const int i = 1; 
const int *p = &i; 
```

指向常量的指针，可以指向非常量对象(**区别**常量指针)；普通指针不能指向常量对象

```c++
const int pi = 3;
int *p1 = &pi; // 错误。普通的指针不能指向常量

int pi2 = 3;
const int *p2 = &pi2; //正确。指向常量的指针可以指向非常累对象
```

同常量引用一样，指向常量的指针仅对**指针可参与的操作做了限定**，对于指向的对象本身是不是常量未做限定

**常量指针**(顶层)

```c++
int i = 1;
const int j = 2;
int *const p1 = &i; //i的值可以通过p1改变，但是p1所指的地址不能改变
const int *const p2 = &j; //指向常量的常量指针

int *p = p2; //错误，int *类型不能指向const int *类型(类似于非常量引用不能用常量引用实体化)
```

### 顶层和底层const

顶层const可以表示任何对象是常量，适用于任何数据类型；底层const则与指针和引用等复合类型的基本类型有关

```c++
const int ci = 1; // 顶层const
int *const p = &ci; //错误，必须用指向常量的指针
int i = ci; //拷贝时忽略顶层const
int *const p = &ci;  // 正确
```

**指针类型**既可以是**顶层(指针本身是常量**)，也可以是**底层(指针所指对象是常量)**

用于声明引用的const**都是**底层

执行对象拷贝操作时：顶层无影响，底层：拷入和拷出的对象都有底层const资格，或两者数据类型可以转换**（非常量可以转换为常量）**

可以用非常量初始化底层const对象，但是不能用底层const对象初始化非常量

```c++
int i = 1;
const int c = 1; //顶层
int *const p1 = &i; //顶层，常量指针
const int *p2 = &c; //底层，指向常量的指针
const int *const p3 = &c; //既是底层，又是顶层
const int &p4 = c; //底层

//拷贝
//顶层不受影响
i = c; // 拷贝操作
p2 = p3; // 两者指向的对象相同，p3顶层const部分不影响

//底层
p2 = p3; //两者都有底层const定义
p2 = &i; //int *能转换为const int *
int *p = p3; //错误，p没有包含顶层或底层的const定义
```

### 指针、常量、数组和类型别名(复合类型别名)

```
char *p; //指向char类型的指针

typedef char *pstring; //pstring是char *类型的别名，是指向char的指针
const pstring cstr = &p; //const char是指向char的常量指针，故cstr是指向char的常量指针
const pstring *ps; //ps是一个指向指向char的常量指针的指针

//错误理解
const char *cstr = &p; //并非指向常量字符的指针

//数组类型
using int_array = in[4];
typedef int int_array[4]; //两个声明等价，将类型"四个整数的数组"别名为int_array

for(int_array *p = ia;p != ia+3;p++) { //int_array *p等价int (*p)[4]
    for(auto q = *p;q != *p+4;q++) //*p是一个含有4个整数的数组，q指向该数组的首元素
        cout << *q << endl;
}
```

### ==初始化和const总结==

#### 默认初始化

定义变量时没赋初值，则变量的默认值由变量类型决定。

定义在函数体内部的内置类型变量不会被初始化。

类的对象如果没有显式初始化，则其值由类确定。绝大多数类都支持无须显式初始化定义对象。

#### 复合类型初始化

##### 引用类型

引用和其绑定对象类型需要一致**(有特例：特例1：const的引用；特例2：)**

引用必须初始化，且**只能**绑定在对象上，不能与**字面值**或某个表达式的计算结果绑定

##### 指针类型

指针和其指向的对象类型需要一致**(有特例：特例1：指向常量的指针；特例2：)**

赋值永远改变的是等号左侧的对象

##### ==const和初始化==

只能在const类型的对象上执行**不改变其内容**的操作：参与运算，转换成Bool，**初始化**

```c++
int i = 42;
const int ci = i;
int j = ci;
```

**const的引用(常量引用)** ==特例1==

常量引用初始化时，允许用**任意**表达式作为初始值，只要该**表达式**能够**转换**成引用类型：非常量的对象、字面值、表达式等

常量引用可以绑定非常量对象；非常量引用初始化必须是**左值**，且不能用常量引用初始化



```c++
int i = 42;
const int &r1 = i; // 允许将const int &绑定到一个普通int对象上
int &r2 = r1; //错误，非常量引用初始化必须是左值，且不能用常量引用初始化

const int &r2 = 42; // 正确
const int &r3 = r1*2; // 正确
```

**指向常量的指针** ==特例1==

指向常量的指针，可以指向非常量对象(**区别**常量指针)；普通指针不能指向常量对象

**想要存放常量对象的地址，只能使用指向常量的指针**

```c++
const int pi = 3;
int *p1 = &pi; // 错误。普通的指针不能指向常量

int pi2 = 3;
const int *p2 = &pi2; //正确。指向常量的指针可以指向非常累对象
```

**常量指针**

```c++
int i = 1;
const int j = 2;
int *const p1 = &i; //i的值可以通过p1改变，但是p1所指的地址不能改变
const int *const p2 = &j; //指向常量的常量指针

int *p = p2; //错误，int *类型不能指向const int *类型(类似于非常量引用不能用常量引用实体化)
```

指针是否是常量以及所指对象是否是常量，两者相互独立(**但**是普通指针不能指向常量对象，同常量引用)

#### 函数形参初始化

形参初始化机理和变量初始化一样

**引用传递**：引用形参即其对应实参的别名

若函数无须改变引用形参的值，最好将其声明为const引用

**传值(拷贝)**：指针形参也是一种传值。此时形参和实参是两个相互独立的对象

==**const形参和实参**==

实参初始化形参会忽略顶层const，当形参有顶层const时，给其传常量对象或非常量对象都可

```c++
void fcn(const int i) {/**/}

int i1 = 1;
const int i2 = 2;

fcn(i1);
fcn(i2);
```

**==指针、引用形参与const==**

**通过加入const关键字，能在一定程度上扩大能接受实参的类型**

**可以用非常量初始化底层const对象，但是不能用底层const对象初始化非常量**：

常量引用可以用非常量引用初始化，非常量引用必须用同类型的对象初始化(即非常量引用不能用常量引用初始化)

指向常量的指针可以用普通指针初始化，普通的指针不能用指向常量的指针初始化

```c++
int i = 0;
const int ci = i;

void reset(int *p) {/*不改变实参*/}
reset(&i); // 正确
reset(&ci); // 错误。需要指向常量的指针

void reset(const int *p) {/*不改变实参*/}
reset(&i); // 正确。指向常量的指针可以用普通指针初始化
reset(&ci); // 正确

void reset(int &p) {/*不改变实参*/}
reset(i); // 正确
reset(ci); // 错误。非常量引用不能初始化常量引用
reset(42); // 错误。不能用字面值初始化常量引用

void reset(const int &p) {/*不改变实参*/}
reset(i); // 正确。常量引用可以用非常量引用初始化
reset(ci); // 正确
reset(42); //正确。常量引用可以用字面值初始化
```

==如果把形参定义成引用，不改变引用的对象，尽量声明成常量引用==

## 标准库函数

```c++
srand(); //随机数生成种子
rand() % n; //生成[0,n)之间的随机数
```

```c++
fstream(const char* filename,int mode,int access):
open(const char* filename,int mode,int access) // int mode以|为分割设置多模式，缺省则以文本模式打开
// 子类
ofstream(const char* filename,int mode) // 写文件
ifstream(const char* filename,int mode) // 读文件
```

## 函数

### 返回值

函数的返回类型不能是函数或数组，但可以是指向函数或数组的指针

### 传参方式

同初始化要求一样

```c++
//传值(拷贝)
//引用/传址
  ① by reference: & //定义时要以&定义类型，传递对象的地址，在函数中对形参的操作等同于对对象本身的操作。调用时不用加&。
  ② by pointer: * //同by reference，调用时加&。
dereference //提领操作(取值)
```

#### const形参和实参

类似于顶层const和底层const

### 函数的声明、定义和分离式编译

函数声明(也称为**函数原型**)通常放在头文件中

函数定义通常在程序代码文件

inline函数定义通常放在头文件

#### 编译和链接多个源文件



**函数指针：**

相对函数重载来说，可以进一步简化程序

1. 获取函数地址—提供函数名称即可

2. ```c++
   int (*a)(); //定义函数指针
   // 调用，推荐第一种
   (*a)();
   a();
   ```

   

3. ```c++
   int (*a[])(); //定义函数指针数组，内存放函数指针
   //调用
   a[]();
   (*a[])();
   
   int *a[]; //存放指针
   ```

   

4. ```c++
   int (*(*a)[])(); //指向包含多个函数指针数组的指针
   //赋值
   a = 地址;
   //调用
   (*a)[]();
   ((*a)[])();
   ```

   

5. 函数的返回类型不能是**数组类型**或**函数类型**，但是可以是指向函数或数组的指针

## 泛型编程

C++是一门强类型语言，所以无法做到像动态语言（python javascript）那样子，编写一段通用的逻辑，可以把任意类型的变量传进去处理。泛型编程弥补了这个缺点，通过把通用逻辑设计为模板，摆脱了**类型的限制**，提供了**继承机制以外**的另一种抽象机制，极大地提升了代码的可重用性。

注意：**模板定义本身不参与编译**，而是编译器根据模板的用户使用模板时提供的类型参数生成代码，再进行编译，这一过程被称为模板实例化。用户提供不同的类型参数，就会实例化出不同的代码。

==与元素类型无关，与容器无关==

消除元素类型依赖：

- 函数重载
- 函数模板

消除容器类型依赖：泛型指针

消除操作依赖(如运算符操作)：函数指针—>function object

**将一个函数改造成一个泛型算法**

### 泛型指针(Iterator)（迭代器）

使用与指针相同的语法以代替指针直接操作

```c++
vector<string>::iterator iter;
iter = svec.begin(); //泛型指针的定义，指向string类型的vector，初值指向svec的起始位置
iter->size(); //调用string提供的底层操作

vector<string>::const_iterator it; //不同于常量指针，const_iterator不能修改其所指的对象的值；const_iterator 对象可以用于const vector 或非 const vector,它自身的值可以改(可以指向其他元素),但不能改写其指向的元素值；传递过来一个const类型的容器，那么只能用const_iterator来遍历
```

### 顺序性容器

```c++
// vector
/*
随机访问效率高
非末端插入删除操作效率低
适合数列
*/
#include <vector>
int ia[3] = {3,1,2};
vector<int> vec(ia, ia+8); //通过一对iterator产生容器，iterator指示作为初值的取值范围

vec.push_back(); //末尾插入
```

```c++
// list
/*
随机访问效率低
任意位置插入删除操作效率高
适合排序
*/
#include <list>
list<string> slist(10,"sss"); //产生指定大小容器并赋予初值
```

```c++
// deque
/*
前端和末端插入删除效率高
*/
#include <deque>
deque<double> deq1;
// 填充deq1...
deque<double> deq2(deq1); //根据原容器产生新容器
```



### 泛型算法和弹性

当函数的弹性显得不足时，如运算符的多用，有两种解决办法

问题例：给定一个整数vector，返回一个新vector，包含原vector中小于10的数值

始源解法：

```c++
vector<int> less_than_10(const vector<int> &vec)
{
    vector<int> nvec;
    for(int i = 0;i < vec.size();i++)
    {
        if(vec[i] < 10)
            nevc.push_pack(vec[i]);
    }
    return nevc;
}
```

不使用template解法：

```c++
//思路：先排序，再用find_if()找出第一个大于指定值的元素位置
vector<int> sub_vec(const vector<int> &vec, int val)
{
    vector<int> local_vec(vec);
    sort(local_vec.begin(), local_vec.end()); //先对vector的副本进行排序
    
    vector<int>::iterator iter = find_if(local_vec.begin(),
                                         local_vec.end(),
                                         bind2nd(greater<int>(), val));
    local_vec.erase(iter, local_vec.end());
    return local_vec;
}
```

两点需求：

- 用户想比较除了10以外的数值
- 用户想加入其他比较运算

#### 函数指针

```c++
//通用比较函数定义
vector<int> filter(const vector<int> &vec, 
                  	int filter_value,
                  	bool (*pred)(int, int)) //filter_value表示比较值，pred指向自定的比较函数
{
    vector<int> nvec;
    for(int i = 0;i < vec.size();i++)
    {
        if(pred(vec[i], filter_value))
            nvec.push_back(vec[i]);
    }
    return nevc;
}

//用户自定义比较函数
bool less_than(int v1, int v2)
{
    return v1 < v2 ? true : false;
}
bool greater_than(int v1, int v2)
{
    return v1 > v2 ? true : false;
}

int main()
{
    vector<int> extern_vec;
    int val;
    //填充...
    
    vector<int> local_vec = filter(extern_vec, val, less_than);
    return 0;
}
```

此处已经实现一定的弹性，用户可以指定比较方式和比较值，但是仍然无法摆脱元素类型以及容器类型的依赖性

#### Function Object

```c++
#include <functional>
```

**adapter**

- binder adapter: 将function object的参数绑定至某特定值，使function object从二元转为一元

**消除元素类型依赖—function template**

**消除容器类型依赖—iterator[first,last)（泛型指针）**

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

using namespace std;

template <typename InputIterator, typename OutputIterator, typename ElemType, typename Comp>
OutputIterator filter(InputIterator first, InputIterator last, OutputIterator at, const ElemType &val, Comp pred)
{
    while((first = find_if(first, last, bind2nd(pred,val))) != last)
    {
        cout << "found value: " << *first << endl;
        *at = *first;
        at++;
        first++; //*at++ = *first++也可
    }
    return at;
}

int main()
{
    const int elem_size = 8;

    int ia[elem_size] = {12,8,43,0,6,21,3,7};
    vector<int> ivec(ia, ia+elem_size);

    int ia2[elem_size];
    vector<int> ivec2(elem_size);

    cout << "less than 8\n";
    filter(ia, ia+elem_size, ia2, elem_size, less<int>());

    cout << "greater than 8\n";
    filter(ivec.begin(), ivec.end(), ivec2.begin(), elem_size, greater<int>());

    return 0;
}
```

设定目的容器大小的做法有时候会造成浪费，利用**insert iterator adapter**，可以实现当元素插入时再扩展容器

以下三种adapter分别以push_back(), insert(), push_front()代替assignment，**即将assignment替换为插入操作**

- back_inserter() //适合vector
- inserter()
- front_inserter() //适合list和deque

```c++
int main()
{
    const int elem_size = 8;

    int ia[elem_size] = {12,8,43,0,6,21,3,7};
    vector<int> ivec(ia, ia+elem_size);

    int ia2[elem_size];
    vector<int> ivec2;

    cout << "less than 8\n";
    filter(ia, ia+elem_size, ia2, elem_size, less<int>());

    cout << "greater than 8\n";
    filter(ivec.begin(), ivec.end(), back_inserter(ivec2), elem_size, greater<int>()); //用back_inserter()代替assign

    return 0;
}
```

### 常用泛型算法

### find()

```c++
// 对于find()传入的两个iterator，类型要统一，比如要么都是const，要么都是non_const
find(fisrt, last, val);
```



## 类

### this

问题：成员函数如何获得对象的数据成员

```c++
struct Salse_data {
    // 成员函数，属于接口部分
    string isbn() {return bookNo};
    
    // 数据成员
    string bookNo;
};
```

在进行如下操作的时候，isbn()如何获得total对象的bookNo数据成员呢？

```c++
Salse_data total; // total对象
total.isbn();

/*
① 成员函数通过一个名为this的隐式形参来访问调用它的对象，该对象的地址被用来初始化this
② 任何对类成员的直接访问都被看作this的隐式引用，如同this->bookNo
*/
//①
Sales_data::isbn(&total);
//②
string isbn() {return this->bookNo;}
```

**this是一个==常量指针==，不允许改变this中保存的地址**

### 常成员函数(const成员函数)

```c++
//格式：在参数列表之后添加const关键字
string isbn() const {return this->bookNo;}
```

**作用：修改隐式this指针的类型**

默认this类型：指向非常量类的常量指针，即：

```c++
Sales_data *const
```

其仍然要遵循初始化规则，即**不能**把this绑定到**常量对象**上(因为不是指向常量的指针)

**导致了一个问题：**不能在一个常量对象上调用普通的成员函数

```c++
const Sales_data total;
total.isbn(); // 错误

/*
解决方法：把this声明成const Sales_data *const，指向常量的常量指针
由于isbn()不会改变this所指的对象，所以这样的声明有助于提高函数的灵活性
*/
/*
把const关键字放在成员函数的参数列表之后，则this是一个指向常量的指针，该成员函数被称作常量成员函数，不被允许修改对象的数据成员
*/
string isbn() const {return this->bookNo;}
```

常量对象，以及常量对象的引用或指针都只能**调用**常量成员函数

常量成员函数，可以被const和non_const类对象**调用**；而非常量成员函数，只能**被**non_const类对象**调用**