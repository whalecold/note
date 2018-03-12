# Inside The C++ Object Model  读书笔记

## 第一章 关于对象

> ### c++在布局相对c的额外负担由virtual引起
> + `virtual function` 机制：
> + `virtual base class`： 用以实现 "多次出现在继承体系中的base class, 有一个单一而被共享的实体"。
> + 多重继承下的额外负担

### C++ 对象模式

>+ class 两种data members : `static nonstatic`
>+ class 三种functions member : `static nonstatic virtual`

#### 三种模型
>##### 简单对象模型
> + `object`是一系列的是`slots` 每个`slot`指向一个`member`
>##### 表格驱动对象模型
>##### c++对象模型
>
> ```
> ADT:抽象数据类型
> ```

#### 加上继承

#### 对象的差异
>```
> paradigm：典范
>```
> + 程序模型`procedural model`
> + 抽象数据类型模型 `abstract data type model, ADT`
> + 面对对象模型 `object-oriented model`
> ##### object大小
> + nonstatic data 总和大小
> + 加上字节对齐的所需要的额外空间
> + 加上为了支持virtual而由内部产生的额外负担

## 第二章 构造函数语意学
>```
>
>implicit    : 隐含
>explicit    : 明确的 (禁止隐式转换)
>trivial     : 没有用的
>nontrivial  : 有用的
>memberwise  : 对每一个member实以...
>bitwise     : 对每一个bit施以...
>semantics   : 语意
>```
>+ [inline](https://www.cnblogs.com/fnlingnzb-learner/p/6423917.html)      : 内联 (定义在类中的成员函数缺省都是内联的，如果在类定义时就在类内给出函数定义，那当然最好。如果在类中未给出成员函数定义，而又想内联该函数的话，那在类外要加上inline，否则就认为不是内联的。**关键字inline 必须与函数定义体放在一起才能使函数成为内联**, 仅将inline 放在函数声明前面不起任何作用)

### 默认构造函数
#### 四种情况下如果没有默认构造函数编译器会构建默认构造函数
+ `class member` has default constructor  
+ `base class` has default constructor  
>+ `virtual function` 和**vptr**有关?
>+ `base class` has `virtual function` 和**vptr**有关?
+ **其他情况下如果没有default constructor也不会被编译器合成出来**

### Copy Constructor 拷贝构造函数
#### 三种情况会调用 Copy Constructor
>+ operator `=`
>+ 传参的时候
>+ 函数返回值

#### 在`bitwise copy semantics`下不会在没有Copy Constructor的情况下构造
#### 编译器构建 Default Copy Constructor 的情型与上面一致



### 程序转化语意`program transformation semantics`

+ 定义是指占用内存的行为
+ **nrv**(name return value)优化 下面有个例子
```
  class test {
      friend  test foo(double);
  public:
  	test() {
  		memset(array, 0, sizeof(array));
  	}
    void test(const test &);
  private:
  	double array[100];
  }
  
  //增加copy constructor 效率会高不少
  inline void test::test(const test &t) {
      memcopy(this->array, t.array, sizeof(this->array));
  }
  
  test foo(double val) {
  	test local;
  	local.array[0] = val;
  	local.array[99] = val;
  	
  	return local;
  }
  
  int main() {
  	for (int count = 0; count < 10000; count ++) {
  		test t = foo(double(32));
  	}
  	return 0;
  }
```
+ 上面例子自己试了下好像没什么效果 - -！

### 成员的初始化队伍
```
    class test {
    public:
        int x;
        int y;
        int z;
    public:
        test(int yy):z(yy), y(yy) {
            x = yy;
        }
    };
```
+ 上述初始化的顺序是 y - > x -> x （ *initialization list 中的初始化顺序是按照类内的申明顺序 但是他的顺序会排在 explicit user code 之前）* 

## 第三章 Data语意学

### Data Member的绑定
### Data Member的布局
### Data Member的存取

#### Static Data Members
#### Nonstatic Data Members
>```
>   Point3d origin, *pt = &origin;
>   origin.x = 0.0;
>   pt->x = 0.0;
>   这两个在一下情况下是不一样的:
>```
+ 当*Point3d*的继承的基类含有虚基类时，并且存取的成员*x* 是从虚基类继承而来的成员，就会有重大的差异。因为这个时候*pt*指向哪一个类型是不确定的，这个操作就必须等到执行期经过间接引导才能够确定。但是如果是origin的话会在编译期间就确定了偏移值。**所以指针调用的效率会低一点**。其它情况下是一致的。


### 继承与Data Member
#### 没有多态的继承
+ 继承之间会进行`alignmengt`
```
eg1:    class class1 {
        private:
            int i_;
            char j_;
        }
        sizeof(class1) = 8
        
        class class2 : public class1 {
        private:
            char z_;
        }
        sizeof(class2) = 12
    
eg2:    class class3 {
        private:
            int i_;
            char j_;
            char z_;
        }
        sizeof(class3) = 8
原因:   在派生类的基类之间赋值会导致数据区错误
```

#### 多态的继承 增加空间负担 
+ virtual table
+ virtual ptr
+ constructor
+ destructor

### 对象成员的效率(Object Member Efficiency)
+ **TODO** 关于`cc` 和 `ncc` 效率问题
```
程序员如果关心效率 必须要进行实际测试 可以推论 但仅当参考
```
+ 虚继承慢的原因是因为有些需要跑到执行期间进行某些操作才会找到正确的路径 而没有虚函数的操作在编译期间就就确定了(这句话不是很正确 但大概就是那么个意思，其实不用指针的话也不会产生*多态* 也就还是在编译期间就确定了)


### 指向Data Member 的指针 (Point to Data Members)
```
Point3d::x 表示 Point3d中的偏移地址(会加 1? 这个不同编译器不一样吧 书上的vs c++ 会 我在centos上测试了不会 所以还是要实际测试啊)
float Point3d::*p2 = &Point3d::x;
Point3d::* 的意思是："指向 Point3d data member" 的指针类型
```

+ 指向Member的指针的效率问题
```
float Point3d::*p2 = &Point3d::x;
Point3d ob;
ob.*p2 === ob.x (相当于把指向data memeber的指针重新绑定到class object上去 但是这样效率非常低)
原来c++还有这么多奇奇怪怪的东西
```
+ 类似的继承类不会影响效率 但是会影响优化效率

## 第四章 Function语意学