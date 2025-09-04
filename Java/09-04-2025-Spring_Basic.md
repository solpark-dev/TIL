# TIL (Today I Learned) - 2025-09-04

오늘은 스프링 프레임워크의 가장 핵심적인 개념인 **IoC 컨테이너**와 **의존성 주입(DI)**에 대해 학습했다. 직접 객체를 생성하고 연결하던 방식에서 벗어나, 프레임워크가 객체의 생명주기를 관리해주는 원리를 XML 설정을 통해 실습했다.

---

## ## 오늘의 학습 여정 (Learning Journey)  Journey

오늘의 학습은 **강한 결합(Tight Coupling)** 상태의 코드에서 출발하여, **느슨한 결합(Loose Coupling)** 구조로 리팩토링하고, 최종적으로 이 과정을 **스프링 프레임워크**에 위임하는 전체적인 흐름을 경험하는 과정이었다.

1.  **문제 인식 (`Player01`과 `DiceA`):** `Player01` 클래스가 `new DiceA()`를 통해 특정 주사위(`DiceA`)를 직접 생성하고 의존하는 **강한 결합** 상태의 코드를 분석했다. 이로 인해 다른 주사위로 교체하기 위해서는 `Player01`의 코드를 직접 수정해야 하는 비유연성의 문제를 확인했다.

2.  **리팩토링 (`Dice` 인터페이스와 `Player02`):** 이 문제를 해결하기 위해 `Dice`라는 **인터페이스(표준 규격)**를 도입했다. `Player02`는 더 이상 구체적인 클래스가 아닌 `Dice` 인터페이스에만 의존하게 하여 **느슨한 결합**을 달성했다.

3.  **의존성 주입(DI) 직접 구현:** `Player02`가 필요한 `Dice` 객체를 외부에서 **생성자(Constructor Injection)**나 **세터 메서드(Setter Injection)**를 통해 주입받는 **의존성 주입(DI)** 패턴을 직접 구현했다. 이 단계에서 `DiceTestApp02`가 객체를 생성하고 주입하는 '조립자' 역할을 수행했다.

4.  **스프링 프레임워크 도입:** 수동으로 하던 '조립' 과정을 스프링에 위임했다. `diceservice.xml`이라는 설계도를 작성하여 `Dice` 구현체들과 `Player02`를 **Bean**으로 등록하고, 스프링 **IoC 컨테이너**가 XML 설정을 읽어 자동으로 의존성을 주입(Wiring)하도록 만들었다.

5.  **에러 해결 및 심화 학습:**
    * `Cannot find the declaration of element 'beans'`: 반복적으로 발생한 XML 스키마 인식 에러를 해결하며 스프링과 IDE의 연동 원리를 이해했다.
    * `No matching constructor found`: XML에 정의된 생성자 인수와 실제 자바 클래스의 생성자가 일치하지 않을 때 발생하는 에러를 해결하며, 스프링이 **리플렉션**을 통해 어떻게 객체를 생성하는지 구체적으로 파악했다.
    * `User` 예제를 통해 다양한 DI 주입 방식(`p-namespace` 포함)을 연습하고, `BeanFactory`와 `ApplicationContext`의 핵심적인 차이(지연/선행 로딩)를 학습했다.

---

## ## IoC 컨테이너와 의존성 주입 (DI)

### ### 1. IoC (Inversion of Control, 제어의 역전)
**IoC**란, 객체의 생성, 관계 설정, 사용 등 프로그래머가 직접 하던 제어권을 프레임워크에게 넘기는 설계 원칙이다. 개발자는 핵심 비즈니스 로직에만 집중하고, 객체 관리와 같은 부가적인 작업은 **IoC 컨테이너**에 위임한다.



### ### 2. IoC 컨테이너 (Bean Container)
**IoC 컨테이너** 또는 **Bean 컨테이너**는 IoC 원칙을 구현한 관리자(Manager)다. XML 설정 파일이나 어노테이션 같은 외부 메타데이터를 읽어들여, 자바 객체(**Bean**)를 생성하고, 객체 간의 의존관계를 설정하며, 생명주기 전체를 관리하는 역할을 한다.

* **BeanFactory**: 스프링의 가장 기본적인 IoC 컨테이너. `getBean()`이 호출될 때 객체를 생성하는 **지연 로딩(Lazy Loading)** 방식을 사용한다. 현재는 거의 사용되지 않는다.
* **ApplicationContext**: `BeanFactory`를 상속받아 더 많은 기능을 제공하는 고급 컨테이너. 컨테이너가 구동되는 시점에 설정 파일의 모든 Bean을 미리 생성하는 **선행 로딩(Pre-loading)** 방식을 사용한다. 대부분의 스프링 애플리케이션에서 사용된다.

---

## ## 의존성 주입 (Dependency Injection) 방식

**Wiring**(와이어링)이란 IoC 컨테이너가 Bean들 사이의 의존 관계를 설정(연결)해주는 과정을 의미한다. 스프링에서는 주로 두 가지 주입 방식을 사용한다.

### ### 1. Setter Injection (세터 주입)
기본 생성자로 Bean 객체를 생성한 후, **setter 메서드**를 호출하여 의존성을 주입하는 방식이다. 선택적인 의존성을 주입하거나, 나중에 의존성을 변경해야 할 때 유용하다.

* **XML 설정:** `<property>` 태그 사용
    ```xml
    <bean id="player" class="com.example.Player">
        <property name="dice" ref="myDice"/>
    </bean>
    ```

### ### 2. Constructor Injection (생성자 주입)
객체를 생성하는 시점에 **생성자의 인자**를 통해 의존성을 주입하는 방식이다. 객체가 생성됨과 동시에 완전한 상태가 되는 것을 보장하므로, 필수적인 의존성을 주입할 때 주로 사용된다.

* **XML 설정:** `<constructor-arg>` 태그 사용
    ```xml
    <bean id="player" class="com.example.Player">
        <constructor-arg ref="myDice"/>
    </bean>
    ```


---

## ## XML을 이용한 Bean 설정 요소

스프링 IoC 컨테이너는 XML 설정 파일을 읽어 객체를 관리한다. 이때 사용되는 주요 속성은 다음과 같다.

* **`id`**: 컨테이너 내에서 Bean을 식별하기 위한 **고유한 이름**. `getBean("id")` 형태로 사용한다.
* **`class`**: Bean을 생성할 클래스의 **패키지 경로를 포함한 전체 이름**(FQCN).
* **`<property name="...">`**: Setter Injection 시 호출할 **setter 메서드의 이름**을 지정한다. `setDice()` 메서드라면 `name="dice"`로 기술한다.
* **`ref`**: 주입할 의존 객체의 **`id`를 참조**할 때 사용한다. `ref="myDice"`는 `id`가 `myDice`인 Bean을 주입하라는 의미다.

```markdown
