## cpp思考和总结

### 基础知识

### 一些有意思的条款

1.尽量用`const/enum/inline`替换掉`#define`

宁肯以编译器替换掉预处理器，因为#define在预处理时会完成替换过程，所有的宏对编译器都是不可见的。

因此，

* 对于单纯常量，最好用`const`对象或者`enums`替换掉`#define`
* 对于形似函数的宏，最好改用`inline`替换掉`#define`

2.尽可能使用`const`

将某些东西声明为`const`可以帮助编译器检测出错误的用法，`const`可以被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数的本体。

编译器强制实施于数据意义上的const(`bitwise constness`)，但是编写代码的时候应该使用概念上的const（`conceptual constness`）

3.确定对象被使用前已经先被初始化了

构造函数的时候最好使用成员初值序列，而不要在构造函数本体内使用赋值操作。初值列出的成员变量，其**<u>排列次序</u>**应该和它们在`class`中的声明次序相同。

```cpp
class A{
public:
    //初始化
    A():x(123), y(0) {
        //赋值
        x = 123, y = 0;
    }
    A():y(123), x(y - 1) {
        //注意尽管如此写，初始化的时候还是会先去初始化x。最后结果是，x废了，y没问题。
    }
private:
    int x;
    int y;
}；
```

### 语法糖

> 指的是计算机语言中的某种添加的语法，语法对语言的功能没有影响但是更加方便程序员使用。

​                         <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240502204107343.png" alt="image-20240502204107343" style="zoom:67%;" />

##### 1.引用

* 基本变量应用

  就是对变量（某一块内存）取了一个别名，定义的同时必须被初始化，同时不能改变为其他的引用，有唯一性。

  事实上，引用的底层实现就是使用了指针

  ```cpp
  int test() {
  	int a = 333;
      int &b = a;
      return b;
  }
  int test2() {
      int a = 333;
      int *p = &a;
      return *p;
  }
  ```

  `g++ test.cpp -S`

  ​                   <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240502204815775.png" alt="image-20240502204815775" style="zoom: 100%;" />

  `rbp`：基址寄存器 `rsp`：栈顶寄存器(栈针) `rax`临时寄存器 `leaq`操作地址 `（%rax)`表示取地址上的值

  首先`push`基址寄存器，保护现场，由于不能同时读写内存，要么从内存中读取，要么往内存中写，因此使用了`rax`

  从汇编可以看到，二者的底层是一样的，引用是指针的一颗语法糖。

* 数组引用

  ```cpp
  int a[10];
  int (&b)[10] = a;
  cout << a << endl;
  cout << b << endl;
  ```

  ​                                         <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240502213447274.png" alt="image-20240502213447274" style="zoom:33%;" />

* 函数引用

  ```cpp
  in test(int &b) {
      b = 444;
      return b;
  }
  int &test2(int &b) {
      b = 444;
      return b;
  }
  int main() {
      int a = 333;
      test(a);
      //此时是一个左值
      test2(a) = 555;
      cout << a << endl;
  }
  ```

  结果为`444`

* const引用(const实质上是只读，编译器喜欢让我们以为它是常的)

  ```cpp
  const int &b = 10;
  cout << b << endl;
  //之前是const int *
  *(int *)&b = 100;
  cout << b << ednl;
  ```

##### 2.const

简单回顾一下

**1. 指针常量**：不能修改指针所指向的**地址**。在定义的同时必须初始化**。**

```cpp
char * const a = &p; //指针常量。
*a = 'a';  // 操作成功
a = &b;  // 操作错误，指针指向的地址不能改变
```

**2. 常量指针**：不能修改指针所指向**地址的内容。**但可以改变指针所指向的地址。

```cpp
const char * a; //常量指针。
char const * a; //同上
*a = 'a';  //操作错误，指针所指的值不能改变
a = &b;    //操作成功
```

### 作用域

```cpp
#define BEGINS(x) namespace x{
#define ENDS() }
using namespace std;

BEGINS(kkb)
    class Ostream {
        public:
        Ostream &operator<<(int x) {
                printf("%d", x);
                return *this;
            }
        Ostream &operator<<(const char *x) {
                printf("%s", x);
                return *this;
        }

        };
    Ostream cout;
ENDS()
    
int main() {
    kkb:cout << "hello world" << std::endl;
    kkb:cout << 3 << std::endl;
    return 0;
}
```

### RAII

RAII 让资源和资源对应的对象的生命周期保持一致，具体来说： - 对象的初始化会导致资源的初始化 - 对象的释放会导致资源的释放 对象创建成功一定意味着资源获取成功；而对象释放成功则资源一定得到释放。

//todo

### 类和对象

一言敝之就是，类是对**<u>对象</u>**的抽象，对象是类的实例个体。类=数据+操作

* public 公共访问权限，类内和类外均可以访问
* private 私有访问权限，只有类内可以访问
* protected 后面将提到 主要是针对继承和派生

类的本质是什么呢？

#### 封装

##### 1.构造和析构

`构造`对我们的对象进行**初始化**的工作。`构造`和`析构`不需要我们手动去调用，程序会帮我们自动去调用。先构造的东西后析构，先定义的东西先析构。构造函数结束，完成了逻辑上的构造。但是，对象的内存，在定义的时候就有了这块内存，只是此时它是一块荒地。

为什么会有这个顺序呢？其实也很好理解

```cpp
A b;
A a(b);
//所以所肯定得后构造的先析构
```

* 默认构造

  啥都不干。计算机本身会帮我们生成默认的无参构造，默认的拷贝构造，默认的析构函数。如果我们已经定义了任何一个构造函数，计算机将不会再生成默认的无参构造，如果我们自己定义了拷贝构造，计算机将不会生成默认的拷贝构造。

  ```cpp
  class A {
      public:
      	//使用默认的系统构造
          A() = default;
          //不使用系统默认构造的拷贝构造
          //A(const A &a) = delete;
      private:
          //将不能再使用拷贝构造
          A(const A &a) = delete;
  };
  ```

  

* 转换构造（有参构造）

  ```cpp
  class A {
    public:
      int x;
      A(int x) {
          //初始化其实是失败的，这里的x始终是局部变量
          x = x;
          //这样是ok的
          this->x = x;
          cout << "transform constructor";
      }
      A（）{cout << "constructor" << endl;}
      ~A() {cout << "destructor" << endl;}
  };
  ```

  继续

  ```cpp
  A a;
  a = 333;
  //此时会发生什么？
  ```

  首先将333转换成一个`A`类的对象，但是它是临时的，又叫做匿名对象，之后再赋值给`a`，这里会有一个赋值运算符的重载

  ```cpp
  A &operator=(const A& a) {
      cout << "operator =" << endl;
      return *this;
  }
  ```

* 拷贝构造

  ```cpp
  A(A a) {
      cout << "copy constructor" << endl;
  }
  //将失败
  //实参要对形参先进行一次构造，就会去找拷贝构造函数，将一直递归
  A(A& a) {
      cout << "copy constructor" << endl;
  }  
  ```

  这里稍微注意一下，默认的拷贝构造是浅拷贝，即会使用拷贝对象的内存，如果有需要的话，建议自己实现一个拷贝构造，进行内存分配，完成深拷贝。

最后注意一下，构造函数当中完成的是初始化的工作，因此使用初始化列表的方式进行初始化，而不应该去赋值。

```cpp
A(x_):x(x_) {}
```

此外需要注意，类内的拷贝构造必须是const的。原因在于：

* 析构

  使用`new`申请类的对象的空间的时候，会自动帮助调用构造函数，`delete`时会帮助调用析构函数。

##### 2.类属性和方法

就是属于整个类，是整体对象的共有属性。

在一个类的实例中，只会存放非静态的成员变量。 如果该类中存在[虚函数](https://so.csdn.net/so/search?q=虚函数&spm=1001.2101.3001.7020)的话，再多加一个指向虚函数列表指针—vptr。

静态成员变量在类声明中进行声明，但在**类外进行定义**和[初始化](https://so.csdn.net/so/search?q=初始化&spm=1001.2101.3001.7020)。静态成员变量只有一个副本，它被所有属于类的对象所共享。静态成员函数在类声明中进行声明，并在**类外进行定义**。与普通成员函数不同，静态成员函数没有隐含的 this 指针，因为它们不依赖于特定的对象。

##### 3.const方法

表示这个方法是`readonly`的，只读不改。

##### 4.返回值优化

```cpp
class A {
    int x;
    public:
        A() {
            cout << this << " default constructor" << endl;
        }
        A(int x): x(x){
            cout << this << " param constructor" << endl;
        }
        A(const A &a):x(a.x) {
            cout << this << " copy constructor" << endl;
        }
        ~A() {
            cout << this << " destructor" << endl;
        }

};
//注意这里不能返回tmp的引用，当函数调用结束，tmp所在的内存会失去定义
A func() {
    A temp(100);
    cout << "tmp: " << &temp << endl;
    return temp;
}

int main() {
    A c = func();
    cout << "c:" << &c << endl;
    return 0;
}
```

 编译打印   

​                              ![image-20240514105541843](https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240514105541843.png)           

将使用匿名对象的中间过程给取消掉，直接调用了有参构造。可以添加编译命令`-fno-elide-consructors`禁止返回值优化

​                                <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240514111612012.png" alt="image-20240514111612012" style="zoom:80%;" />

> 一共有三次构造和析构的过程。过程：开辟c对象数据区->调用函数func开辟对象->temp数据区调用temp对象构造函数->使用temp调用临时匿名变量拷贝构造函数->销毁temp对象(局部变量消失)->使用临时匿名变量->调用c的拷贝构造函数->销毁临时匿名变量->销毁c对象。

##### 5.重载

* 函数重载

  没啥好说的，就是函数名可以相同，参数类型和返回值类型可以不同

* 运算符重载

  ```cpp
  class Point {
      int x, y;
  public:
      Point();
      Point(int x, int y);
      friend Point operator+(const Point &a, const Point &b);
      friend ostream &operator<<(ostream &out, const Point &a);
  
      Point operator+(Point &b) {
          cout << "inner" << endl;
          return Point(this->x + b.x, this->y + b.y);
      }
  
      Point &operator+=(int x) {
          this->x += x;
          this->y += x;
          return *this;
      }
      int &operator[](string x) {
          if (x == "x")
              return this->x;
          if (x == "y")
              return this->y;
      }
      void func(){
          cout << "hahaha" << endl;
      }
  };
  
  //Point::Point ():x(0), y(0){}
  //委托构造
  Point::Point ():Point(0, 0) {}
  Point::Point (int m, int n):x(m), y(n) {}
  
  Point operator+(const Point &a, const Point &b) {
      cout << "outter" << endl;
      return Point(a.x + b.x, a.y + b.y);
  }
  
  ostream &operator<<(ostream &out, const Point &a) {
      out << "(" << a.x << "," << a.y <<")" << endl;
      return out;
  
  }
  ```

##### 6.friend

看下面这段示例代码即可

```cpp
class People {
    int money;
public:
    People():money(1000000) {}
    void cost() {
        money -= 1000;
    }
    void show() {
        cout << money << endl;
    }
    friend void useOthersMoney(People &a);
};
useOthersMoney(People &a){
    a.money -= 1000;
}
int main() {
    People xiaok;
    xiaok.show();
    xiaok.cost();
    xiaok.show();
    useOthersMoney(xiaok);
    xiaok.show(); 
   return 0;
}

```

##### 7.函数指针

函数指针访问类中的成员函数

```cpp
class A {
public:
    void output(){
        cout << "hello" << endl;
    }
};
int main() {

    A a;
    A *k = &a;
    void (A::*p)() = &A::output;
    (k->*p)();
    (a.*p)();

    return 0;
}
```

普通函数的话可以不用取函数名的地址，调用的时候也不用取*。类成员函数需要指定取地址。

#### 多态

我就是我，颜色不一样的烟火。即使，当子类和父类有相同的方法的时候，应该能够区分。父类指针指向子类，会发生多态。指向自己还是多态吗？关键就是看`call`的是谁，动态绑定才是多态。如果父类的引用指向的是自己的对象，那么就不会发生多态。注意，指针无论指向谁都是会发生多态的。**<u>引用不能重新绑定，所以优化编译时可以编译期判定。</u>**

```cpp
class Animal {
public:
   void run() {
        cout << "I don't know how to run" << endl;
    }
   void say() {
        cout << "Animal say" << endl;
    }
};
void run() {
    cout << "I run with 4 legs" << endl;
}
class Cat:public Animal{
public:
    void run(){
        cout << "I run with 4 legs" << endl;
    }
    void say() {
        cout << "Cat say" << endl;
    }
};
```

```cpp
	Cat c;
    c.run();
	//用一个Animal指针指向Cat指针（父类指针调用子类的方法）
    Animal *p = &c;
    //仍然会调用Animal的run
    p->run();
    p->say();
    Animal &r = c;
    //仍然会调用Animal的run
    r.run();
    r.say();
```

那么，调用`run`函数是谁确定的呢？调用`run`方法的时候，将会传递一根对象的`this`指针。

​                             <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240515211308232.png" alt="image-20240515211308232" style="zoom:67%;" />

因此需要使用`virutal`和`override`

```cpp
class Animal {
public:
   virtual void run() {
        cout << "I don't know how to run" << endl;
    }
};
class Cat:public Animal{
public:
    //这里override很大的一个作用是告诉这里在对父类的一个虚函数进行重写
    //final关键字表示对该虚函数的重写到此为止，此后的继承不再允许被重写
    void run() override final{
        cout << "I run with 4 legs" << endl;
    }
};
```

此时父类的指针再去调用子类的方法，都将正常。

​                                 <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240515211229683.png" alt="image-20240515211229683" style="zoom:67%;" />

> 普通函数在编译之后就确定了，多态是发生在程序运行的过程之中。

<img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240515213139285.png" alt="image-20240515213139285" style="zoom: 50%;" />

可以看到`p->say()`是**<u>静态绑定</u>**的，直接调用了`Animal::say()`方法,这是代码编译后就确定的。但是`p->run()`而是调用了`rax`，这里面存的是一个函数入口地址，执行的时候才知道，称为<u>**动态绑定**</u>。这就是虚函数和其他普通函数最大的区别。

##### 虚函数

虚函数表里会存储一系列的指针数组，占据8个字节。在重写的时候，就会拿重写的函数的入口地址替代掉父类的函数入口地址。

先来看一下类在内存中的大小。

```cpp
class A {
public:
	void run() {
		cout << "hello A" << endl;
	}
};
sizeof(A)是多少？
//应该是1，即至少有一个字节。用来占位
class B {
public:
	virtual void run() {
		cout << "hello A" << endl;
	}
};
sizeof(B)是多少？
//应该是8，因为虚函数的声明，必须有一个8个字节的虚函数表
//如果是多个虚函数声明呢？其大小还是8
//注意，这里的讨论是没有加上成员变量的
//此时相当于一根指针指向好多的地址
```

底层的实现？**<u>指针数组</u>**！！！也就是说这8个字节就是指针数组的首地址。子类继承之后，也都会有这样一个指针，指向一个指针数组。当我们进行重写的时候，就会用自己定义的重写的函数的入口地址会去覆盖。

这里注意三个点：

* 在C++中，类的成员函数不会随着类的实例化而在内存中占用额外的空间。也就是说，无论创建多少个类的对象，成员函数只有一份在内存中的实例。这是因为类的所有对象都共享同一份成员函数的代码。
* 虚函数的实现代码，就像其他函数一样，存储在内存的代码段中。代码段是程序的一部分，保存了程序的所有只读代码。每个虚函数都在这里有一份实现。
* 虚函数表（Virtual Table 或 vtable）一般是存储在程序的只读数据段中的，不在堆和栈上。它是由编译器在编译时期创建的，并且它只有一份实例，对于同一个类的所有对象来说，他们的虚函数表是共享的。
* "只读"是指虚函数表的内容在程序运行期间不会改变，即虚函数的实现和它们在虚函数表中的顺序是固定的。这是由编译器在编译阶段确定的，所以说虚函数表是“只读”的。而"动态绑定"则是针对虚函数的调用方式来说的。
* 当我们通过一个基类指针调用一个虚函数时，这个函数具体调用哪个实现（即基类的实现还是某个派生类的实现）是在程序运行时才能确定的，这就是所谓的“动态绑定”。在执行这种虚函数调用时，会通过对象中的虚函数表指针（vptr）找到对应的虚函数表，然后根据虚函数在虚函数表中的偏移量，找到并调用正确的函数实现。这个过程是在运行时进行的，所以说是“动态”的。

```cpp
class A {
public:
    int x;
    //写成虚函数之后，
    virtual void run(int x) {
        this->x = x;
        cout << "A run x = " << this->x << endl;
    }
    virtual void say() {
        cout << "A say" << endl;
    }
    virtual void sleep() {
        cout << "A sleep" << endl;
    }
};
class B:public A {
public:
    void run(int x) override {
        this->x = x;
        cout << "B run x = " << this->x << endl;
    }
    void say() override {
        cout << "B say" << endl;
    }
};
typedef void(*funcp) ();
int main() {
    A a;
	B b, c;
}
```

来个丧心病狂的玩法

我想找到虚函数表的入口地址。

```cpp
cout << *(void**)&b << endl;
cout << *(void**)&c << endl;
((funcp**)&b)[0]();
((funcp**)&b)[1]();
//指针数组里存放的是funcp的指针
//那么指针数组的首地址是funcp*
//funcp*会被放入到b或者c里
//那么&b或者&c就等价于funcp**
(*(void (***)(B *,int))&b)[0] (&b,100);
//一定是调用的B的run，所以改的是a的x
(*(void (***)(void *,int))&b)[0] (&a,99);
```

编译完

​    <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240516190840276.png" alt="image-20240516190840276" style="zoom: 50%;" />

B继承自A，刚开始应该是一样的表，但是由于重写了`run`和`say`函数，它会被改写成自己的函数入口。

  <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240516200857613.png" alt="image-20240516200857613" style="zoom: 50%;" />

过程如下：如右侧的汇编所示，16是因为需要偏移两个函数指针。

当然，虚函数也可以写为纯虚函数，这样的类称为纯虚类。但是纯虚类是不能定义对象的。继承的子类如果没有把虚函数重写，任然是抽象类，而不是具象类。纯虚类的虚表里面是一个宏占有纯虚函数。

#### 继承

咱两谁是谁的爹，叫声爹不吃亏。这是抽象到具象得一个过程。

**子类访问父类权限**

​                  <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240514194603649.png" alt="image-20240514194603649" style="zoom:67%;" />

protected受保护的，儿子们是可以去访问的。儿子会把父类的都拷贝一份，但是有不代表都能访问，要根据权限才能确定。儿子访问遗产只和父类的权限有关系，外人想访问这块遗产需要父类权限和子类权限综合决定。

**外部访问权限**

​                  <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240514200210261.png" alt="image-20240514200210261" style="zoom: 67%;" />

```cpp
class Animal {
protected:
    string name;
public:
    Animal(string n = "animal"):name(n){
        cout << "Animal constructor" << endl;
    }
    Animal(const Animal &a) {
        cout << "Animal copy constructor" << endl;
    }
    void animalTell() {
        cout << "Animal:name is" << name << endl;
    }
    ~Animal() {
        cout << "Animal destructor" << endl;
    }
};

class Cat:public Animal {
    string color;
public:
    Cat():Animal("big cat"), color("yellow") {
        cout << "Cat cosntructor for Animal" << endl;
    }
};
```

子类的拷贝构造建议调用父类的拷贝构造

```cpp
   Cat(const Cat &c) {
       cout << "copy constructor" << endl;
   }
```

这样写的子类的拷贝构造不正确，如果父类还有指针的话极容易导致浅拷贝，因此要把父类的拷贝构造也先实现。赋值运算符的重载类似。

```cpp
    Cat(const Cat &c):Animal(c), color("yellow") {
        cout << "Cat copy constructor" << endl;
    }
```

##### 多继承

```cpp
class Animal {
public:
    int age;
    Animal():age(1) {}
};

class Horse:public Animal {
public:
    Horse() {
        cout << "horse constructor" << endl;
    }
    void setAge(int a) {
        cout << "&age = " << &age << endl;
        age = a;
    }
    ~Horse() {
        cout << "horse destructor" << endl;
    }
};

class Donkey:public Animal{
public:
    Donkey() {
        cout << "donkey constructor" << endl;
    }    
    int getAge() {
        cout << "&age = " << &age << endl;
        return age;
    }
    ~Donkey() {
        cout << "donkey destructor" << endl;
    }
};
class Mule:public Horse, public Donkey {
public:
    Mule() {
        cout << "mule constructor" << endl;
    }

    ~Mule() {
        cout << "muel destructor" << endl;
    }
};
```

​                                       <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240515170339532.png" alt="image-20240515170339532" style="zoom:67%;" />

多继承的情况下，先继承，先构造。注意，age在donkey和horse种均有继承，所以在mule继承时已经有三个age了。

* 只继承一个带数据属性的类
* 多个只有方法的类（类似于java中的接口）

下面是一个玩法

```cpp
class Uncopyable {
public:
    Uncopyable() = default;
    Uncopyable(const Uncopyable&) = delete;
    Uncopyable &operator=(const Uncopyable &) = delete;
};
class B: public Uncopyable {

};
class A: public Uncopyable {

};
```

继承完成后，A和B都是不可被拷贝构造和赋值的。

#### 类指针

注意，类的成员函数不能直接转换成指针，因为传参当中有个隐式的`this`指针

```cpp
class A {
public:
    int x, y;
    A():x(100), y(200) {}
    void vxLogin() {
        cout << "weixin login" << endl;
    }
    void zhiFuLogin() {
        cout << "zhiFulogin" << endl;
    }
    void upLogin() {
        cout << "uplogin" << endl;
    }
};
int main() {
    //不能直接转换
    //类的成员函数会偷偷地传递函数指针
    void (A*p)() = &A::vxLogin;
```

正确做法

```cpp
void (A::*p)() = &A::vxLogin;
```

### 强制转换

#### 隐式转换

```cpp
//隐式转换
int a = 3.5
a = 3;
double b= a + 3.5;
b = 6.5;
double c = (int)3.5 + 3.5;
c = 6.5;
```

c++风格

```cpp
double b = static_cast<int>(3.6) + 5;
```

#### dynamic_cast

`dynamic_cast`动态转换，确定指向到底是谁。主要用于多态情况下的确定。

```cpp
    Animal *p;
    switch(rand() % 2) {
        case 0:
            p = new Horse;
            break;
        case 1:
            p = new Donkey;
            break;
    }
    p->run();
    cout << dynamic_cast<Horse *>(p) << endl;
    cout << dynamic_cast<Donkey *>(p) << endl;
	//转换失败会是0否则是对应的地址
```

`dynamic`是基于虚表中的`type info`信息判断的，专业的名字叫`RTTI`(运行时的类型信息)。编译器默认是打开的，`dynamic_cast`基于此来判断

`-fno-rtti`禁止`RTTI`

欺骗`dynamic_cast`

```cpp
    Horse h;
    Donkey y;
    Animal *k = &h;
    cout << "==========" << endl;
    cout << dynamic_cast<Donkey *>(k) << endl;
    *(void **)&h = *(void **)&y;
    cout << dynamic_cast<Donkey *>(k) << endl;
    cout << "==========" << endl;
```

如果没有虚函数，将无法发产生虚表，也就是说`RTTI`不存在。那么在这种情况下，依旧进行对象指针的互相转换，如何实现呢？

析构函数就应该是虚的。如果析构非虚，

```cpp
class Animal {
public:
   void run() {
        cout << "I don't how to run" << endl;
    }
    //析构函数就应该是虚的
    Animal () {
        cout << "Animal constructor" << endl;
    }
    ~Animal() {
        cout << "Animal destructor" << endl;
    }
};
class Horse:public Animal {
public:
    void run() {
        cout << "fun fast" << endl;
    }
    void say() {
        cout << "horse say" << endl;
    }
    Horse() {
        cout << "Horse constructor" << endl;
    }
    ~Horse() {
        cout << "Horse destructor" << endl;
    }
};
Animal *p = new Horse;
delete p;
```

​                           <img src="C:\Users\wangbanjin\AppData\Roaming\Typora\typora-user-images\image-20240517114821750.png" alt="image-20240517114821750" style="zoom:67%;" />

这里没有Horse的析构

如果析构设置成虚

```cpp
 virtual ~Animal() {
        cout << "Animal destructor" << endl;
 }
```

​                           <img src="C:\Users\wangbanjin\AppData\Roaming\Typora\typora-user-images\image-20240517115033162.png" alt="image-20240517115033162" style="zoom:67%;" />

### 左值 右值 左值引用 右值引用

左值就是一块内存，可以往里面写数据。右值，则是可以把其中的内容读进来。

判断准则：在后续是否可以对其进行访问。于是，x++和++x的区别就出来了，前者右值，后者左值。

讲几个特殊的

```cpp
x + 3; //右值
x *= 33; //左值
y += 33; //左值
y * 3; //右值
```

*=和+=可以回想运算符重载，返回的是引用。

左值引用和右值引用，在写法上在左值引用的的基础上再加一个`&`

```cpp
//左值引用
void __func(int &x, string str) {
    cout << str <<  "left value" << endl;
    return ;
}

//右值引用
void __func(int &&x, string str) {
    cout << str <<  "right value" << endl;
    return ;
}
//万能
void __func(const int &x, string str) {
    cout << str <<  "right value" << endl;
    return ;
}
```

在上述函数当中，`int &x`中 `x`是一个左值， `int &&x`中 `x`此时也是一个左值。

如何再变回来呢？

```cpp
//右值引用
void __func(int &&x, string str) {
    cout << str <<  "right value" << endl;
    __func(move(x));
    return ;
}
```

`forward`更加灵活`forward<int&&>(x)`再次改回右值。

注意`const int &x`可以接受左值，也可以接受右值，接受所有找不到家的人。所以我们在进行拷贝构造的时候会习惯上加一个`const`，也有这个原因。

什么是右值引用？

```cpp
int &&r = 123;
```

补全最后一块设计拼图，把右值变回左值。引用的作用就是把一个右值变成左值。

```cpp
class A {
    int x;
public:
    A (int x = 0):x(x) {
        cout << this << ":default constuctor" << endl; 
    }
    A operator+(const A &a) {
        return A(x + a.x);
    }
    A(const A &a ) {
        cout << this << "copy constructor" << endl;
    }
    ~A() {
        cout << this << "destructor" << endl;
    }
};
A a(1), b(2);
A &&r = a + b;
```

使用右值引用将少一次拷贝构造，右边的匿名对象被盘活了。

```cpp
const int &a = 123;
//将123变成了一块左值，只是它是只读的
int &&b = 123;
```

​                <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240528214239164.png" alt="image-20240528214239164" style="zoom: 50%;" />

二者都是把右值变成了左值，只是前者是只读，后者可写。

​                 <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240529211403816.png" alt="image-20240529211403816" style="zoom:50%;" />

对于函数本身而言，三者没有任何区别。真正区别在于调用的时候绑定的顺序不一样，左值和右值会去不同的分支。在传入右值的时候去到特殊的分支，从而有特殊的操作。

拷贝构造

```cpp
   //利用传右值的引用，来进行拷贝构造，从而节省一次拷贝构造，也被叫做移动构造
    //千万注意，我们移动构造直接抢空间，然后让将原来对象和该空间的联系删除即可
    vector(vector &&v):__size(v.__size), data(v.data) {
        cout << "move constructor" << endl;
        v.__size = 0;
        v.data = nullptr;
    }
```

把临时的对象直接掏过来使用。

### auto关键字

如果是正常的变量的定义，建议还是指明具体的变量类型。当遇到迭代器这类，比较冗长的情况，可以使用`auto`进行自动推断。如果想查看具体的类型，可以使用`typeid`查看。`auto`不支持定义数组。

```cpp
cout << typeid(x).name() << endl;
auto c = 3.5;
if (type(c).hash_code() == type(double).hash_code())
    cout << "c is double" << endl;
```

### 静态局部变量

静态局部变量的特性

1. **生命周期**：静态局部变量的生命周期从第一次创建它的函数调用开始，直到程序结束。它在整个程序的运行期间一直存在。
2. **唯一性**：静态局部变量在函数的整个生命周期内只会初始化一次。即使函数被多次调用，静态局部变量也不会重复初始化。

线程安全的保证

从 C++11 开始，静态局部变量的初始化是线程安全的。也就是说，如果多个线程同时访问一个静态局部变量，并且该变量尚未初始化，C++11 及以后的标准会保证只有一个线程执行初始化，其他线程会等待初始化完成。

示例代码分析

```
cpp复制代码LayerRegisterer::CreateRegistry& LayerRegisterer::Registry() {
    static CreateRegistry* kRegistry = new CreateRegistry();
    CHECK(kRegistry != nullptr) << "Global layer register init failed!";
    return *kRegistry;
}
```

在这个函数中：

- `static CreateRegistry* kRegistry = new CreateRegistry();` 这一行声明并初始化了一个指向 `CreateRegistry` 对象的静态局部指针变量 `kRegistry`。
- 由于 `kRegistry` 是静态局部变量，它在第一次调用 `Registry` 函数时会被初始化，并且在整个程序运行期间只会初始化一次。
- 即使多个线程同时调用 `Registry` 函数，C++11 及以后的标准保证 `kRegistry` 只会被初始化一次，这就是为什么 `kRegistry` 是唯一的并且只会被创建一次。

线程安全的实现（C++11 及以后）

C++11 之前，静态局部变量的初始化不是线程安全的，必须手动使用互斥锁来保证线程安全。C++11 及以后，这种初始化由编译器和运行时库自动处理，保证了线程安全。

例如，在 C++11 及以后的实现中，可以放心地使用静态局部变量而无需额外的同步措施：

```
cpp复制代码LayerRegisterer::CreateRegistry& LayerRegisterer::Registry() {
    static CreateRegistry* kRegistry = new CreateRegistry();
    CHECK(kRegistry != nullptr) << "Global layer register init failed!";
    return *kRegistry;
}
```

这个实现可以确保 `kRegistry` 只会被初始化一次，并且在多线程环境下也是安全的。

### 显示转换

```cpp
const int x = 123;
int *p = const_cast<int *>(&x);
*p = 456;
cout << &x << endl;
cout << x << endl;
cout << p << endl;
cout << *p << endl;
```

在汇编的时候x这个符号就是123，在读取的时候显示123。

### constexpr关键字

```cpp
const int x = 123;
//要求在编译的时候就需要确定表达式的值
constexpr int y = 456;
```

而`const`则是在运行的时候确定，要求会低一点。

`constexpr`也可以用来修饰类和函数，在编译阶段就确定他们。

```cpp
class A{
   public:
    constexpr A() {}
}；
    constexpr A a;
```

### nullptr和NULL

测这两的类型

```cpp
#include<typeinfo>
void *p = NULL;
void *k = nullptr;
cout << typeid(NULL).name() << endl;
cout << typeid(nullptr).name() << endl;
```

在c语言里`NULL`功能很强大，既可以表示0又可以表示万能的指针`(void*)0`。在c++当中，继承了c语言里`NULL`的功能，`nullptr`只表示一根空指针。为什么要这样切割呢？

举个栗子

```cpp
void test(int x) {
    cout << __PRETTY_FUNCTION__ << endl;
    return;
}
void test(int* x) {
    cout << __PRETTY_FUNCTION__ << endl;
    return;
}
//将无法区分到底应该掉哪个函数
test(NULL);
//jiang'nen第二个函数
test(nullptr)
```

### 智能指针实现

```cpp
class A {
    public:
        A() {
            cout << this << ":constructor" << endl;
        }
        ~A() {
            cout << this << ":destructor" << endl;
        }
        void output() {
            cout << "hahaha" << endl;
        }
};
namespace kkb {
    class shared_ptr {
        A *obj;
        mutable int *cnt;
    public:
        shared_ptr(): obj(nullptr), cnt(nullptr) {}
        shared_ptr(A *obj):obj(obj), cnt(new int(1)) {}
        shared_ptr(const shared_ptr &s):obj(s.obj), cnt(s.cnt) {
            if (cnt != nullptr)
                *cnt += 1;
        }
        shared_ptr &operator=(const shared_ptr &p) {
            //先处理旧空间
            if (this->obj != p.obj) {
                if (cnt != nullptr) {
                    *cnt -= 1;
                    if (*cnt == 0) {
                    delete obj;
                    delete cnt;
                    }
                }
            }
            //指向新的空间和计数器
            cnt = p.cnt;
            obj = p.obj;
            if (cnt != nullptr)
                *cnt += 1;
            return *this;
        }
        A *operator->() {
            return obj;
        }
        A &operator*() {
            return *obj;
        }
        ~shared_ptr() {
            if (cnt != nullptr) {
                *cnt -= 1;
                if (*cnt == 0) {
                    delete obj;
                    delete cnt;
                }
            }
            obj = nullptr;
            cnt = nullptr;
        }
        int use_count() {
            return *cnt;
        }
    }; 
}
```

### 排序算法实现

### 时间检测工具

### function模板实现

多态的体现。

基类写为一个纯虚类`Base`指向任意普通函数`NormalFunction`和任意函数对象`functor`

```cpp
template <typename T, typename ...ARGS>
class Base {
public:
	virtual T run(ARGS...) = 0;
    //virtual Base<T, ARGS...> *getCopy(ARGS...) = 0;
    virtual Base<T, ARGS...> *getCopy() = 0; 
    virtual ~Base();
};
```

* 指向任意函数。这里存的是任意函数指针

  ```cpp
  template <typename T, typename ...ARGS>
  class NormalFunction:public Base<T, ARGS...> {
  private:
      //普通函数指针。
      T (*ptr) (ARGS...);
  public:
      NormalFunction (T(*p) (ARGS...)) : ptr(p) {}
  
      T run (ARGS ...args) override {
          //保证args的类型不发生改变 左值还是左值 左值引用还是左值引用等
          return ptr(std::forward<ARGS>(args)...);
      }
      
      Base<T, ARGS...> *getCopy() override{
          return new NormalFunction(*this);
      }
  };
  
  
  ```

  

* 指向函数对象。存任意类型的函数对象

  ```cpp
  template <typename CLASS_T, typename T, typename ...ARGS>
  class functor:public Base<T, ARGS...> {
  private:
      CLASS_T __obj;
  public:
      functor(CLASS_T &obj):__obj(obj) {}
  
      T run(ARGS ...args)override {
          return __obj(forward<ARGS>(args)...);
      }
      Base<T, ARGS...> *getCopy() {
          return new functor(*this); 
      }
  };
  ```

  最终效果

```cpp
template <typename T, typename ...ARGS> class function;
template <typename T, typename ...ARGS> 
class function<T(ARGS...)> {
private:
    //不能直接定义抽象类的对象，但是可以定义抽象类的指针
    Base<T, ARGS...> *ptr;
public:
    function(T (*p)(ARGS...)):ptr(new NormalFunction<T, ARGS...>(p)) {}

    template <typename CLASS_T>
    function(CLASS_T obj):ptr(new functor<CLASS_T, T, ARGS...>(obj)) {}
    T operator() (ARGS ...args) {
        return ptr->run(std::forward<ARGS>(args)...);
    }

    function &operator=(const function &f) {
        delete ptr;
        ptr = f.getCopy();
        return *this;
    }

    virtual ~function() {
        delete ptr;
    }
};
```



### 模板

把类型给抽象没了。比如，加分本身提取出来，支持所有对象。这就是泛型编程和模板编程。

函数模板-------->模板去解决面向过程的问题

类模板---------->模板去解决面向对象的问题

#### 函数模板

```cpp
template <typename T, typename U>
T add(T a, T b) {
    return a + b;
}
```

​             <img src="https://raw.githubusercontent.com/wangbanjin1/pictures/main/image-20240529212613905.png" alt="image-20240529212613905" style="zoom:50%;" />

模板的声明,只有调用的时候才会实例化一个相应的函数。模板的本质其实是帮助程序员写代码，在调用的时候帮助程序员生成函数。

如果我调用`add(1, 2.3)`,会报错的，我们可以显式调用`add<int>(1, 2.3)`。

或者

```cpp
decltype(T() + U())add(T a, U b) {
    return a + b;
}
```

但是注意如果`T`或者`U`的默认构造不存在的话 将无法使用`decltype`。将函数返回值类型后置将能解决这类问题。

```cpp
//使用函数的返回值后置
template<typename T, typename U>
auto add(T a, U b)->decltype(a + b){
    return a + b;
}
```

从而彻底解决了两个不同的数相加

#### 类模板

```cpp
//类模板
//template <typename T>
class PRINT {
public:
    //成员函数模板
    template <typename T>
    PRINT& operator()(T t){
        cout << t << endl;
        return *this;
    }
}
```

#### 引用折叠

```cpp
//这里T&&表示的是传T的引用，但是具体是左值引用还是右值引用需要编译器自己推导，
void swap(T &&a, T &&b) {
    //法1
    /*
    auto c = a;
    cout << typeid(c).name() << std::endl;
    a = b;b = c;
    */

    typename remove_reference<T>::type c = a;
    a = b; b = c;
    return ;
}
int main() {
    int m = 3;
    int n = 5;
}
```

比如说我调用一个`swap(m ,n)`此时传入的是两个左值，那么T会被推导`int&`

#### 模板特化

此时还是在`add`函数里，但是我们传入指针，那么按照之前的方法，模板将无法帮助我们继续写代码。

```cpp
template <typename T, typename U>
auto add(T *a, U *b)->decltype(*a + *b){
    return *a + *b;
}
int main() {
    int m = 123, n = 456;
    int *p = &m, *q = &n;
    cout << add(p, q) << endl;
    return 0;
}
```

考虑下面的特化模板的写法

```cpp
template <typename T, typename U>
class Test {
public:
    Test() {
        cout << "nomal template<T, U>" << endl;
    }
};

template <>
class Test<int, double> {
public:
    Test() {
        cout << "specialization template" << endl;
    }
};

template <typename T>
class Test<int, T> {
public:
    Test() {
        cout << "partial specialization template" << endl;
    }
};

int main() {
    Test<int, double> t1;
    Test<double, int> t2;
    Test<int, int> t3;
    return 0;
}
```

依次命中全特化，偏特化和普通模板。

​                             ![image-20240531142925228](C:\Users\wangbanjin\AppData\Roaming\Typora\typora-user-images\image-20240531142925228.png)

#### 不定参

用`...ARGS`表示不定参

```cpp
//相当于递归终止条件
template <class T>
void print(T a) {
    cout << a << endl;
}

template <class T, class ...ARGS>
void print(T a, ARGS... args) {
    cout << a << ",";
    print(args...);
}
```

用于取类的模板写法如下

```cpp
template <class T, class ...ARGS>
class N_ARGS {
public:
    typedef T type;
    typedef N_ARGS<ARGS...> rest;
};

template<class T>
class N_ARGS<T> {
public:
    typedef T type;
    typedef T last;
};
N_ARGS<int, double, string>::type a;
N_ARGS<int, double, string>::rest::type b;
N_ARGS<int, double, string>::rest::rest::last c;
```

加入函数模板

```cpp
template<class T, class ...ARGS>
class Test2 {
    T operator()(typename N_ARGS<ARGS...>::type a, typename N_ARGS<ARGS...>::rest::type b) {
        return a + b;
    }
}
    Test2<int (int, int)> t1;
    Test2<double (double, int)> t2;

    cout << t1(1.1,2.2) << endl;
    cout << t2(1.1,2.2) << endl;
```

#### 非类型模板

```cpp
template <int M>
void print() {
    cout << M << ",";
    print<M - 1>();
    return ;
}
//特化用于停止
template <>
void print<1> (){
    cout << 1 << endl;
    return ;
}
```

在上面的情况上继续优化

```cpp
template <int N, typename T, typename ...ARGS>
//struct 默认都是公有
struct C_Args {
    typedef typename C_Args<N - 1, ARGS...>::type type;
};

template <typename T>
//struct 默认都是公有
struct C_Args<1, T> {
    typedef T type;
};

template <typename T, typename ...ARGS>
//struct 默认都是公有
struct C_Args<1, T, ARGS...> {
    typedef T type;
};
```

可以使用类似`C_Args<1, ARGS...>::type`来取得参数

获得参数个数的工具类

```cpp
//获得参数个数工具类
template <typename T, typename ...ARGS>
struct NUM_ARGS {
    //在编译的时候就确定好函数参数的个数
    static constexpr int n = NUM_ARGS<ARGS...>::n + 1;
};

template <typename T>
struct NUM_ARGS<T> {
    static constexpr int n = 1;
};
```

#### 高级萃取工具

```cpp
//自己实现的remove_reference模板类
template<typename T>
struct remove_reference {
    typedef T type;
};
template<typename T>
struct remove_reference<T&> {
    typedef T type;
};

template<typename T>
struct remove_reference<T&&> {
    typedef T type;
};
```

测试一下

```cpp
    typename remove_reference<int>::type a;
    typename remove_reference<int &>::type b;
    typename remove_reference<int &&>::type c;

    cout << typeid(a).name() << endl;
    cout << typeid(b).name() << endl;
    cout << typeid(c).name() << endl;
```

#### bind

把函数变成函数对象，可以自定义修改传参个数和顺序

```cpp
void add1(int &x) {
    x++;
    cout << x << endl;
}

void show(int n, const char *msg) {
    cout << n << ":" << msg << endl;
}

int main() {
    int m = 3, n = 4;
    //直接传递m的话，m并不会自增，需要指明此时的m是一个引用
    auto t2 = bind(add1, ref(m));
    t2(); 
    t2(); 
    t2(); 
    cout << m << endl;
    return 0;
}
```

### c++11标准的多线程

引入标准头文件`#include<thread>`之后直接使用`thread`类即可

```cpp
void func() {
    cout << __PRETTY_FUNCTION__ << endl;
}
thread t1(func);
t1.join();
cout << __PRETTY_FUNCTION__ << endl;
```

在linux当中，编译链接的时候添加`-lpthread`命令

在上锁的时候通常来说尽可能使得上锁的区域小一点

```cpp
#include <mutex>
mutex mtx;
int count_prime(int l, int r) {
    for (int i = l;i <= r;i++) {
        int x = is_prime(i);
        unique_lock<mutex> lock(mtx);
        cnt += x;
        lock.unlock();
    }
    return cnt;
}
```

以素数计算为例,只在临界变量`cnt`被访问到的地方加锁即可。linux下也可以使用原子操作`__sync_fetch_and_add(&cnt, x)`

#### 一个简单的线程池

```cpp
class Task {
public:
    template<typename FUNC_T, typename ...ARGS>
    Task(FUNC_T f, ARGS... args) {
        func = bind(f, std::forward<ARGS>(args)...);
    }
    void run() {
        func();
    }
private:
    function<void()> func;
};

class ThreadPool {
public:
    ThreadPool(int n = 1):trr(n), startFlag(false) {
        this->start();
    }
    void worker() {
        auto id = this_thread::get_id();
        running[id] = true;
        while (running[id] == true) {
            //获取任务
            Task* task = getTask();
            task->run();
            //执行
            delete task;
        }
        return;
    }
    Task* getTask() {
        unique_lock<mutex> lock(mtx);
        //处理假醒状态
        while (taskQueue.empty()) {
            condition.wait(lock);
        }
        Task* t = taskQueue.front();
        taskQueue.pop();
        return t;
    }

    template<typename FUNC_T, typename ...ARGS>
    void addTask(FUNC_T f, ARGS... args) {
        //不定参保持原有的类型
        //上锁
        unique_lock<mutex> lock(mtx);
        taskQueue.push(new Task(f, std::forward<ARGS>(args)...));
        //解锁同时通知队列中有人物的条件已经达成
        //todo
        condition.notify_one();
        return;
    }

    void stop() {
        if (startFlag == false)
            return;
        for (int i = 0;i < trr.size();i++) {
            //添加毒药任务
            addTask(&ThreadPool::stopRunning, this);
        }
        for (int i = 0;i < trr.size();i++) {
            trr[i]->join();
        }
        for (int i = 0;i < trr.size();i++) {
            delete trr[i];
            trr[i] = nullptr;
        }
        return;
    }

    void start() {
        if (startFlag == true) {
            return;
        }
        for (int i = 0;i < trr.size();i++) {
            trr[i] = new thread(&ThreadPool::worker, this);
        }
        startFlag = true;
        return;
    }
    virtual ~ThreadPool() {
        this->stop();
        while (!taskQueue.empty()) {
            delete taskQueue.front();
            taskQueue.pop();
        }
    }
private:
    void stopRunning() {
        auto id = this_thread::get_id();
        running[id] = false;
        return;
    }
    vector<thread*> trr;
    unordered_map<thread::id, bool> running; 
    queue<Task*> taskQueue;
    mutex mtx;
    condition_variable condition;
    bool startFlag;
};
```

### 异常

这里先介绍一个概念，**异常安全**指的是，当异常发生时, 既不会发生资源泄露，系统也不会处于一个不一致的状态。我们以矩阵乘法为例，注意这个是伪代码，`result`还没实现。

```cpp
class matrix {

public:
   matrix(size_t nrows, size_t ncols):nrows_(nrows), ncols_(ncols) {
       data_ = new float[nrows * ncols];
   }
   ~matrix() {
       delete[] data_;
   }
   friend matrix operator*(const matrix&, const matrix&);
private:
    float* data_;
    size_t nrows_;
    size_t ncols_;
};
matrix operator*(const matrix& lhs, const matrix& rhs) {
   if (lhs.ncols_ != rhs.nrows_) {
       throw std::runtime_error("matrix sizes mismatch");
   }
   matrix result(lhs.nrows_, rhs.ncols_);
}
```

错误可能出现的地方如下：

* 内存分配失败，一般会得到`bad_alloc`，对象构造失败。此时，在catch捕捉到这个异常之前，所有的栈上的对象会全部被析构，资源全部清理完成。（不理解这段的，可以去看看栈展开）
* 矩阵长宽不满足矩阵乘法的条件，抛出异常即可
* 乘法函数内存分配失败，`result`没构造出来，此时仍然安全
* 乘法过程失败，析构函数自动释放空间，同样不会出现资源泄露

以`vector`里的`at`方法为例，访问越界会报`out of range`的访问越界的错误。

​          <img src="C:\Users\wangbanjin\AppData\Roaming\Typora\typora-user-images\image-20240602142845687.png" alt="image-20240602142845687" style="zoom:67%;" />

所以使用`try`和`catch`

```cpp
try {
        vector<int> arr(10);
        for (int i = 0;i < 10;i++) {
            arr[i] = rand();
        }
        cout << arr[-1] << endl;
        cout << arr.at(-1) << endl;
        cout << "check point 1" << endl;
    } catch(...) {
        cout << "catch an exception" << endl;
    }
```

`...`可以改成特定的异常比如说`out_of_range &e `访问越界的异常，但是记住引用头文件`#include<stdexcept>`。

```cpp
try {
        vector<int> arr(10);
        for (int i = 0;i < 10;i++) {
            arr[i] = rand();
        }
        cout << arr[-1] << endl;
        cout << arr.at(-1) << endl;
        cout << "check point 1" << endl;
    } catch(out_of_range &e){
        cout << "catch an exception" << endl;
        cout << e.what() << endl;
        //如果写在前面，抓捕异常将会直接到此分支
    } catch(exception &e) {
        cout << e.what() << endl;
    } catch(...) {
        cout << "catch an obj" << endl;
    }
```

当然也可以自己`throw`自定义一个对象，然后在`catch`里取抓住。

另一种常见的用法是在自定义的函数中就`throw`调用者再放`try`里。

```cpp
int sum(int l, int r, vector<int> v)throw(out_of_range,int,exception){
    int sum=0;
    for (int i = l;i < r;i++) {
        sum += v.at(i);
    }
    return sum;
}
```

如果一定可以保证没有异常可以声明`noexcept`

再一种常见的用法是去继承`exception`这个类

```cpp
class ZeroDivException:public runtime_error {
    public:
    	ZeroDivException(const string &msg = "zero div"):runtime_error(msg){}
    	const char *what()const noexcept override {
            retrun "__what__zero__div";
        }
}

try {
    //零除错误
    cin >> n;
    int x = 10 / n;
    if (n == 0)
    	throw(ZeroDivException());
}
```

#### 构造和析构是否应该抛出异常

先说结论。从语法上说都是可以抛出异常的。

实际上，析构是不应该抛出异常的，而且其缺省的修饰就是`nexcept`的。因为一旦抛出异常，异常之后的代码将无法执行，容易造成资源泄漏。如果非要抛出异常，那就应该在析构里面捕获并且处理完成，不要将控制权转移到析构外面。

对构造函数而言异常是必要的，因为抛出异常才能告诉程序构造函数失败的原因。抛出异常时，已经构造好的非动态申请的对象将逆序被析构，而回收资源。背后的原因是栈展开和析构。所以，这里也进一步回答了析构中最好不要抛出异常。构造函数抛出异常时，对于动态申请的资源，有两种方式处理，要么在捕获异常后手动地释放资源，要么考虑使用智能指针。如果是`new`申请内存失败的情况，会抛出`bad_alloc`的异常，会层层往上回溯，如果没被`catch`，最后将由`std::terminate`终止程序。

### 设计模式

它适用于面向对象的语言，结合封装，继承和多态的特性，来设计程序顶层结构的一种方法。

##### 单例模式



### 线程安全问题

[多线程](https://so.csdn.net/so/search?q=多线程&spm=1001.2101.3001.7020)操作一个共享数据的时候，保证所有线程的行为是符合预期的则称为线程安全。

#### [智能指针](https://so.csdn.net/so/search?q=智能指针&spm=1001.2101.3001.7020)的线程安全

##### 2.1 智能指针的线程安全隐患（shared_ptr）

主要是以下几个方面：
(1) 引用计数的加减操作是否线程安全。
(2) 修改`shared_ptr`指向是否线程安全。
(3) `shared_ptr<T>`的`T`的并发操作的安全性，也应该被讨论。

##### 2.1结论

1）同一个shared_ptr被多个线程“读”是安全的；

2）同一个shared_ptr被多个线程“写”是不安全的；

`shared_ptr`拷贝时发生下列行为

1）拷贝智能指针指向的资源（非原子操作）

2）增减引用计数（原子操作）

![img](https://raw.githubusercontent.com/wangbanjin1/pictures/main/75cb5080cf544f92a69ec1292f7cbb1e.png)

3）共享引用计数的不同的shared_ptr被多个线程”写“ 是安全的；
