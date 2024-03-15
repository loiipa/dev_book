---
author: "lama"
date: 2024-02-24
linktitle: 다중상속시 메모리 레이아웃
weight: 10
---


## 다중상속시 메모리 레이아웃

### Child Class 생성

아래와 같이 Coffee, Milk를 상속받는 CafeLatte Class를 생성 후, size와 메모리 레이아웃을 확인해보려고 한다.
``` c++
class Coffee
{
public:
    virtual ~Coffee() = default;
    virtual bool HasCoffee() { return false; }
    int shotSize;
};

class Milk
{
public:
    virtual ~Milk() = default;
    virtual bool HasMilk() { return false; }
    int milkSize;
};

class CafeLatte : public Coffee, public Milk
{
public:
    bool HasCoffee() override { return true; }
    bool HasMilk() override { return true; }

    int price;
};

int main()
{
    CafeLatte* polyCafeLatte = new CafeLatte;

    int size = sizeof(CafeLatte);       // size = 40

    // 디스어셈블리를 통한 메모리 레이아웃 확인용
    polyCafeLatte->price = 5000;
    polyCafeLatte->shotSize = 3;
    polyCafeLatte->milkSize = 250;

    polyCafeLatte->HasCoffee();
    polyCafeLatte->HasMilk();
    ///////////////////////////////////////////

    delete polyCafeLatte;
} 
```

이때, 메모리 레이아웃은 아래와 같다.
``` c++
(size : 40, 8byte padding)

[32]	CafeLatte::price
[24]	Milk::milkSize
[16]	Milk::vtable
[08] 	Coffee::shotSize
[00]	Coffee::vtable

```

### Child Class 생성, Parent Class에 정의

만약 아래와 같이 다형성을 이용하여 parent class에 child class를 정의하면 어떻게 될까?  
```c++
int main()
{
    Coffee* polyCoffee = new CafeLatte;
    Milk* polyMlik = new CafeLatte;

    delete polyCoffee;
    delete polyMlik;
} 
```

특이하게, Milk에 정의하는 CafeLatte 생성 이후, 메모리 주소를 10h(16) 더한다.  
```c++
Milk* polyMlik = new CafeLatte;
    00007FF6B2BE64E1  mov         rax,qword ptr [rbp+1F8h]  
    00007FF6B2BE64E8  add         rax,10h  
    00007FF6B2BE64EC  mov         qword ptr [rbp+200h],rax 
```

즉, polyMlik의 주소는 메모리 레이아웃에서 16을 가리킨다.  
물론, polyCoffee 주소는 00을 가리킨다.
