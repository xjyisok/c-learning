# c-learning
my note of c++
# c++ 编译和链接
在 c++中代码的运行分为两个阶段，编译和链接以下为例  
在`log.cpp`中实现一个简单的函数如下
```
void log(const char* message){  
std::cout<<message<<std::endl}
```
在`main.py`中对其进行声明 
```
#include<iostream>
using namespace std
void log(const char *message)
int main(){
const char* message="xjy"  log(message)}
```
在编译阶段呢编译器会通过main入口寻找log声明，但是并不执行log函数，只有在link阶段才会在整体项目中寻找log函数的具体实现。
这里将
```
#include<iostream>
using namespace std
//void log(const char *message)
int main(){
const char* message="xjy"  log(message)}
```
中的log声明注释掉`ctrl+f7`编译会报错，但是`f5`运行在执行link后虽然依旧有报错但是还是可以正常打印`xjy`  
此外在编译阶段由于不存在链接操作，因此即使在log.py中log函数被错写为logr函数依旧不影响编译，但是在链接期间则会出现`unresolved external symbol`  
表示未找到函数。

### tips
```
void log(const char* message);
int mulply(int a,int b) {
	log("ab");
	return a * b;
}
int main() {
	const char* message = "chenro";
	log(message);
}
```
即使在main项目入口中mulply没有使用到，是死代码，但是若`log.py`中log函数名错写为`logr`在link阶段依旧会报错`unresolved external symbol`  
这是由于编译器会默认这个函数在项目的其他部分可能被调用，也会参与到link中。
static关键字可以解决这一问题，在mulply前添加static关键字可以告诉编译器函数只在当前cpp文件中会被调用，因此若入口函数中没有调用mulply函数也不参与link
## 函数定义
c++和python的函数定义有巨大不同,python允许在项目文件的不同位置重复定义函数，但是c++不行，正常情况下在整个cpp项目中同一个函数只允许被定义一次，否则编译器会无法确定链接哪个导致链接错误。如果需要在不同cpp文件中调用同一个函数一般有一下三个方法
以一个如下结构的项目为例子：  
```
log.cpp
init.h
main.cpp
```
### `log.cpp`  
```
#include<iostream>
#include "init.h"
void initlog(const char* message) {
	log(message);
}
```
### `init.h`  
```
void log(const char* message) {
	std::cout << message << std::endl;
}
```
### `main.cpp`  
```
#include <iostream>
#include "init.h"
void log(const char* message);
int mulply(int a,int b) {
	log("ab");
	return a * b;
}
int main() {

	const char* message = "chenro";
	log(message);
}
```
### 1:  
`#include`关键字本质上就是将对应文件中的内容粘贴到当前文件中include的位置，所以在build整个项目文件时相当于在log.cpp和main.cpp中重复定义了log函数导致link错误。但是在项目中不可避免的函数需要重复调用，避免这一问题的第一种方式就是static关键字，在`init.h`中为log函数田间static关键字，那么在引用时相当于`main.cpp`和`log.cpp`中都定义了一个静态的函数，在link阶段其他文件无法访问也就不会发生重复链接错误。
### 2：
`inline`关键字。inline关键字在将函数调用转变为函数内容调用。避免重复定义，为init.h重的log函数添加inline关键字，这样log.cpp和main.cpp引用的实际上是`std::cout << message << std::endl;`仅仅执行log函数操作而不做定义
### 3：  
`init.h`头文件只保存函数声明，在第三方组织代码中保存函数定义，这样每次应勇init.h只会包含log函数声明，在link阶段编译器会自己根据函数声明到第三方组织代码中去寻找具体函数实现。
# 指针和引用：
## 指针定义  
```
int a=114514;
int* p=&a;
```
或者
```
int* p;
*p=114514;
```
指针指向变量的地址是一个保存变量地址的整数
```
int a=114514;
int* p=&a;
*p=123;
```
此时a=123。
## 引用定义
```
int a=123;
int& b=a;
b=114514;
```
此时a=b=114514,b只存在于源码中，在编译时不会被创建，对b做的一切操作都相当于对a进行同样的操作，b是a的引用相当于为a提供了一个别名。
常用例子：  
```
void add(int& value){
value++;
}
void shift(int* a,int* b){
int temp=*a;
*a=*b;
*b=temp;
}
```
# 类(默认private)
## 类的定义和初始化  
```
class player{
public:
int x,int y;
}
int main(){
player p;
p.x=1,p.y=2;
}
```
## 类中的函数
```                                      
class player{                         
public:
int x,int y;
void move(int ax,,int ay){
x+=ax;
y+=ay;
}
}
int main(){
player p;
p.x=1,p.y=2;
p.move;
}
```
等价于  
```
class player{                         
public:
int x,int y;
}
void move(player& p,int xa,int ya){
p.x+=xa;
p.y+=ya;
}
```
## static  
c++项目中不允许存在两个相同名字的全局变量。如果要在一个cpp文件中使用另一个cpp文件中定义的全局变量可以采用以下方式  
`extern`关键字，extern让编译器在项目的其他地方寻找同名变量，而非重新定义。__!!! extern后面跟着的必须是全局变量，静态变量不行，静态变量无法被外部编译单元访问，因此编译器无法通过extern在外部寻找该变量__
## 类中的静态
```
class Entity{
public:
static int x,y;
void printxy(){
std::cout<<x<<y<<std::endl;
}
}
int Entity::x,int Entity::y;
int main(){
Entity e;
Entity::x=1;
Entity::y=2;
Entity e1;
Entity::x=3;
Entity::y=4;
e.printxy;
}
```
__输出为3，4__
如果printxy也是静态的那么甚至不需要定义实例```Entity::=1，Entity::y=2,Entity::printxy```同样实现一样的功能，因为所有操作都是静态的。
__此外静态方法无法访问非静态变量!!!__  
__原因__：  
1. 实例依赖性
非静态成员变量依赖于具体的类实例。它们在内存中的存储是与特定对象相关联的。因此，除非你创建了一个类的对象，否则这些成员变量根本不存在。

2. 静态函数的调用上下文
静态函数可以在没有任何对象实例的情况下被调用。如果静态函数被允许访问非静态成员变量，当静态函数在没有任何对象存在时被调用，这将引发问题：它尝试访问哪一个对象的成员呢？由于不存在具体的对象，这样的访问将是不确定的，甚至是非法的。

3. 设计上的一致性
限制静态函数只能访问静态成员，这种设计强化了类设计中成员的独立性。这使得静态成员可以作为全类共享的资源，而非静态成员则属于各个实例。
## local static  
```
int i=0;//全局变量全局可访问。
void function(){
i++;
std::cout<<i<<std::endl;
}
int mian(){
for(int j=0;j<5;j++){
function();}
}//输出为1，2，3，4，5
```
这里的i全局可被访问，因此呢可以在函数运行的任何地方修改i的值。这是不安全的，为了解决这一问题可以在函数中定义local static变量i如下  
```
void function(){
static int i=0;
i++;
std::cout<<i<<std::endl;
}
```
这里的i在函数体中被唯一定义一次且函数外部不可访问。
## 单例类
所谓单例类即在整个运行周期内该类只有一个实例，所有操作都是对这唯一实例做操作，单例类可如下定义。
```
class singleton {
private:
	singleton() {}
	singleton(const singleton&) = delete;
	singleton& operator=(const singleton&) = delete;
public:
	int i;
	static singleton& Get() {
		static singleton instance;
		return instance;
	}
	void hello() { std::cout << "hello" << std::endl; }
};
int main() {
	// 获取单例的引用
	singleton& instance1 = singleton::Get();
	instance1.i = 1;  // 设置实例的成员变量 i
	std::cout << "Instance1.i = " << instance1.i << std::endl;  // 输出应为 1

	// 再次获取单例的引用
	singleton& instance2 = singleton::Get();
	std::cout << "Instance2.i = " << instance2.i << std::endl;  // 输出应为 1，验证单例的一致性

	// 修改 instance2 的 i 值
	instance2.i = 2;
	std::cout << "Instance1.i = " << instance1.i << std::endl;  // 输出应为 2，验证 instance1 和 instance2 是同一个实例
ue
	// 调用成员函数
	instance1.hello();  // 输出 "hello"

	// 尝试复制和赋值操作的编译错误验证（如果需要验证这部分，请取消注释下面的代码）
	// singleton instance3 = instance1;  // 应当因为删除的复制构造函数而出错
	// singleton instance4;
	// instance4 = instance2;  // 应当因为删除的赋值运算符而出错

	return 0;
}

```
__注意！！！__:  
  ```
  singleton() {}
	singleton(const singleton&) = delete;
	singleton& operator=(const singleton&) = delete;
```
这里的```singleton() {}```被设置为private是阻止在类外部实例化	
如在main函数中```singleton a```会出现编译错误。	
```singleton(const singleton&) = delete;```删除了拷贝构造函数。如果在main中进行```singleton s1=singleton::Get();singleton s2=s1```会出现编译错误	
```singleton& operator=(const singleton&) = delete;```防止的操作如下	
```singleton s1 = singleton::Get();
singleton s2;
s2 = s1; // 这行代码将会导致编译错误
```	
虽然因为singleton() {}已经被设置为private导致无法在外部直接创建类，但是```singleton& operator=(const singleton&) = delete;```这句话依旧有意义如下	

1增强可读性和意图表达：	
  通过明确地删除拷贝构造函数和赋值运算符，代码更具可读性和自解释性，明确告诉阅读代码的人这个类是单例，不允许复制或赋值。	

2防止潜在的修改错误：	
  假设在未来某个时候，类的私有构造函数被修改为保护或公共构造函数，为了某些特定的原因（例如测试）。如果没有删除拷贝构造函数和赋值运算符，就有可能会引入复制和赋值错误。	

3符合单例模式的标准实现：	
  大多数单例模式的实现都会删除拷贝构造函数和赋值运算符，这是一个标准的实现方法，确保不会因为意外的复制或赋值导致单例属性失效。	

4编译时保护：	
  多层次的保护可以防止由于代码变更或维护不当而引入的错误。尽管在当前上下文中私有构造函数已经阻止了直接实例化，但删除拷贝构造函数和赋值运算符提供了额外的编译时保护。	
  
```singleton(const singleton&) = delete;```的意思是删除拷贝构造函数。拷贝构造函数的基本形式为```classname(const classname&);```在所给例子中类的拷贝构造函数为```singleton(const singleton& other):value(other.value){}```
在main函数中的应用场景  
```
Example a(10);       // 调用参数化构造函数
Example b(a);       // 调用拷贝构造函数
```
or
```
Example a=10;       // 调用参数化构造函数
Example b=a;       // 调用拷贝构造函数
```
上下两端代码对应的是c++类中的两种初始化方式自上而下分别是直接初始化和拷贝初始化  
__但是在有多个参数时参数构造函数只能使用直接初始化，但是拷贝构造函数则不受影响__
__conclusion__:对于拷贝构造函数直接初始化和拷贝初始化等价，但是对于参数构造函数，拷贝初始化只适用于单一参数
此外对于静态成员变量初始化不能在类的定义中直接赋值，必须在类的定义体外进行。C++11 引入了一些新的特性，使得常量静态成员变量可以在类内部初始化，但一般的静态成员变量仍需在类外部初始化。
例如
### 一般静态成员变量的初始化
```
#include <iostream>

class Example {
public:
    static int staticVar;  // 静态成员变量声明

    int nonStaticVar;      // 非静态成员变量

    Example(int val) : nonStaticVar(val) {
        std::cout << "构造函数: nonStaticVar = " << nonStaticVar << std::endl;
    }

    static void setStaticVar(int val) {
        staticVar = val;
    }

    static int getStaticVar() {
        return staticVar;
    }

    void display() const {
        std::cout << "静态成员变量: " << staticVar << ", 非静态成员变量: " << nonStaticVar << std::endl;
    }
};

// 在类定义体外初始化静态成员变量
int Example::staticVar = 0;

int main() {
    Example::setStaticVar(42);

    Example obj1(10);
    Example obj2(20);

    obj1.display();
    obj2.display();

    std::cout << "静态成员变量通过静态成员函数获取: " << Example::getStaticVar() << std::endl;

    return 0;
}

```
### 常量静态成员变量的初始化
```
class Example {
public:
    static const int staticConstVar = 42;  // 常量静态成员变量，可以在类内部初始化

    static void printStaticConstVar() {
        std::cout << "staticConstVar: " << staticConstVar << std::endl;
    }
};
```
## Enums枚举	
```
enum level{
A,B,C
};//默认A=0,B=1,C=2
```
__初始化__
```
level num=A;
```
__在类中__	
```
class Log{
public:
enums level{
A=0,B,C
};
void setlevel(level Level){
num=level;
std::cout<<num<<std::endl;
}
private:
level num=A;
}
int main(){
Log e;
e.setlevel(Log::B);
}
```
## Deconstructor
  在C++中，析构函数（destructor）是一个特殊的成员函数，它在对象生命周期结束时被自动调用以释放资源和执行清理操作。析构函数对于防止资源泄漏（如动态分配的内存未被释放）至关重要，特别是在涉及到资源管理的类设计中。
析构函数的特点包括：
  自动调用：当对象的生命周期结束时（例如对象的作用域结束或者通过delete被删除时），其析构函数会被自动调用。
  无参数和返回值：析构函数不接受任何参数，也不返回任何值。它的名字由波浪符（~）后跟类名组成，例如，如果类名为MyClass，则析构函数为~MyClass()。
  用途：通常用于释放对象在生命周期内分配的资源，如动态分配的内存、打开的文件句柄、网络连接等。
  单一性：每个类只能有一个析构函数，不能被重载，这与构造函数不同，后者可以有多个重载形式。
析构函数的自动调用性质确保了一旦对象不再被使用，所有的资源都将被正确地清理，这是资源管理中的一个重要机制。正确使用析构函数可以帮助避免内存泄漏等常见编程错误，是良好程序设计的基本要求。
```
#include <iostream>
#include <cstring>

class Car {
private:
    char* licensePlate;

public:
    // 构造函数
    Car(const char* license) {
        licensePlate = new char[strlen(license) + 1];
        strcpy(licensePlate, license);
    }

    // 析构函数
    ~Car() {
        delete[] licensePlate;
        std::cout << "Car destructor called, memory freed." << std::endl;
    }

    // 显示牌照号
    void displayLicense() const {
        std::cout << "License Plate: " << licensePlate << std::endl;
    }
};

int main() {
    Car myCar("ABC123");
    myCar.displayLicense();

    // 当main函数结束时，myCar的生命周期也结束了
    // 这将自动调用Car类的析构函数
    return 0;
}

```
在这段代码中定义了一个汽车类，在类构造函数中使用new在堆上创建了一个```lisencePlate```变量，在析构函数中使用delete函数手动进行内存释放，由于析构函数默认执行所以在实例生命周期结束后可以自动释放内存。
__情况2__
```
#include <iostream>

class Box {
public:
    Box() {
        std::cout << "Box is created" << std::endl;
    }
    ~Box() {
        std::cout << "Box is destroyed" << std::endl;
    }
};

int main() {
    // 使用new在堆上创建Box类的实例
    Box* myBox = new Box();

    // 做一些处理...

    // 最后，使用delete来释放分配的内存
    delete myBox;

    return 0;
}
```
如果使用new在堆山创建实例，那么必须使用delete显式释放内存析构函数才会执行。
此外需要注意的是每一个new都需要对应一个delete
样例如下:	
```
class Car {
private:
    char* licensePlate;

public:
    // 构造函数
    Car(const char* license) {
        licensePlate = new char[strlen(license) + 1];
        strcpy(licensePlate, license);
    }

    // 析构函数
    ~Car() {
        delete[] licensePlate;
        std::cout << "Car destructor called, memory freed." << std::endl;
    }
int main(){
Car mycar=new Car("BYD");
delete mycar;
}
```
上述代码可以完成```Car```类实例的创建和释放，但是如果将```delete[] licensePlate```删除，那么即使使用delete释放实例mycar也无法释放为licensePlate在堆上分配的内存。同样的如果不使用delete，析构函数不执行licensePlate内存同样无法释放。
在类和结构体中delete一个实例的作用实际上是触发该实例的析构函数，所以要想释放在堆上创建的数据对象必须在析构函数中使用delete释放内存。
此外如果在C++中你使用new创建了一个类的实例，即使这个类中不包含任何数据成员，你仍然需要使用delete来释放这个实例所占用的内存。这是因为new操作符分配的不仅仅是类成员的内存，还包括为该对象本身维护的一些额外的内存管理信息。举个例子，即使是一个空类	
```
class EmptyClass {};

// 在堆上创建实例
EmptyClass* myObject = new EmptyClass;

// 代码执行...

// 需要使用delete来释放内存
delete myObject;
```
在这种情况下，EmptyClass可能看起来没有分配任何有用的数据，但是通过new创建的每个对象都占用了一定的内存空间，这些内存用于支持对象的运行时信息（如类型信息和析构函数的调用）。因此，即使类是空的，你也需要用delete来释放这部分内存，避免内存泄漏。

这也是为什么推荐在使用完动态分配的对象后总是确保调用delete，或者更好地，使用智能指针来自动管理内存，从而避免忘记手动释放内存。智能指针如std::unique_ptr和std::shared_ptr在销毁时会自动调用delete，从而简化内存管理。
## 继承	
```
#include<iostream>
#include<string>
class Entity {
public:
	virtual std::string getname() {
		return "Entity";
	}
};
class Player :public Entity {
private:
	std::string m_name;
public:
	Player(const std::string& name):m_name(name){}
	std::string getname() override {
		return m_name;
	}
};

void print(Entity* e) {
	std::cout << e->getname() << std::endl;
}
int main() {
	Entity* e=new Entity();
	std::cout << e->getname() << std::endl;
	Player* p=new Player("chenro");
	std::cout << p->getname() << std::endl;
	print(p);
	Entity* entity = p;
	std::cout << entity->getname() << std::endl;
}
```
上述代码中一共涉及到三个知识点：__虚函数__,__接口__,__继承__	
###__1虚函数__:	
__1. 虚函数表（vtable）__
每个包含虚函数的类都有一个虚函数表。这个表是一个函数指针数组，每个指针指向一个虚函数的具体实现。当类被继承并且虚函数被重写时，派生类的虚函数表会被更新，使得相应的函数指针指向新的函数实现。

__2. 对象的指针和虚函数表__
每个对象都包含一个指向其类的虚函数表的指针（通常称为vptr）。这个指针在对象创建时由构造函数设置，确保它指向正确的虚函数表。如果对象属于派生类，它的虚函数表将包含指向重写函数的指针。

__3. 运行时的函数调用解析__
当通过基类的指针或引用调用虚函数时，C++运行时不是查看指针的类型，而是查看指针指向的对象的虚函数表，并通过这个表来调用正确的函数实现。这就是为什么即使指针类型是基类，实际调用的也可以是派生类中重写的版本。

__4. 多态的体现__
这种机制允许在不改变代码外部行为的前提下，扩展或修改类的功能。基类可以定义接口（虚函数），而派生类可以通过重写这些虚函数来提供具体的实现。这使得程序可以在运行时动态地调用正确的函数，实现多态。	
[具体可参考]:https://blog.csdn.net/li1914309758/article/details/79916414?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522171869762816800227453122%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=171869762816800227453122&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-79916414-null-null.142^v100^pc_search_result_base8&utm_term=c%2B%2B%E8%99%9A%E5%87%BD%E6%95%B0%E8%A1%A8&spm=1018.2226.3001.4187	
### 纯虚函数：	
```
class Entity {
public:
	virtual std::string getname()=0;
};
class Player :public Entity {
private:
	std::string m_name;
public:
	Player(const std::string& name):m_name(name){}
	std::string getname() override {
		return m_name;
	}
};
```
在类中只定义没有实现的函数叫纯虚函数。带有纯虚函数的类被称之为抽象类只可以作为基类使用，不能对其进行实例化，一般有以下几个功能	
定义接口：抽象类提供了一组接口，派生类必须实现这些接口。
多态行为：通过抽象基类的指针或引用，可以调用派生类中重写的函数，实现多态。
防止直接实例化：抽象类确保用户不能直接创建基类对象，只能通过派生类来使用其功能。
__接口样例__	
```
#include<iostream>
#include<string>
class Printable {
public:
	virtual std::string getname() = 0;
};
class Entity :public Printable {
public:
	std::string getname() override {
		return "entity";
	};
};
class Player :public Entity {
private:
	std::string mname;
public:
	Player(const std::string& name) : mname(name) {}
	std::string getname() override {
		return mname;
	}
};
void print(Printable* p) {
	std::cout << p->getname() << std::endl;
};
int main() {
	Entity* e = new Entity();
	print(e);
	Player* p = new Player("xjy");
	print(p);
}
```
输出为```entity,xjy```	
c++中纯虚函数作用是作为一个接口所有继承于该抽象类的派生类都必须重新实现该函数。此外呢可以发现一个问题就是Entity类中已经重写了Printable类中的getname()并且没有添加virtual关键字，但是Player类中依旧可以重写。这是因为任何由纯虚函数重写衍生来的函数都保持虚函数状态被添加到虚函数表中，直到继承链结束。	
## const 关键字	
__const用于定义常量__
```
int main(){
const int m=100;
int* a=new int;
a=(int*)&m;
*a=2;
std::cout<<*a<<std::endl
} \\输出为2
```
这里有个奇怪的事情就是直接输出max显示的是max的值没有变化，但是*a却发生了变化，这是因为部分编译器会对const常量进行优化，直接嵌入到代码中，而非每次都从内存中读取。导致其直接被绕过。	
```
int main(){
int max=100;
const int* a=new int;或者是 int const* a=new int
*a=2\\会报错；
a=(int*)&max;\\不会报错；
}
```
当const在```*``之前的时候不能对的地址内容做改变但是可以改变地址，当const在```*```之后的时候反过来可以改变内容但是不能改变地址如	
```
int* const a=new int;
```
### 类中const应用	
```
class entity{
private:
int x,y;
public:
int getx() const{
return x;
}
}
```
getx函数是一个只读函数，一般情况下函数内部不能发生数据修改例如在getx()函数中添加一行代码```y=2```这时会报错。但是当```mutable```关键字允许类中const函数存在元素修改操作。例如```mutable int z```这时就可以在get()函数中为z赋值;此外const类型的类只能调用const类型的函数如下	
```
class entity{
private:
int x,y;
public:
int getx() {
return x;
}
}
int main(){
const Entity e;
std::cout<<e.getx()<<std::endl；}
```
## C++成员初始化列表	
```
#include<iostream>
#include<string>
class Example {
public:
	Example() {
		std::cout<<"example created" << std::endl;
	}
	Example(int x){
		std::cout << "example" << x << "created" << std::endl;
	}
};
class Entity {
private:
	std::string name;
	Example exa;
public:
	Entity():exa(Example(8))
 {
	name = "unknow"; }
		
	Entity(const std::string ename) {
		name = ename;
	}
	const std::string getname() {
		return name;
	}
};
int main() {
	Entity e;
	std::cout << e.getname() << std::endl;
	Entity e1("xjy");
	std::cout << e1.getname() << std::endl;
	std::cin.get();
}
```
成员初始化列表在功能上和构造函数内初始化有一点例如将```Entity```构造函数改成```Entity(){exa=Example(8);name="unknow"}```这时实际会创造两个Example实例，一个在参数初始化阶段，一个构造函数阶段。虽说最后只保存了一个实例，但是函数试行过程中首先创造了一个默认example实例销毁他，并且用另外一个实例8覆盖，造成了性能浪费。总之二者差异如下。	
__成员初始化列表：成员变量在进入构造函数体之前就被初始化。只会调用一次构造函数。__	
__构造函数体内赋值：成员变量先被默认构造，然后在构造函数体内重新赋值。会调用多次构造和赋值操作。__	
## 对象创建方式	
### 1在栈上创建	
```
#include<iostream>
class Entity {
private:
	std::string name;
public:
	const std::string& getname() {
		return name;
	}
	Entity(const std::string mname):name(mname){}
};
int main() {
	Entity* e;
	{
		Entity entity("chenro");
		e = &entity;
		std::cout << entity.getname() << std::endl;
	}
std::cin.get();
}
```
### 2在堆上创建	
```
#include<iostream>
class Entity {
private:
	std::string name;
public:
	const std::string& getname() {
		return name;
	}
	Entity(const std::string mname):name(mname){}
};
int main() {
	Entity* e;
	{
		Entity* entity=new Entity("chenro");
		e = entity;
		std::cout << entity.getname() << std::endl;
	}
std::cin.get();
delete entity;
}
```
这里的大括号就是一个栈，在栈道上创建的数据会在栈上内容执行完毕后被销毁，例如在 __1__ 中执行到```std::cin.get()```时将鼠标放在```e```上会显示```name=""```代表 entity被销毁。但是在堆上创建的如果不执行delete则不会被销毁。	
## c++隐式转换和explicit关键字	
```
#include<iostream>
#include<string>
class Entity {
private:
	std::string name;
	int m;
public:
        Entity(const std::string& mname) :name(mname), m(-1) {}
	Entity(int mm) :name("unknow"), m(mm) {}
};
void printff(const Entity& e) {
}
int main() {
	printff(2);
	printff("amns");//报错
	Entity e = 22;
	Entity e1 = std::string("chenro");
}
```
隐式转换只允许一次```printff(2)```可以执行因为构造函数只需要接受2就能创建Entity实例，但是```printff("chenro")```不行因为```chenro```是char字符数组而不是string类型字符串需要先转换成string再创建实例，但是隐式转换只允许一次所以会报错。如果为```Entity(const std::string& mname) :name(mname), m(-1) {}```之前添加explicit关键字那么就相当于禁止隐士转换此时```printff(22)```和```Entity e=22```会报错，```Entity(const std::string& mname) :name(mname), m(-1) {}```同理。
## 运算符重载	
```
#include<iostream>
class Vector {
public:
	int x, y;
public:
	Vector(int xt, int yt) :x(xt), y(yt) {};
	Vector operator + (const Vector& other) {
		return Vector(x + other.x, y + other.y);
	}
	Vector operator * (const Vector& other) {
		return Vector(x * other.x, y * other.y);
	}
};
std::ostream& operator << (std::ostream & stream, const Vector& other) {
	stream << other.x << other.y;
	return stream;
}
int main() {
	Vector V1(1, 1);
	Vector V2(2, 2);
	std::cout << V2 + V1 << V1 * V2 << std::endl;
}```
在这里主要两个问题拿```<<```运算符的重载为例所谓的运算符重载可以视为函数的重写，可以将每一个运算符视为一个函数,``` (std::ostream & stream, const Vector& other)```是他的参数列表，理论上来说呢```+```和```*```运算符在重载的时候应该也需要左右两边两个参数相加，但是由于其在类中被定义为类的成员函数，其拥有```this```指针隐式指向调用对象的引用。可以视为```return Vector(this->x * other.x, this->y * other.y);```.
但是当运算符重载函数被定义为全局函数或者友元函数，那么其由于没有this指针，所以需要将参数完整定义出来例如
```Vector operator + (const Vector& other) {
		return Vector(x + other.x, y + other.y);
	}```如果被定义为友元函数应该改成如下
```
```
class Vector{
\**\
 friend Vector operator + (const Vector V1,const Vector& other) {
		return Vector(V1.x + other.x, V1.y + other.y);
	}
 }
```
### this 指针的用处	
1用于返回对象自身
   ```
   class Vector {
public:
    int x, y;

    Vector(int x, int y) : x(x), y(y) {}

    // 增加成员函数以便进行链式调用
    Vector& setX(int x) {
        this->x = x;
        return *this;
    }}
   ```
2. 解决名称冲突
在构造函数或其他成员函数中，当参数名称与成员变量名称冲突时，可以使用 this 指针来区分成员变量和参数。
```
class Vector {
public:
    int x, y;

    Vector(int x, int y) {
        // 使用 this 指针区分成员变量和参数
        this->x = x;
        this->y = y;
    }
```
3. 在复制构造函数和赋值运算符中
在编写复制构造函数和赋值运算符时，可以使用 this 指针来处理自赋值的情况。
```
class Vector {
public:
    int x, y;

    Vector(int x, int y) : x(x), y(y) {}

    // 拷贝构造函数
    Vector(const Vector& other) : x(other.x), y(other.y) {}

    // 赋值运算符重载
    Vector& operator=(const Vector& other) {
        if (this != &other) { // 检查自赋值
            this->x = other.x;
            this->y = other.y;
        }
        return *this;
    }}
```
## c++对象的生存周期	
```
#include<iostream>
class Entity {
public:
	Entity() {
		std::cout << "Entity Created" << std::endl;
	}
	~Entity() {
		std::cout << "Entity Destoryed" << std::endl;
	}
};
class Solver {
private:
	Entity* ptr;
public:
	Solver(Entity* eptr):ptr(eptr){}
	~Solver(){
		delete ptr;
	}
};
int main() {
	{
		Solver ptr=new Entity();
	}
}
```
Solver是对智能指针实现的一个简单演示，```Solver ptr=new Entity();```首先创建一个Entity实例，该实例在堆上被创建并且返回一个指向该实例的指针，该指针被赋值给Solver实例ptr的变量```Entity* ptr;```这里可能会造成疑惑这里实际上并不是类似于```Entity* e=new Entity()```这样直接将创建的实例指针赋值给e,这里完成的效果实际等价于语句```Solver ptr=Solver(new Entity())```或者```Solver ptr(new Entity())```	
由于这里的ptr在栈上创建也就是其前后两个大括号分割出的作用域，所以在执行到栈作用域结束时ptr生命周期结束自动调用析构函数。这里会有一个问题就是删除的实际是指向Entity实例的指针，而非实例。实际上指针是在栈上的而非在堆上delete关键字删除的不仅仅是指针还有指针指向的内容，因此析构函数执行后不仅删除了指针还删除了ptr指针指向的Entity实例。	
```
int* createarray(){
int array[50];
return array;
}
int main(){
int* a=createarray();
}
```
这里就会出现一个问题由于int* a=createarray();中createarray()在该语句执行完后其创建的数组就被删除了，因此指向该数组的指针也没有意义。解决这一问题可以如下	
```
int* createarray(){
int* array=new int[50];
return array;
}
int main(){
int* a=createarray();
}
```
## 赋值与拷贝构造函数	
```
#include<iostream>
#include<string>
class String {
private:
	char* mptr;
	unsigned int msize;
public:
	String(const char* string) {
		msize = strlen(string);
		mptr = new char[msize + 1];
		memcpy(mptr, string, msize);
		mptr[msize] = 0;
	}
	~String() {
		delete[] mptr;
	}
	friend std::ostream& operator<< (std::ostream& stream, String& string);
	char& operator[](unsigned int index) {
		return mptr[index];
	};
};
std::ostream& operator<< (std::ostream& stream,String& string){
	stream << string.mptr;
	return stream;
}
int main() {
	String a = "xjy";
	String b = a;
	b[2] = 'w';
	std::cout << a << std::endl;
	std::cout << b << std::endl;
	std::cin.get();
}
```
这里会报错这是因为c++中类默认的拷贝构造函数实际上是浅拷贝上述例子中使用到拷贝构造函数实际上是如下	
```
String(String& other){
mptr=other.mptr;
msize=other.msize;
}
```
这会出现一个问题就是由于使用的是浅拷贝，所以b只拷贝了a的mptr指针但是没有拷贝指针指向的字符串，因此b和a的mptr实际上是指向同一块内存区域的指针，在函数执行结束析构函数会在同一块内存区域内执行两次，由于第一次执行该内存区域已经空了，再次尝试会造成运行崩溃。解决这一问题可以进行深拷贝。	
```
String(String& string):msize(string.msize){
mptr=new char[msize+1];
memcpy(mptr,string.mptr,msize+1)
mptr[msize]=0;
}
```
这样b和a进行了一次深拷贝，b重新在堆上分配了一块内存用于存储自己的字符串其mptr指针和a的也不是指向同一块区域。	
## c++动态数组与优化
```
#include<iostream>
#include<vector>
class Vertex {
private:
	int x, y, z;
public:
	Vertex(int ex, int ey, int ez) :x(ex), y(ey), z(ez) {
		std::cout << "vetex created" << std::endl;
	}
	Vertex(const Vertex& vertex):x(vertex.x),y(vertex.y),z(vertex.z) {
		std::cout << "vertex copied" << std::endl;
	}
};
int main() {
	std::vector<Vertex>vedlist;
	vedlist.push_back({ 1, 2, 3 });
	vedlist.push_back({ 4, 5, 6 });
	vedlist.push_back({7, 8, 9});
	std::cin.get();
}
```
运行结果如下	
```
vetex created
vertex copied
vetex created
vertex copied
vertex copied
vetex created
vertex copied
vertex copied
vertex copied
```
1这是因为push_back本质上是在main这个栈上首先创建了实例然后将这个实例拷贝到为变长数组分配的内存中，所以每次create都会跟随一个copied。	
2create后的copied数量逐渐增加这是由于起先并没有为vertex预先分配足够内存空间，默认分配了一个。随着后续实例的添加，变长数组需要重新开辟内存空间这时就需要先拷贝当前内存空间内的实例进行一次copied	
这种拷贝机制会造成大量的性能浪费因此可以采用emplace_back和预分配内存来处理	
```
int main() {
	std::vector<Vertex>vedlist;
        vedlist.reserve(3)
	vedlist.emplace_back( 1, 2, 3 );
	vedlist.emplace_back( 4, 5, 6 );
	vedlist.emplace_back(7, 8, 9);
	std::cin.get();
}
```
``` vedlist.reserve(3)```为变长数组预先分配了足够的内存空间让其避免因为当前内存空间不够进行拷贝操作，```emplace_back```直接在vedlist中创建实例，避免了从main栈中拷贝。
## c++模板	
```
#include<iostream>
#include<string>
template<typename T>
void Print(T name) {
	std::cout << name << std::endl;
}

template<typename T,int M>
class Array {
private:
	T array[M];
public:
	int getsize() const {
		return M;
	}
	Array(const T* string) {
		for (int i = 0; i < M; i++) {
			array[i] = string[i];
		}
	}
	void Printmessage() {
		for (int i = 0; i < M; i++) {
			std::cout << array[i];
		}
	}
};
int main() {
	Print(1);
	Print(1.1);
	Print("Helloworld");
	char b[5] = {'a','b','c','d','\0'};
	Array<char, 5>a(b);
	a.Printmessage();
}
```
## 堆与栈内存的比较	
堆上分配内存需要在空闲列表上寻找足够大小内存块，记录空闲内存有哪些内存被使用又有哪些被拿走了，在使用完后又要delete再次维护空闲列表。
## 函数指针	
```
#include<iostream>
#include<vector>
void helloworld() {
	std::cout << "helloworld" << std::endl;
}
void Print(int value) {
	std::cout << value << ' ';
}
void Printarray(std::vector<int>& values,void(*func)(int)){
	for (int i = 0; i < values.size(); i++) {
		func(values[i]);
}
}
int main() {
	typedef void(*helloworldfunction)();
	auto Helloworld = helloworld;
	helloworldfunction wpe = helloworld;
	wpe();
	Helloworld();
	std::vector<int>array = { 1,2,3,4,5 };
        Printarray(array, [](int value) {std::cout << value << ' '; });
	Printarray(array, Print);
	std::cin.get();
}
```
```void(*func)(int)```形式有点类似于函数签名，区别是在函数名前面添加*。返回的实际上是一个指向无返回值且传参为int的函数的指针。通过（）可以调用该指针指向的函数。```Printarray(array, [](int value) {std::cout << value << ' '; });```是lamba表达式的使用，这里的和```Printarray(array, Print);```的区别之处就是将指向全局函数```Print```的函数指针改编成了指向```[](int value) {std::cout << value << ' '; }```这一只在```Printarray```被调用时生成的临时函数。	
## 命名空间	
```
#include<iostream>
#include<string>
#include<algorithm>
namespace apple{
	void print(const char* message) {
		std::cout << message << std::endl;
	}
}
namespace orange {
	void print(std::string message) {
		std::reverse(message.begin(), message.end());
		std::cout << message << std::endl;
	}
}
int main() {
	using namespace apple;
	using namespace orange;
	const char* message = "hello";
	print(message);
}
```
这里的输出应该是```hello```因为print并没有指定哪个命名空间所以采用就近原则这里的就近原则指的是是否存在隐式转换，apple::print的传参类型是```const char*```和message类型一致所以调用的是apple::print此外这里要注意的是隐式转换并不是双向的const char* 到std::string存在隐式转换但是反过来就不行例如我将```using namespace apple```注释掉此时没问题const* char可以隐式转换为std::string。但是我若将```using namespace orange```注释掉同时messgae类型为std::string这时就会在print（message)那报错。	
## 线程	
```
#include<iostream>
#include<thread>
static bool is_thread_finished = false;
void print() {
	using namespace std::chrono;
	while (!is_thread_finished) {
		std::cout << "message" << std::endl;
		std::this_thread::sleep_for(1s);
	}
}
int main() {
	std::thread worker(print);
	std::cin.get();
	is_thread_finished = true;
	worker.join();
	std::cin.get();

}
```
这里创建了一个名为worker的线程，传入的参数是函数签名。在没有worker。join之前worker和主线程并发执行	
## c++中的sort	
```
#include<iostream>
#include<vector>
#include<algorithm>
int main(){
std::vector<int>array={1,2,3,4,5};
std::sort(array.begin(),std::end(),std::greater<int>());
}
```
这里有一个困惑的地方就是```std::greater<int>()```是个什么东西，很明显std::greater<int>是一个模板类定义因此呢这句话实际上是用这个模板类定义了一个函数对象。这也是一个很让人疑惑的地方，类怎么能定义一个函数对象。这个函数对象虽然是一个类实例但是却可以像函数一样调用这是个很奇怪的事情。这里实际上涉及到仿函数。不妨看看std::greater<int>这一模板类的底层实现	
```
namespace std {
    template <typename T>
    struct greater {
        bool operator()(const T& lhs, const T& rhs) const {
            return lhs > rhs;
        }
    };
}
```
这里呢重载了std命名空间中greater类的()操作符使其可以作为一个仿函数被调用。__注意这里```std::greater<int>()```是一个整体，是一个实例__，__std::greater<int>不是数据类型，“（）”不是由```std::greater<int>```创建出的实例！！！！！！！！！！！！！！！！__ ```std::greater<int>()```是c++中除了在栈上创建例如```greater g```和在堆上创建例如```greater* g=new greater()```这两种方法外另外一种比较特殊的对象创建方式，被称之为 __临时对象__ 和lamba表达式有点相似都是在使用时才会被创建使用完立马消失。	
那么现在就明了了std::greater<int>()是一个仿函数实例，接受来自迭代器的传参```return bool=greater(lhs,rhs)```这里的自动调用实际上就涉及到一堆东西了比如
编译器解析：当编译器遇到 greater(2, 3) 时，它识别出这是一个函数调用语法。
成员函数查找：编译器会检查 greater 对象所属的类（即 Adder）是否定义了 operator() 运算符。
运算符调用：如果找到，编译器会生成代码来调用这个 operator() 成员函数，就像普通的成员函数调用一样。	
这里要注意的是这个operator()是所有仿函数类中必须被重载的，这里就涉及到c++标准了，具体得去看文档。
## 类型双关		
```
#include <iostream>

int main() {
    int a = 50;
    double b = *(double*)&a;
    std::cout << b << std::endl;
    return 0;
}
```
这里的核心代码就在``` double b = *(double*)&a;```这里首先将a的地址转换为double类型的指针然后再利用这个指针对其所指向的地址进行解包。	
可能有人会疑问唉这个指针不是还是指向a的地址吗，确实。但是这里需要分清楚指针和地址实际上是完全不同的两个概念，在我理解的指针概念中指针是用来读取其指向的内存地址的一个方式。不同数据类型的指针其差距其实是其读取内存内容方式上的差距。例如```int*``` 类型的指针如果对其进行解包他会从其指向的内存地址中读取四个字节的数据，```double*```类型的呢则会读取8个字节的数据。这就可以解释为什么输出的b会是一个和a毫不相关的数字。	
可以将a的内存地址表示出来	
__假设int a=50__ 的内存布局如下
```
| 地址     | 内容（十六进制表示） |
|--------|----------------|
| 0x1000 | 0x32           |
| 0x1001 | 0x00           |
| 0x1002 | 0x00           |
| 0x1003 | 0x00           |
```
当你执行``` (double*)&a``` 时，你是在将 ```a``` 的地址```（0x1000）```解释为指向 ```double``` 类型的指针。然后解引用这个指针意味着从 ```0x1000``` 地址开始读取 ```double``` 类型的数据。```double``` 通常占用 8 个字节，因此系统将尝试读取从 ```0x1000```开始的 8 个字节的数据，并将其解释为一个 ```double``` 值。但是呢由于我们一开始定义的```a```的数据类型是```int```并且在输出b之前也没有对a进行强制类型转换，因此系统只会为分配4个字节的内存空间，但是double*类型的指针要读取8个字节啊，然后就出问题了，该指针指向的内存地址只有四个字节是分配给a的，后面的四个字节完全就是没有被定义的随机16进制数，但是由于指针是double的他们也会被读取进去，经过解码成b以后就造成了输出的b是一个奇奇怪怪的东西。	
读取的内容如下		
```
| 地址     | 内容（十六进制表示） |
|--------|----------------|
| 0x1000 | 0x32           |
| 0x1001 | 0x00           |
| 0x1002 | 0x00           |
| 0x1003 | 0x00           |
| 0x1004 | 随机数据       |
| 0x1005 | 随机数据       |
| 0x1006 | 随机数据       |
| 0x1007 | 随机数据       |
```
然后呢如果在这时候我们将b的类型改变成```double& b```并且```b=0.0```这时候程序会崩溃。这是因为改成引用类型后b写入就代表a写入0.0是一个八字节的浮点数，但是由于a只占用了四个字节，后续的四个字节的内存区域是未被定义的。往未被定义的内存区域中写入内容造成了程序崩溃。
## 联合体	
```
int main() {
	struct Union {
		union {
			float a;
			int b;
		};
	};
	Union u;
	u.a = 2.0f;
	float c = 2.0f;
	int d = *(int*)&c;
	std::cout << d << std::endl;
	std::cout << u.a << ' ' << u.b << std::endl;
}
```
联合体其作用是用不同的解读方式解释同一块区域的内存中的内容定义多种不同的访问方式这里的d输出和u.b相同都是2的30次方对应的就是2.0的IEEE754表示的二进制码```0100 0000 0000 0000 0000 0000 0000 0000```转换为十进制整数的形式。具体的计算方式得查询计算机组织体系相关书籍。	
这里给出一个计算过程吧	
IEEE 754 浮点数表示
一个 32 位（4 字节）的 float 类型数值在 IEEE 754 标准下的表示形式如下：
```
SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM
S：符号位，占 1 位。
E：指数位，占 8 位。
M：尾数，占 23 位（隐含一个首位 1，即 24 位）。
```
```
2.0 的浮点数表示
2.0 的二进制表示是 10.0。
在标准化形式下，2.0 表示为 1.0 * 2^1。
因此：
符号位 S = 0（表示正数）。
指数 E = 1（真实指数） + 127（偏置）= 128，即 10000000。
尾数 M = 00000000000000000000000（去掉隐含的首位 1）。
因此，2.0 的 IEEE 754 表示为：
0 10000000 00000000000000000000000
```
将0 10000000 00000000000000000000000转变成10进制就变成2^30了；
```
#include<iostream>
struct vector2 {
	float i, j;
};
struct vector4 {
	//float a, b, c, d;
	/*vector2& getaA() {
		return *(vector2*)&a;
	}*/
	union {
		struct {
			float x, y, z, w;
		};
		struct {
			vector2 m, n;
		};
	};
};
void printvector2(const vector2& v){
	std::cout << v.i << ' ' << v.j << std::endl;
}
int main() {
	vector4 v = { 1.0f,2.0f,3.0f,4.0f };
	printvector2(v.m);
	printvector2(v.n);
}
```
这是联合体在类中的应用
## 虚析构函数	
```
#include<iostream>
class Base {
public:
	Base() {
		std::cout << "Base project constructed" << std::endl;
	}
	~Base() {
		std::cout << "Base Project destructed"<<std::endl;
	}
};
class Derived :public Base {
private:
	int* marray;
public:
	Derived() {
		marray = new int[5];
		std::cout << "Derived project constructed"<<std::endl;
	}
	~Derived() {
		delete[] marray;
		std::cout << "Derived project destructed" << std::endl;
	}
};
int main() {
	Base* b = new Base();
	delete b;
	std::cout << "-------------" << std::endl;
	Derived* d = new Derived();
	delete d;
	std::cout << "---------------" << std::endl;
	Base* ba = new Derived();
	delete ba;
}
```
输出为	
```
Base project constructed
Base Project destructed
-------------
Base project constructed
Derived project constructed
Derived project destructed
Base Project destructed
---------------
Base project constructed
Derived project constructed
Base Project destructed
```
可以看出当我们将基类指针```ba```指向派生类对象的时候，当调用析构函数时并没有调用派生类的析构函数而只调用了基类的，这样如果在派生类的构造函数中有在堆上创建变量的操作由于无法释放内存很可能会造成内存泄漏的问题。为了解决这一问题为基类的析构函数添加virtual关键字表明可能还有其它析构函数。	
这里要注意一个问题 __虚析构函数和普通成员函数不同的是，普通成员函数通过维护一个虚函数表实现基于基类成员函数的重写和覆盖。但是析构函数不同，析构函数是在基类析构函数的基础上添加派生类的析构函数，其本身并不会被覆盖__。
```
virtual ~Base() {
	std::cout << "Base Project destructed"<<std::endl;
}
```
## dynamic_cast	
```
#include<iostream>
#include<string>
class Entity {
public:
	virtual void get() {};
};
class Player :public Entity {

};
class Enemy :public Entity {

};
int main() {
	Player* player = new Player();
	Entity* e = new Entity();
	Entity* e1 = new Enemy();
	Entity* e2 = new Player();
	Player* p = dynamic_cast<Player*>(e2);
	if (p) {
		std::cout << "convert succeeded" << std::endl;
	}
	else {
		std::cout << "can't convert" << std::endl;
	}
}
```
dynamic_cast时运行时安全的转移多态类型，主要用于在继承关系中进行向下转换（downcasting），即从基类指针或引用转换到派生类指针或引用。和static_cast相比他多了运行时类型检查即RTTI。防止了在强制类型转换中可能会出现的未定义行为，例如将一个指向基类或者其余派生类的基类指针强制转换为当前类指针。说到多态这里还有一点想法补充，也是在学习这一章时的浮现出的一个问题	
c++中多态的实现实际上是靠基类维护一个虚函数表，这个虚函数表使得基类可以通过其基类指针调用派生类中独有的函数，但是呢很多情况下派生类中不仅包含重写于基类虚函数的函数还包含其独有函数，这种情况下通过基类指针无法调用派生类的这些独有函数。除了上述最简单的dynamic_cast还可以将所有可能的实体类中出现的函数都添加到基类的虚函数表中，但是这样的结果就是基类太过于冗余，在函数继承中会造成大量的性能浪费。有一种特殊的方法是采用访问者模式具体实现如下	
```
#include<iostream>
class Base;
class Derived;
class Visitor {
public:
	virtual void visit(Base* base) = 0;
	virtual void visit(Derived* derive) = 0;
};
class Base {
public:
	~Base() = default;
	virtual void accept(Visitor* visitor) = 0;
};
class Derived:public Base{
public:
	void accept(Visitor* visitor) override {
		visitor->visit(this);
	}
	void derivedFunction() {
		std::cout << "Derived specific function" << std::endl;
	}
};
class Derived_visitor:public Visitor {
public:
	void visit(Base* base) override {
		std::cout << "Visiting Base" << std::endl;
	}
	void visit(Derived* derive)override {
		std::cout << "Visiting Derive" << std::endl;
		derive->derivedFunction();
	}
};
int main() {
	std::unique_ptr<Base>base = std::make_unique<Derived>();
	Visitor* visit = new Derived_visitor();
	base->accept(visit);
}
```
通过使用访问者模式可以将对象的操作分离到独立的访问对象之中，如果要实现通过基类指针调用派生类独有函数的操作只需要在Visitor中定义visit纯虚函数，在Derived_visitor中重写visit同时在Derived类中添加visit操作。本质上就是在基类和派生类之间找了个中间类，让基类和派生类的独有函数之间架起了一个指针桥梁，基类通过虚函数表指向visit类，再通过visit类的指针再通过虚函数表中的accept函数调用派生类的独有函数。
## 共享指针share_ptr
基本使用
```
class MyClass {
public:
    MyClass() {std::cout << "MyClass Constructor" << std::endl;}
    ~MyClass() {std::cout << "MyClass Destructor" << std::endl;}
    void display() const {
        std::cout << "MyClass display" << std::endl;
    }
}
int main() {
    {
        std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
        {
            std::shared_ptr<MyClass> ptr2 = ptr1; // ptr2 共享 ptr1 管理的对象
            ptr2->display();
            std::cout << "ptr2 use count: " << ptr2.use_count() << std::endl; // 引用计数
        }
        std::cout << "ptr1 use count: " << ptr1.use_count() << std::endl; // 引用计数
    } // 超出作用域，引用计数为0，对象被销毁
    return 0;
```
自定义删除器	shared_ptr 可以使用自定义删除器，在对象销毁时执行特定操作。
```
void customDeleter(int* p) {
    std::cout << "Custom Deleter called for " << *p << std::endl;
    delete p;
}
int main() {
    std::shared_ptr<int> ptr(new int(10), customDeleter);
    return 0;
}
```
循环引用	
```
class Node {
public:
    std::shared_ptr<Node> next;
    ~Node() {
        std::cout << "Node Destructor" << std::endl;
    }
};

int main() {
    {
        std::shared_ptr<Node> node1 = std::make_shared<Node>();
        std::shared_ptr<Node> node2 = std::make_shared<Node>();
        node1->next = node2;
        node2->next = node1; // 这将导致循环引用，两个对象都不会被销毁
    } // node1 和 node2 超出作用域，但对象不会被销毁，因为存在循环引用
```
## std::any std::variant	
```
#include<iostream>
#include<any>
#include<variant>
int main() {
	std::any b;
        b = "asdjaks";
	std::string s = std::any_cast<std::string>(b);
	//const char* s = std::any_cast<const char*>(b);
	std::cout << s << std::endl;
}
```
std::any是类型不安全的，例如上述代码中的b实际上是const char*类型的但是由于程序员的失误有可能原本目的是创建一个std::string类型的，但是由于std::any是在运行时检查实际存储的类型是否和请求的类型相匹配，这就可能导致程序崩溃
但是 std::variant相比之下就安全许多，它要求在定义阶段就限制变量数据类型，避免了因为失误造成的崩溃	
```
int main() {
	std::variant<std::string, int>d;
	d = "asdasdasd";
	std::string ds = std::get<std::string>(d);
	std::cout << ds << std::endl;
}
```
这时d虽然看似是const char*类型但是由于在定义初期就将d的数据类型范围限制死了所以d的数据类型会被自动匹配为std::string实现了类型安全。	
## 字符串优化	
```std::string```虽然没有显示的在堆上分配，但是在```std::string```类中存在堆上分配内存的操作可以手动实现一个简单的c++```std::string```类如下	
```
class string{
private:
   const char* data;
public
  string（const char* String）{
  string_size=strlen(String);
  data=new char[stringsize+1];
  std::strcpy(data,s);
}
  ~string(){
delete[] data;
}
}
```
可以看到std::string在进行赋值构造时会进行堆分配操作然而堆分配操作相比于在栈上分配内存需要维护空闲表这是一个耗时的操作，因此为了堆字符串处理速度进行优化，我们可以使用指针的形式c++17中提供了如下工具	
```
static uint32_t allocatecount=0;
void* operator new(size_t size) {
	allocatecount++;
	std::cout << "allocatecount: " <<size<< std::endl;
	return malloc(size);
}
void Printname(const std::string_view& name) {
	std::cout << name << std::endl;
}
int main() {
	std::string name = "asdadalsp";
#if 0
	std::string firstname = name.substr(0,3);
	std::string lastname = name.substr(3, size(name)-2);
#else
	std::string_view firstname(name.c_str(),3);
	std::string_view lastname(name.c_str()+3,6)；
#endif
	Printname(lastname);
	std::cout << allocatecount << std::endl;
}
```	

