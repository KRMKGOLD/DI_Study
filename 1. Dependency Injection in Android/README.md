# #1 Dependency injection in Android

- 이 글은  [Dependency Injecion in Android](https://developer.android.com/training/dependency-injection) 의 내용 중 일부를 정리 및 분석한 내용입니다.

- 의존성 주입(Dependency Injection, DI)는 프로그래밍에서 널리 사용되는 기법으로, 안드로이드 개발 시에 사용하면 좋은 기법이다. 의존성 주입의 원칙을 따름으로써, 좋은 앱 아키텍처의 기초를 닦을 수 있습니다. 의존성 주입을 프로젝트에 구현하면 아래와 같은 장점들을 얻을 수 있다.

  1. 코드의 재사용성

  2. 리팩토링 용이

  3. 테스트 용이



1. 의존성 주입 기본 원리

- 의존성 주입이란?

  - 개발 과정에서 클래스는 종종 다른 클래스의 참조를 요구한다.

    ex) Car 클래스가 Engine 클래스에 대한 참조를 필요로 하는 경우 → 이런 것들을 의존성(dependencies)이라고 부르며, Car 클래스는 Engine 인스턴스에 대한 의존한다.(dependant)

    → 위와 같이 필요한 객체를 얻는 법에는 아래와 같이 3가지가 있음

    1. Car 클래스 내에서 Engine 인스턴스를 생성, 초기화 해서 사용한다.
    2. 일부 안드로이드 API가 작동하는 방식으로,  `Context getter`나 `getSystemService()` 를 이용해 객체를 생성한다.
    3. **매개변수로 주입받는다.  클래스가 구성될 때, 타 클래스의 인스턴스가 필요한 경우 매개변수를 통해서 전달할 수 있다. → 의존성 주입의 정의**

    ```kotlin
    class Car {
    
      private val engine = Engine()
    
      fun start() {
          engine.start()
      }
    }
    
    fun main(args: Array) {
        val car = Car()
        car.start()
    }
    ```

  ![https://developer.android.com/images/training/dependency-injection/1-car-engine-no-di.png](https://developer.android.com/images/training/dependency-injection/1-car-engine-no-di.png)

  - 위의 코드는 사진처럼 Car 클래스가 Engine 클래스를 내부에서 자체적으로 제작하고 있다. 위와 같은 경우에는 아래와 같은 내용으로 문제가 발생할 수 있다.

    1. Car 클래스와 Engine 클래스가 매우 긴밀하게 결합되어 있어, Car 인스턴스가 생성되면 내부에서 Engine 인스턴스도 생성되기 때문에 자식 클래스를 생성하거나 비슷하나 다른 클래스 (대체 구현 ? ) 도 쉽게 만들기 힘들다.

       → Car 클래스가 Engine 클래스에 대해 의존도가 높기 때문에 테스트 하는데 어려움을 겪는다. 테스트 과정에서 Engine 클래스는 내부에서 실제 인스턴스가 생성되기 때문에 Test Double( ex) Mocking..., Facking...  ) 을 이용해 Engine을 변경할 수 없다.

  - 의존성 주입을 이용하는 코드

    ```kotlin
    class Car(private val engine: Engine) {
        fun start() {
            engine.start()
        }
    }
    
    fun main(args: Array) {
        val engine = Engine()
        val car = Car(engine)
        car.start()
    }
    ```

    ![https://developer.android.com/images/training/dependency-injection/1-car-engine-di.png](https://developer.android.com/images/training/dependency-injection/1-car-engine-di.png)

    Car 인스턴스가 생성되어 init 과정을 거칠 때, Engine 객체를 내부에서 생성하는 것이 아닌 생성자에서 매개변수를 통해서 엔진 객체를 주입받는다.

  - 의존성 주입의 장점

    - 그렇다면 위와 같이 의존성 주입을 이용하여 코드를 작성한다면 어떤 이점이 있을까? 
      1. Car 객체를 재사용할 수 있다 : Car 클래스가 사용하기를 원하는 ElectricEngine이라는 엔진의 하위 클래스를 정의해서 사용할 수 있다. 전 코드에서 의존성 주입을 사용하지 않았다면 Car 인스턴스를 다시 생성해서 사용해야 했지만 의존성 주입을 사용한 코드에서는 ElectricEngine을 재사용할 Car 클래스에 주입해 사용하면 된다
      2. 쉬운 테스팅 과정 : 주입을 통해 실제 객체를 생성하는 과정이 Car 클래스에서 없어졌기 때문에, FakeEngine 등 테스트 더블 객체를 생성하여 테스팅할 수 있다.

  - 안드로이드에서의 의존성 주입

    - 안드로이드에서는 크게 두 가지로 의존성 주입이 나뉜다. 

      1. Constructor Injection : 위의 코드와 같은 방식으로 생성자를 통해 인스턴스를 전달하는 방식.

      2. Field Injection(or Setter Injection) : Activity나 Fragment는 안드로이드 내부 시스템을 통해서 인스턴스화 되기 때문에 1번과 같은 방식은 불가능하다, 이 때 필드 주입을 이용하면 클래스가 생성된 후 주입하는 방식이다. 방식은 아래와 같다.

         ```kotlin
         class Car {
             lateinit var engine: Engine
         
             fun start() {
                 engine.start()
             }
         }
         
         fun main(args: Array) {
             val car = Car()
             car.engine = Engine()
         		// engine을 Car 객체가 생성된 이후에 setter를 이용해 주입하는 방식
             car.start()
         }
         ```

1. 자동 종속성 주입

   - 위의 예처럼 라이브러리에 의존하지 않고 직접 종속성을 생성하고 제공, 관리하는 경우는 수동 종속성 주입이라고 한다. Car 클래스 예제처럼 의존성이 한 개가 아닌, 의존성과 클래스가 증가하면 수동적 주입이 지루하고 어려운 과정이 될 수 있다.

   - 대형 앱의 경우 종속성을 올바르게 주입하고 연결하려면 많은 보일러 플레이트 코드를 작성해야 할 수 있다. 최상위 레이어의 객체를 생성하기 위해서는 그 하위 레이어의 모든 종속성을 제공에 주입해줘야 한다.

   - 또한 lazy한 초기화나, App flow에 맞게 종속성을 구성할 수 없는 경우에는 종속성 수명을 관리하는 사용자 정의 컨테이너(또는 종속성 그래프)를 작성하고 유지 및 관리해야 한다.

   - 위와 같은 이유로 의존성 주입을 자동으로 해 주는 라이브러리들이 있다 

     - 런타임에 의존성을 연결해주는 리플렉션 기반 라이브러리.

     - **컴파일 과정에서 의존성을 연결하는 코드를 만들어 적용하는 정적 라이브러리.**

   - **Dagger는 구글이 유지하고 있는 안드로이드의 대표적인 의존성 주입 라이브러리로, 프로젝트 내에서 DI를 쉽게 사용할 수 있다.** Guice와 같이 리플렉션 기반인 DI 라이브러리의 개발 및 성능 문제를 대체할 수 있는 완전 정적 및 컴파일 시간 의존 라이브러리를 제공한다.

2. 의존성 주입의 대한 대안

   - DI의 대안으로 서비스 로케이터 패턴을 사용하는 방법이 있음

   ```kotlin
   object ServiceLocator {
   	fun getEngine(): Engine = Engine()
   }
   
   class Car {
       private val engine = ServiceLocator.getEngine()
   
   	fun start() {
       	engine.start()
   	}
   }
   
   fun main(args: Array) {
       val car = Car()
       car.start()
   }
   ```

   - [서비스 로케이터는 안티패턴이다](https://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) 라는 글을 읽고 작은 프로젝트가 아니라면 직접적으로 사용하지 않을 것이라고 생각했음

3. 프로젝트에 적합한 기술 선택

- 프로젝트 내 클래스 등의 의존성을 관리하기 위한 몇 가지 다른 기술들이 존재한다.
  1. 수동 의존성 주입은 크기가 작아 상대적으로 작은 앱에만 의미가 있다. 큰 프로젝트에서는 많은 보일러 플레이트 코드를 필요로 한다.
  2. 서비스 로케이터 패턴은 수동 의존성 주입보다는 적은 보일러 플레이트 코드를 발생시키지만, 싱글톤에 의존하기 때문에 Testing 측면에서는 더 좋지 않다.
  3. Dagger는 규모가 작은 프로젝트에도, 규모가 큰 프로젝트에도 알맞게 적용할 수 있다. 복잡한 앱을 만들기에도 적합한 DI 라이브러리이다.
- 작은 프로젝트가 확장되어 큰 프로젝트가 될 것 같다면 Dagger로 일찍이 변경하는 것을 고려해야 할 것이다.

4. 결론
   - 아래와 같은 장점들이 존재한다.
     - 클래스의 재사용 가능성과 의존성 분리
     - 리팩토링의 용이함
     - 테스트의 용이함

