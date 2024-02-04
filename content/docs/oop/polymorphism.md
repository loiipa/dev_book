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

## 형태

### Ad hoc polymorphism
개별적으로 지정된 타입의 임의의 집합에 대한 공통 인터페이스를 정의한다.   
대표적으로 **function overloading**, **operator overloading**이 있다.  

{{< tabs "ad hoc polymorphism" >}}
{{< tab "function overloading" >}}
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

Add operator+(const int lhs, const Add& rhs) 
{
    return Add(lhs + rhs.value);
}
```
{{< /tab >}}
{{< /tabs >}}


### Parametric polymorphism
구체적인 유형(concrete type)을 지정하지 않고, 대신 모든 타입을 대체할 수 있는 추상기호를 사용한다.  

파라메트릭 다형성 함수와 데이터 타입은 **generic function**과 **generic datatypes**으로 불리며, 제네릭 프로그래밍(generic programing)의 기초를 형성한다.

c++에서는 Template를 통해 generic programming 기법을 사용할 수 있다. (대표적으로 std::swap)

{{< tabs "ad hoc" >}}
{{< tab "generic function" >}}
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
}
```
```
// out
3
3.75
c
```
{{< /tab >}}
{{< /tabs >}}

### Subtyping
(*subtype polymorphism* 혹은 *inclusion polymorphism*이라고도 함)  
이름이 어떤 공통 수퍼클래스(superclass)에 의해 연관된 다양한 클래스들의 인스턴스(instance)를 나타낼 때이다.  

![example of subtyping](/subtyping.png)  

1987년 oop 컨퍼런스 기조 연설에서 이를 널리 알린 Babara Liskov의 이름을 따서 **Liskov substitution principle**([리스코프 치환 원칙](https://ko.wikipedia.org/wiki/리스코프_치환_원칙))이라고 불리기도 한다.


가변 객체(mutable objects)를 고려해야 하기 때문에, Liskov가 정의한 이상적인 subtyping([behvioral subtyping](https://en.wikipedia.org/wiki/Behavioral_subtyping))은 type checker에서 구현할 수 있는 것 보다 훨씬 더 강력하다.

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

OOP 언어에서는 subclassing(*inheritance*(상속)이라고도 함)을 사용하여 subtype polymorphism을 제공한다.

일반적인 구현에서 각 class는 *virtual table*([가상 테이블](https://en.wikipedia.org/wiki/Virtual_method_table), 줄여서 vtable)을 포함하며, 각 객체는 해당 class의 vtable에 대한 pointer를 포함하고, polymorphic method(다형성 메서드)가 호출될 때 참조된다.


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
