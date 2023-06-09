#  1. 再探构造函数

##  1.1 初始化列表

我们习惯在定义变量的同时将它初始化：

```cpp
int a = 1;
```

但初始化和赋值是有区别的，赋值是初始化的一部分。

而被const修饰的变量或对象表明它不被修改的属性，言外之意const修饰的变量或对象必须在定义时被初始化，这是它唯一的值。

构造函数的

### 1.1.1 格式

以冒号`:`开始，以逗号`,`分隔。每个变量后都有一个括号，括号中的是初始值或表达式。

```cpp
class Date
{
	Date(int year, int month, int day)
    :_year(year)
    ,_month(month)
    ,_day(day)
    {}
private:
    int _year;
		int _month;
		int _day; 
};
```

###   1.1.2 特点

当类中有以下三种成员，必须将它们放在初始化列表中初始化：

- 引用；
- const成员变量；
- 没有默认构造函数的自定义类型成员。

###  1.1.3 用途

试看以下代码：

```cpp
class Time
{
  //...
private:
	int hour;  
};

class Date
{
	Date(int year, int hour)
  {
    //函数体内初始化：
    _year = year;
    Time t(hour);
    _t = t;
  }
private:
    int _year;
		int _month;
		int _day; 
  	int _t;
};
```

这样初始化成员是在函数体内赋值，但是会出问题：没有默认构造函数。原因是没有提供自定义类型的默认构造，初始化列表就能解决像这样的问题。

###  1.1.4 小结

C++有一个“不公平”的机制，只处理自定义类型，内置类型不处理，而C++11以后的补丁可以在类成员声明时给出缺省值，内置类型的缺省值在没给构造函数的情况下会被初始化列表使用。

在很多类中，初始化和赋值的区别会在效率上体现：前者直接初始化数据成员，后者先初始化再赋值。除了效率之外的更重要的是，一些诸如被const修饰的变量必须被初始化，所以不论如何，都建议为变量初始化，以避免编译错误，同时也能避免许多低级错误，如野指针问题。

内置类型用哪种方法都行，主要是那三种情况，索性都用初始化列表。能用初始化列表就用，但是有时候不能一棍子打死，有时还是放在函数初始化比较好。例如用初始化列表开辟一块内存空间，这个类的初始化工作放在初始化列表就略显繁琐。在函数体内就较为合理。

题：问初始化顺序：

```cpp
class A
{
  //...
public:
  A(int a)
    :_a1(a)
    ,_a2(_a1)
    {}
private:
 	int _a2;
  int _a1;
};

```

初始化顺序：a2、a1

原因之前有学习过：按照声明次序初始化，与初始化列表中的顺序无关。

##  1.2 关键字explicit

构造函数可以构造和初始化对象，也有类型转换的作用。

先看代码：

```cpp

class Date
{
  //...
private:
    int _year;
		int _month;
		int _day; 
};
int main()
{
  Date d1(2022);
  Date d2 = 2022;
  return 0;
}
```

main函数中的第一行是直接调用构造函数构造对象d1，而d2发生了隐式类型转换：本来要构造一个对象d2，就必须用2022构造出另外一个临时对象出来，然后再用这个临时对象调用拷贝构造函数构造d2。但是编译器优化：直接用2022构造。

构造函数的类型转换作用是针对于**单个参数**或**除第一个参数无默认值其余均有默认值**的构造函数，加上explicit关键字就禁止类型转换。

**单参数的构造函数：**

```cpp
class Date
{
public:
    explicit Date(int year)
        :_year(year)
{}
```

**除第一个参数无默认值其余均有默认值**的构造函数：

```cpp
explicit Date(int year, int month = 1, int day = 1)
  	: _year(year)
  	, _month(month)
    , _day(day)
{}
```

`Date d2 = 2022`在讲述构造d2的过程时，常常将赋值操作符左侧的2022称为匿名对象。

> 匿名对象：作用域只有一行，但是它也会直接调用析构函数。
>
> 意义：如果只是一次性使用对象中的函数，实例化一个对象的成本有些大，使用匿名对象一行代码更简洁。

#  2. static成员

被static修饰的类成员称为静态成员，有静态成员变量和静态成员函数。

**规则：**静态成员变量必须在类外部初始化。

```cpp
class A
{
  //...
private:
  static int _val;
};

int A :: _val = 0;
```

##  2.1 静态成员变量

静态成员变量属于类的所有对象，它存储在静态区，所有对象共用一份。通过对象名调用这个静态成员，其实并不是通过对象找到静态成员，而是找到属于静态成员的**类域**。也可以直接指明类域调用，就像在类外部初始化静态成员变量一样。

一般不建议使用全局变量，因为它可能会被任意修改。将它改成静态成员变量也能达到相同的效果，也即它的公有性。但是需要另外写Get接口访问静态成员变量。

##  2.2 静态成员函数

可以访问私有静态成员变量，但没有this指针，且只能访问静态成员变量。

## 2.3 特点

- 静态成员为所有类对象所共享，不属于某个具体的对象，存放在静态区；
- 静态成员变量必须在类外定义，定义时不添加static关键字，类中只是声明；
- 类静态成员即可用 `类名::静态成员` 或者 `对象.静态成员` 来访问；
- 静态成员函数没有隐藏的this指针，不能访问任何非静态成员；
- 静态成员也是类的成员，受public、protected、private 访问限定符的限制。

##  2.4 练习

【题】实现一个类，计算程序中创建出了多少个类对象。

```cpp
#include <iostream>
using namespace std;
class A
{
public:
    //构造
    A(const A& t)
    {
        ++_scount;
    }
    A()
    {
        ++_scount;
    }
 
  	//静态成员函数
    static int GetACount()
    {
        return _scount;
    }
  	//析构函数
    ~A()
    {
        --_scount;
    }
private:
  	//静态成员变量
    static int _scount;
};
//在类的外部初始化静态成员变量
int A::_scount = 0;
int main()
{
    cout << A::GetACount() << endl;
    A a1, a2;
    A a3(a1);
    cout << A::GetACount() << endl;
    return 0;
}
```

利用静态成员，静态成员变量设为私有，当做计数器，对象创建的标志之一就是调用构造函数，但还不够，销毁对象会调用析构函数。将静态成员函数作为Get接口。

【问】静态成员函数可以调用非静态成员函数吗？

【答】不能。因为静态成员函数没有this指针，但是可以访问私有静态成员变量。

【问】非静态成员函数可以调用类的静态成员函数吗？

【答】可以。原因相同，因为有this指针。

【题】设计一个只能在栈上定义对象的类

```cpp
class StackOnly
{
public:
	static StackOnly CreateObj()
	{
		StackOnly so;
		return so;
	}

private:
	StackOnly(int x = 0, int y = 0)
		:_x(x)
		, _y(0)
	{}
private:
	int _x = 0;
	int _y = 0;
};

int main()
{
	//StackOnly so1; // 栈
	//static StackOnly so2; // 静态区
	StackOnly so3 = StackOnly::CreateObj();

	return 0;
}

```

#  3. 友元

私有成员只能在类的成员函数内部访问，如果想在别处访问对象的私有成员，只能通过类提供的接口（成员函数）间接地进行。这固然能够带来数据隐藏的好处，利于将来程序的扩充，但也会增加程序书写的麻烦。

友元分为友元函数和友元类。

> 友元虽然为访问私有成员变量提供了便利，但是它破坏了封装的安全性，增加耦合度，所以需合理使用友元。

##  3.1 友元函数

###  3.1.1 引例

假设重载流插入运算符`<<`为成员函数：operator<<。然而这没有友元函数是做不到的。

> `<<`和`>>`在C语言中是作为左移运算符和右移运算符使用的，在C++中常常搭配cout用在输入输出语句。实际上它们并没有这样的功能，只是被重载了。
>
> cout是ostream对象，ostream类将`<<`多次重载为成员函数，这才能使得C++的输出语句看起来十分“智能”，对于这些它们，只不过是枚举了所有情况罢了，就像围棋大师“Alpha Go”。

但是cout的输出流对象和this指针的位置都在最左侧，这发生了冲突。将operator<<重载为全局函数可以解决位置冲突，但是不能访问到类的私有成员。友元函数则能将它既作为类外函数又能访问私有成员。

###  3.1.2 概念

友元函数可以允许其他类或函数访问另一个类的非公有成员，增加一条以friend关键字开头的函数声明语句即可。

它是定义在类外部的普通函数，不属于任何类，但需要用friend在类中声明。也就是说，友元声明只能出现在类定义的内部，但是在类内出现的具体位置不限。友元不是类的成员也不受它所在区域访问控制级别的约束。

```cpp
class Date
{
	//在类的内部声明友元函数
	friend ostream& operator<<(ostream& _cout, const Date& d);
	friend istream& operator>>(istream& _cin, const Date& d);

public:
	Date(int year = 2002, int month = 2, int day = 1)
		: _year(year)
		, _month(month)
		, _day(day)

	{}
private:

	int _year;
	int _month;
	int _day;
};
//在类外部定义友元函数
ostream& operator<<(ostream& _cout, const Date& d)
{
	_cout << d._year << "/" << d._month << "/" << d._day << endl;
	return _cout;
}
istream& operator>>(istream& _cin, const Date& d)
{
	_cin >> d._year;
	_cin >> d._month;
	_cin >> d._day;
	return _cin;
}
```

###  3.1.3 特点

- 友元函数可以访问类的私有和保护成员，但不是类的成员函数；
- 友元函数不能被const修饰；
- 一个函数可以是多个类的友元函数；
- 友元函数可以在类定义的任何地方声明，不受类访问限定符限制；
- 友元函数的调用过程和普通函数无异。

##  3.2 友元类

###  3.2.1 概念

一个类 A 可以将另一个类 B 声明为自己的友元，类 B 的所有成员函数就都可以访问类 A 对象的私有成员。

```cpp
class A
{
	//将B作为A的友元类
	friend class B;

	//...

private:
	int _a;
};

class B
{
	//...
	B(A a)
	{
		_aa = a._a;//这个赋值需要重载
	}
private:
	int _b;
	A _aa;
};
```

其中：_a是A类的私有成员，将B作为A的友元类后，在B类中可以访问它。

###  3.2.2 特点

- 友元关系在类之间不能传递，即类A是类B的友元，类B是类C的友元，并不能推出类A是类C的友元。“朋友的朋友不一定是朋友”。
- 友元关系是单向的；
- 友元关系不能继承。

#  4. 内部类

##  4.1 概念

如果一个类定义在另一个类的内部，这个内部类就叫做内部类。内部类是一个独立的类，它不属于外部类，更不能通过外部类的对象去访问内部类的成员。

外部类对内部类没有任何优越的访问权限。**内部类就是外部类的友元类**，内部类可以通过外部类的对象参数来访问外部类中的所有成员。但是**外部类不是内部类的友元**

```cpp
class A
{
public:
	class B
	{
		//...
	public:
		void func()
		{
			cout << "B是A的内部类" << endl;
			A a;
			cout << "A的私有变量_a:" << a._a << endl;
		}
	};
private:
	int _a = 1;
};

int main()
{
	A::B b;
	b.func();
	return 0;
}
```

# 5. 练习

### 5.1 [**求1+2+3+...+n**](https://www.nowcoder.com/practice/7a0da8fc483247ff8800059e12d7caf1?tpId=13&tqId=11200&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

![image-20220824212921608](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220824212921608.png)

思路：按照题目的要求，用C是很难完成的，而刚学习的内部类和静态成员变量能巧妙地解决它。

首先是计数器：由于静态成员变量是属于类而不属于任何对象的，相当于一个全局变量。

其次是n是题目传入的对象个数，将Sum作为内部类，将计数器包含在Sum构造函数中，这样不论是直接构造还是拷贝构造都会使计数器改变，而在上一层类中的Sum_Solution函数中，用内部类Sum定义一个长度为n的数组，每个元素的类型都是Sum。

```cpp
class Solution {
public:
    class Sum
    {
      public:
        Sum()
        {
            _sum += _i;
            _i++;
        }
    };
    int Sum_Solution(int n) {
        Sum a[n];
        return _sum;
    }
private:
    static int _i;
    static int _sum;
};
int Solution::_i = 1;
int Solution::_sum = 0;
```

##  5.2 [****计算日期到天数转换****](https://www.nowcoder.com/practice/769d45d455fe40b385ba32f97e7bcded?tpId=37&&tqId=21296&rp=1&ru=/activity/oj&qru=/ta/huawei/question-ranking)

![image-20220829205115090](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220829205115090.png)

思路：

用静态数组存储每个月的天数；因为闰年和闰月的关系，需要额外增加一个函数处理，天数的累加：如果只有1月，那就直接打印即可，大于1月的将当前月之前到1月（包括1月）所有月份天数累加，然后再加上当前天数，即为该日期对应该年的第几天。

```cpp
#include <iostream>
using namespace std;
class Date
{
public:
    Date(int year, int month, int day)
        :_year(year)
        , _month(month)
        , _day(day)
    {}
    int GetMonthDay(int year, int month)
    {
        static int days[13] = { 0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
        int day = days[month];
        if (month == 2
            && ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)))
        {
            day += 1;
        }

        return day;
    }
    int GetDay(int year, int month, int day)
    {
        int Ret = 0;
        if (month == 1)
            return day;
        for (int i = month - 1; i >= 0; i--)
        {
            Ret += GetMonthDay(year, i);
        }
        Ret += day;
        return Ret;
    }
    void DayPrint()
    {
        cout << GetDay(_year, _month, _day) << endl;
    }
private:
    int _year;
    int _month;
    int _day;
};
void test()
{
    int year = 0, month = 0, day = 0;
    cin >> year >> month >> day;
    Date d(year, month, day);
    d.DayPrint();
}
int main() {
    test();
    return 0;
}
```



##  5.3 [****日期差值****](https://www.nowcoder.com/practice/ccb7383c76fc48d2bbc27a2a6319631c?tpId=62&&tqId=29468&rp=1&ru=/activity/oj&qru=/ta/sju-kaoyan/question-ranking)

![image-20220829213050381](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220829213050381.png)

思路：直接复用上一道题的代码和思路

- 同一年同月：直接Day之差；
- 其他情况：假设两个日期的年份不同：![image-20220829221717752](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220829221717752.png)

​	通过割补，可以有这个格式解决不同日期相差的天数。

复用了上题的代码，并作出了改动：新增了判断闰年函数isLeepYear()，新增GetYearDay()用于得到每年的总天数（365/366）。注意日期之差要+1才是天数。

```cpp
#include <iostream>
#include <stdio.h>
using namespace std;
class Date
{
public:
    Date(int year, int month, int day)
        :_year(year)
        , _month(month)
        , _day(day)
    {}
    //得到每个月的天数
    int GetMonthDay(int year, int month)
    {
        static int days[13] = { 0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
        int day = days[month];
        if (month == 2 && isLeepYear(year))
        {
            day += 1;
        }

        return day;
    }
    int GetYearDay()
    {
        if (isLeepYear(_year))
            return 366;
        else
            return 365;
    }
    //得到当前日期在当年是第几天
     int GetDay()
    {
        int Ret = 0;
        if (_month == 1)
            return _day;
        for (int i = _month - 1; i >= 0; i--)
        {
            Ret += GetMonthDay(_year, i);
        }
        Ret += _day;
        return Ret;
    }
     //闰年
     bool isLeepYear(int year)
     {
         if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0))
         {
             return true;
         }
         return false;
     }

private:
    int _year;
    int _month;
    int _day;
};
void test()
{
    int year1 = 0, month1 = 0, day1 = 0;
    int year2 = 0, month2 = 0, day2 = 0;
    scanf("%04d%02d%02d", &year1, &month1, &day1);
    scanf("%04d%02d%02d", &year2, &month2, &day2);

    Date d1(year1, month1, day1);
    Date d2(year2, month2, day2);
    //同年同月
    if (year1 == year2 && month1 == month2)
    {
        cout << day2 - day1 + 1<< endl;
    }
    else//其他情况
    {
        int YearDay = d1.GetYearDay();
        int DayNum1 = d1.GetDay();
        int DayNUm2 = d2.GetDay();
        cout << YearDay - DayNum1 + DayNUm2 + 1 << endl;
    }
}
int main() {
    test();
    return 0;
}
```



## 5.4 [****打印日期****](https://www.nowcoder.com/practice/b1f7a77416194fd3abd63737cdfcf82b?tpId=69&&tqId=29669&rp=1&ru=/activity/oj&qru=/ta/hust-kaoyan/question-ranking)

![image-20220829235210220](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220829235210220.png)

思路：
	还是复用刚才的代码和思路。

这道题需要改动的地方就是把给出的天数转换成月数。

- 在1-12循环中判断：总天数比月天数大就下个月，然后直接将循环数给month，减少++的次数也能提升一点效率。

> 请注意：
>
> 在输入时只要两个%d即可，不要被上一题的%4d影响......
>
> 这浪费了我半小时。

```cpp
#include <iostream>
#include <stdio.h>
using namespace std;
class Date
{
public:
    Date(int year, int month, int day)
        :_year(year)
        , _month(month)
        , _day(day)
    {}
    //得到每个月的天数
    int GetMonthDay(int year, int month)
    {
        static int days[13] = { 0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
        int day = days[month];
        if (month == 2 && isLeepYear(year))
        {
            day += 1;
        }

        return day;
    }
    //闰年
    bool isLeepYear(int year)
    {
        if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0))
        {
            return true;
        }
        return false;
    }

    void DayPrint()
    {
        int month = 1;
 				int day = _day;
        for (int i = 1; i <= 12; i++)
        {
            if (day > GetMonthDay(_year, i))
            {
                day -= GetMonthDay(_year, i);
            }
            else
            {
                month = i;
                break;
            }
        }
        printf("%4d-%02d-%02d\n", _year, month, day);
        
    }

private:
    int _year;
    int _month;
    int _day;
};
void test()
{
    int year = 0, day = 0;
    while (scanf("%d%d", &year, &day) != EOF)
    {
        Date d(year, day);
        d.DayPrint();
    }
}
int main() {
    test();
    return 0;
}
```

##  5.5 [****日期累加****](https://www.nowcoder.com/practice/eebb2983b7bf40408a1360efb33f9e5d?tpId=40&&tqId=31013&rp=1&ru=/activity/oj&qru=/ta/kaoyan/question-ranking)

![image-20220829235541720](https://raw.githubusercontent.com/shawyxy/IMG/main/IMG/MDimage-20220829235541720.png)

思路：

使用运算符重载。实际上运算符重载只是一个语法名称，它本质上还是一个函数，思路和上一题类似：以被加的日期为起点，然后将两个天数加起来，符合天数大于月天数就月数往后走，一旦超过12，年数加1，以此类推，直到不符合条件。

```cpp
#include <iostream>
using namespace std;
class Date
{
public:
    Date(int year, int month, int day)
        :_year(year)
        , _month(month)
        , _day(day)
    {}
    int GetMonthDay(int year, int month)
    {
        static int days[13] = { 0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
        int day = days[month];
        if (month == 2
            && ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)))
        {
            day += 1;
        }

        return day;
    }
    //运算符重载
    Date& operator=(const Date& d)
    {
        if (this != &d)
        {
            _year = d._year;
            _month = d._month;
            _day = d._day;
        }
        return *this;
    }
    Date& operator+=(int day)
    {
        _day += day;
        while (_day > GetMonthDay(_year, _month))
        {
            _day -= GetMonthDay(_year, _month);
            ++_month;
            if (_month == 13)
            {
                _year++;
                _month = 1;
            }
        }

        return *this;
    }
    void DayPrint()
    {
        printf("%4d-%02d-%02d\n", _year, _month, _day);
    }
private:
    int _year;
    int _month;
    int _day;
};
void test()
{
    int year = 0, month = 0, day = 0, n = 0, num = 0;
    cin >> n;
    while (n)
    {
        scanf("%d %d %d %d", &year, &month, &day, &num);
        Date d(year, month, day);
        d += num;
        d.DayPrint();
        n--;
    }
}
int main() {
    test();
    return 0;
}
```

两个运算符重载，+和+=，后者复用了前者，比较高效。
