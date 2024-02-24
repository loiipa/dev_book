---
date: 2024-02-04
title: Polymorphism (다형성)
type: docs
---
# Polymorphism

## 정의

C++ 창시자인 Bjarne Stroustrup은 다형성을 아래와 같이 [정의](https://www.stroustrup.com/glossary.html#Gpolymorphism)했다.
> Polymorphism is the provision of a single interface to entities of diffeent types  
(서로 다른 타입의 엔터티에 단일 인터페이스를 제공하는 것)

[자바 튜토리얼](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html)에서는 생물학에서 유기체(organism)나 종(species)이 여러 가지 형태나 단계를 가질 수 있는 원리를 시작으로 다형성에 대해 설명한다.

## 내가 생각하는 다형성
나는 다형성을 개념보다는 C++ 코드의 function overloading으로 처음 접했고,
template, parent - child class 상속관계는 실제 사용하긴 했지만, 다형성이라는 범주로 이해하지는 않고 있었다.

다형성이라는 것을 학습하고 나서는, 다형성을 **하나의 심볼(상징이 될 수도 있는 무언가의 형태)에 이와 유사한 여러 가지 의미를 투영할 수 있는 것** 이라 생각이 들었다.

ad hoc과 Parametric은 function name을 기준으로 각각 인자값과 불특정 T값으로 인해 의미가 확장되고
subtyping은 base class를 기준으로 여러 child class로 인해 의미가 확장되기 때문이다.

물론, 단순히 같은 function name에 다른 인자를 받긴 하지만, 리턴되는 값이 의도한 바와 매우 이질적이라면, 이는 다형성을 의도하기 위한 구조적인 형태는 지켰지만, 이것이 실제 다형성이라 하기는 어렵다고 보인다.



## 형태

### Ad hoc polymorphism
ad hoc polymorphism(임시 다형성)은 쉽게 표현하면,   
다형성 함수(polymorphic function)가 인자를 받는 부분을 다르게 하여 차별을 두는 다형성의 일종이다. 

객체지향 개념에서는  **function overloading**(함수 오버로딩), **operator overloading**(연산자 오버로딩) 이 있다.  

{{< tabs "ad hoc polymorphism" >}}
{{< tab "function overloading" >}}

* 함수 오버로딩은 아래와 같이, 같은 이름을 가진 함수가 인자를 다르게 두어 각각 입력된 인자의 의도에 맞는 함수에 진입하여 과정을 수행하거나, 결과를 가져오는 것을 의도하는 것으로 이해할 수 있다.

```c++
int Area(int width, int height)
{
    return width * height;
}

double Area(double radius)
{
    return 3.14 * radius * radius;
}
```
{{< /tab >}}

{{< tab "operator overloading" >}}

* 연산자 오버로딩은 함수 오버로딩과 마찬가지로, 같은 이름을 가진 연산자가 인자를 다르게 두어 의도에 맞는 연산자로 진입하는 것으로 이해할 수 있다.

```c++
class Add
{
public:
    Add(const int value) : value(value) {}

    Add operator+(const Add& rhs) const
    {
        Add temp = *this;
        temp.value += rhs.value;
        return temp;
    };

    int value;
};

// lhs는 어떤 유형이 와도 상관없다.
Add operator+(const int lhs, const Add& rhs) 
{
    return Add(lhs + rhs.value);
}

Add operator+(const double lhs, const Add& rhs) 
{
    return Add(lhs + rhs.value);
}
```
{{< /tab >}}
{{< /tabs >}}


### Parametric polymorphism
구체적인 유형(concrete type)을 지정하지 않고, 대신 모든 타입을 대체할 수 있는 추상기호를 사용한다.  

파라메트릭 다형성 함수와 데이터 타입은 **generic function**과 **generic datatypes**으로 불리며, 제네릭 프로그래밍(generic programing)의 기초를 형성한다.

c++에서는 Template를 통해 generic programming 기법을 사용할 수 있다. (대표적으로 std::swap())

{{< tabs "ad hoc" >}}
{{< tab "generic function" >}}

* c++에서 템플릿은 컴파일 시간에 함수의 인자를 기반으로 함수를 생성하는 구문이다. 컴파일 시간에 T의 타입을 추론 하여 특정한 타입(int, double 등.. )으로 인스턴스화 한다.

```c++
template <typename T>
T Add(const T& left, const T& right)
{
    return left + right;
}

main()
{
    std::cout << Add(1, 2) << std::endl;
    std::cout << Add(1.5, 2.25) << std::endl;
    std::cout << Add('1', '2') << std::endl;
    std::cout << Add(std::string("1"), std::string("2")) << std::endl;
}
```
```
// output
3
3.75
c
12
```
{{< /tab >}}

{{< tab "generic datatype" >}}

* class에 template를 이용하여 class 내부의 멤버변수를 generic으로 사용할 수 있다.

```c++
template <typename T>
class Shape
{
public:
    Shape(const T& area) : area(area) {};

    T GetArea() const { return area; }
    void SetArea(const T& area) { this->area = area; }

private:
    T area;
};

int main()
{
    Shape<int> shapeInt(3);
    Shape<std::string> shapeStr("101.0");
    Shape<double> shapeDouble(3.1414);

    std::cout << shapeInt.GetArea() << std::endl;
    std::cout << shapeStr.GetArea() << std::endl;
    std::cout << shapeDouble.GetArea() << std::endl;
}
```

```
// output
3
101.0
3.1414
```

{{< /tab >}}
{{< /tabs >}}

### Subtyping
이론적으로는 상위 유형(supertype)의 요소에서 작동하도록 작성된 요소가 하위 유형에서도 작동하는 것을 뜻한다.

OOP 언어에서는 subclassing(*inheritance*(상속)이라고도 함)을 사용하여 subtype polymorphism을 제공한다.

C++에서는 부모 클래스를 상속받은 자식 클래스들의 인스턴스를, 부모 클래스의 포인터나 레퍼런스를 통해 사용하는 것이 대표적이다.

![example of subtyping](/dev_book/subtyping.png)  


{{< tabs "Subtyping" >}}
{{< tab "virtual function" >}}
```c++
class Animal
{
public:
    virtual std::string Speak() = 0;
};

class Dog : public Animal
{
public:
    std::string Speak() override { return "woof"; }
};

class Cat : public Animal
{
public:
    std::string Speak() override { return "meow"; }
};

void PrintSpeak(std::shared_ptr<Animal> animal)
{
    std::cout << animal->Speak() << std::endl;
}

int main()
{
    PrintSpeak(std::make_shared<Dog>());
    PrintSpeak(std::make_shared<Cat>());
}
```
```
// out
woof
meow
```
{{< /tab >}}
{{< /tabs >}}

일반적인 구현에서 각 class는 *virtual table*([가상 테이블](https://en.wikipedia.org/wiki/Virtual_method_table), 줄여서 vtable)을 포함하며, 각 객체는 해당 class의 vtable에 대한 pointer를 포함하고, polymorphic method(다형성 메서드)가 호출될 때 각 class의 vtable이 참조된다.

다형성 메서드는 위와 같이 animal에 dog가 정의되었을 때, 기존 dog 객체가 가지고 있던 vtable pointer를 통해 Dog::Speak()함수가 호출되게 된다.

#### 리스코프 치환 원칙과 behvioral subtyping

리스코프 치환 원칙은 자료형 **S**가 자료형 **T**의 subtype이라면, 속성 변경 없이 자료형 **T**의 객체를 자료형 **S**로 치환할 수 있어야 한다는 원칙이다.

이 원칙은 1987년 oop 컨퍼런스 기조 연설에서 이를 널리 알린 Babara Liskov의 이름을 따서 **Liskov substitution principle**([리스코프 치환 원칙](https://ko.wikipedia.org/wiki/리스코프_치환_원칙))이라고 불리기도 한다.

가변 객체(mutable objects)를 고려해야 하기 때문에, Liskov가 정의한 이상적인 subtyping([behvioral subtyping](https://en.wikipedia.org/wiki/Behavioral_subtyping))은 type checker에서 구현할 수 있는 것 보다 훨씬 더 강력하다.

예를 들면, 직사각형을 상속받은 정사각형이 있는데, 각각 가로와 세로 멤버변수를 수정하는 setter 함수가 있다고 하자.  정사각형의 가로와 세로는 똑같기 때문에, 가로를 변경하면, 세로도 같이 변경이 되어야 한다.  
이때, 정사각형의 어느 한 변을 수정시, 가로와 세로 둘 다 변경되기 때문에, 직사각형의 각각의 변의 길이의 수정에 대한 의도에 벗어난 정사각형의 변 크기 수정은 리스코프 치환 원칙에서 위배된다고 할 수 있다.




## 참고자료
**Polymorphism**  
https://en.wikipedia.org/wiki/Polymorphism_(computer_science)  
https://www.stroustrup.com/glossary.html#Gpolymorphism
https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html


**Ad hoc polymorphism**  
https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B


**Parametric polymorphism**  
https://en.wikipedia.org/wiki/Parametric_polymorphism  

**Subtyping**
https://ko.wikipedia.org/wiki/%eb%a6%ac%ec%8a%a4%ec%bd%94%ed%94%84_%ec%b9%98%ed%99%98_%ec%9b%90%ec%b9%99


2024.02 작성
