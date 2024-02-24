---
author: "lama"
date: 2024-02-24
linktitle: 다중상속 순서의 중요성
weight: 10
---


## 문제상황

업무 중에 C++ 관련 Chart Library를 A -> B로 교체하는 일이 있었다.
이때 A를 완전히 없애지 않고 B 라이브러리를 추가하기 위해 아래와 같이 class 구조를 계획했다.  
![example - multiple inheritance](/dev_book/multiple_inheritance_001.png)  

그리고 MFC의 View를 상속받은 ViewBase에서 ChartCtrlBase를 연결해주었다. (MFC에서 Create() 함수 호출)  
쉽게 말하면, viewBaseClass는 chartCtrlBase class의 부모가 된다.  
그래서 ChartCtrl->GetParent() 함수를 호출시 View class의 주소를 얻게 된다.  
![view to chart](/dev_book/multiple_inheritance_002.png)  

그리고 mouse event 연결을 위해 ChartCtrlA class에서 this->GetParent() 함수를 호출하여 ViewBaseClass의 주소를 가져와서 message를 전달하고자 했다.  
하지만 GetParent()의 리턴값이 NULL인 상황이라 message 전달에 실패하였다.


## 문제원인

### 주솟값 차이

문제상황에 대해 계속 추론하며 확인하는 도중, ChartCtrl에서 실제 Create() 함수를 호출할 때의 주소와, GetParent()를 호출하는 this 주소가 달랐고, 주소의 차이는 8이었다.  
![memory diff is 8](/dev_book/multiple_inheritance_003.png)  

### 주솟값 차이는 다중상속과 관련이 있다.

다중상속시 메모리 할당이 어떻게 되있는지를 알고 있다면, 해당 문제가 무엇인지 쉽게 파악할 수 있다.


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


## 문제해결

### 문제 상황 재정리
다중상속에서의 메모리 레이아웃을을 확인한 결과, 해당 문제는 다중상속의 순서에 의해 문제가 발생했음을 알 수 있다.  

대략적인 로직은 아래와 같다.  
```c++
class ChartCtrlA : public ChartCtrlBase, public ChartLibraryA
```

```c++
// 1. 다형성으로 인한 정의
std::unique_ptr<ChartCtrlBase> chart = std::make_unique<ChartCtrlA>();

// 2. view에 chart 등록
// 이때, 실제 Create가 불리는 class는 ChartCtrlA이다.
chart->Create( ..., this );

// 3. event handler
// ChartLibraryA 에서 event가 발생할 때, ChartCtrlA class에서 이를 처리.
```

### 문제 해결 방안
해결방안은 쉽다. 상속순서를 바꾸면 된다.  
```c++
class ChartCtrlA : public ChartLibraryA, public ChartCtrlBase
```
