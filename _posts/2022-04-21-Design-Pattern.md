# Design Pattern

---

## Overview

![DesignPattern Overview](https://github.com/DavidEugen/DavidEugen.github.io/blob/main/img/posts/Design-Patterns-Relationships.png?raw=true)



## 디지안 패턴의 분류

![디자인패턴분류](https://user-images.githubusercontent.com/10527294/165739867-afef22cb-a546-4499-89e3-1c5b22847300.jpeg)

- 목적 (purpose) 에 따라 
  - 생성, 구조 행동 으로 분류
- 범위 (Scope) 에 따라
  - 패턴을 주로 클래스에 적용하는지 아니면 객체에 적용하는지 구분
  - 클래스 패턴은 클래스와 서브클래스 간의 관계(relationship) 다루는 패턴
    - 관계 주로 상속이며, 컴파일 타임에 정적으로 결정됨
  - 객체 패턴은 객체간의 관계을 다루는 패턴
    - 런타임에 변경할 수 있으며 더 동적인 성격을 가짐



## 생성 - 객체의 생성 과정에 관여

생성 패턴은 **인스턴스를 만드는 절차를 추상화** 하는 패턴. 객체를 생성, 합성하는 방법이나 객체의 표현 방법과 시스템을 분리해 준다. 

| 클래스 생성 패턴                                             | 객체 생성 패턴                                 |
| ------------------------------------------------------------ | ---------------------------------------------- |
| 인스턴스로 만들 클래스를 다양하게 만들기 위한 용도로 상속을 사용 | 인스턴스화 작업을 다른 객체에게 떠넘길 수 있음 |

생성 패턴은 시스템이 어떤 구체 클래스를 사용하는지에 대한 정보를 캡슐화 한다.

생성 패턴은 이들 클래스의 인스턴스들이 어떻게 만들고 어떻게 서로 맞붙는지에 대한 부분을 완전히 가려준다.

결론적으로 생성패턴을 이용하면 **무엇이** 생성되고, 누가 이것을 생성하며, 이것이 **어떻게** 생성되는지, **언제** 생성할 것인지에 대한 <u>유연성을 확보 할 수 있다</u>.

### 클래스

객체를 생성하는 책임의 일부를 서브 클래스가 담당하도록 넘김

- Factory Method Pattern

### 객체

객체를 생성하는 책임의 일부를 다른 객체에게 위임

- [Singleton Pattern][싱글톤패턴_URL]
  - 인스턴스를 하나로 공유해서 쓰고 싶을 때
  - 여러개의 인스턴스를 생성할 필요가 없을 때
  - 전역에서 접근 할 수 있는 포인트를 제공
- Abstract Factory Pattern
  - 상세화된 서브클래스를 정의하지 않고도 서로 관련성이 있거나 의존적인 여러 객체의 군을 생성하기 위해 인터페이스를 제공
- Build Pattern
- Prototype Pattern

## 구조 - 클래스나 객체의 합성에 관한 패턴

### 클래스

상속을 이용하여 클래스를 합성(compose)함

- Adapter Pattern

### 객체

객체를 조립(Assemble)하는 방법을 설명함

- Adapter Pattern ( object )
- Bridge Pattern
- Composite Pattern
- Decorator Pattern
- Facade Pattern
- Flyweight Pattern
- Proxy Pattern

## 행동 - 클래스나 객체들이 상호작용하는 방법과 책임을 분산하는 방법에 관한 패턴

### 클래스

상속을 이용하여 알고리즘과 제어 흐름을 기술

- Interpreter  Pattern
- Template Method Pattern

### 객체

하나의 작업을 수행하기 위해 객체 집합이 어떻게 협력하는지 기술

- Chain Of Responsibility Pattern
- Command Pattern
- Iterator Pattern
- Mediator Pattern
- Memento Pattern
- Observer Pattern
- State Pattern
- Strategy Pattern
- Visitor Pattern

---

참고

[디자인패턴 이미지 RUL]: https://www.researchgate.net/figure/Design-Patterns-Relationships-3-Deene-the-context-in-which-the-design-patterns-are_fig7_220306648
[싱글톤패턴_URL]: https://github.com/DavidEugen/TIL/blob/main/DesignPattern/Create_Singleton.md
