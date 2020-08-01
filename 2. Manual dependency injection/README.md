# #2 Manual dependency injection

- [안드로이드 권장 앱 아키텍처](https://developer.android.com/jetpack/docs/guide#recommended-app-arch)는 코드를 나눠 **관심사를 분리**하는 것을 장려하고 있다. 이것은 계층의 각각의 클래스가 하나의 정의된 역할을 가지게 하는 원칙이다. 이 때 분리된 계층 간 의존성을 충족시키기 위해서는 서로 연결되어야 하는 더 많고 작은 계층들로 이어진다.

![https://developer.android.com/topic/libraries/architecture/images/final-architecture.png](https://developer.android.com/topic/libraries/architecture/images/final-architecture.png)

- 클래스 간의 의존성은 각 클래스가 의존하는 클래스에 연결되는 그래프로 나타낼 수 있다. 그리고 그래프에서 각 클래스는 종속된 클래스에 연결된다. 그리고 모든 종속성을 표시하면 어플리케이션 그래프가 구성된다.

- 어플리케이션 그래프의 추상적인 개념을 보여준다. 예로 VIewModel이 Repository에 의존하고 있기 때문에 그래프에서는 ViewModel에서 Repository를 가리키는 선을 그린다.

- 의존성 주입을 통해서 클래스는 쉽게 연결할 수 있고, 테스트를 위해서 구현체를 교체할 수 있다.

  ```kotlin
  class MainViewModel {
      private val clubRepository = ClubRepository()
      
      init {
          // ...
      }
  }
  
  class ClubRepository {
      private val remoteClubDataSource = RemoteClubDataSource()
      
      fun getClubList(val clubName : String) {
          remoteClubDataSource(clubName)
      }
      
  }
  ```

  - 만약 위의 코드에 있는 `MainViewModel` 클래스를 테스트 할 때, 테스트를 위해서는 내부에 있는`ClubRepository`를 인스턴스화해야 한다. 그래서 `ClubRepository` 내부를 확인해 보니`RemoteClubDataSource`도 같이 인스턴스화해야 하고, `RemoteClubDataSource`가 또 다른 인스턴스를 생성해야 하는, 결합도가 아주 강한 상태라고 가정하자.
  - `MainViewModel` 하나를 테스트하려고 내부에 있는 `ClubRepository`와 `RemoteClubDataSource`, 그 이외의 객체들을 인스턴스화해야 한다고 생각해 보자. -> ViewModel 하나 테스트 한다고, Repository와 DataSource 인스턴스도 생성한다면 비효율적이다. 테스트 용이성(Testability)이 떨어진다고 할 수 있다.

  ```kotlin
  class MainViewModel(private val clubRepository : ClubRepository) {
      
      
      init {
          // ...
      }
  }
  ```

  - 위처럼 `MainViewModel` 밖에서 `ClubRepository`를 주입받아 작성하는 형태로 작성하고, `ClubRepository`의 가짜 구현체나 모의 구현체 (Fake, Mock)를 사용해 테스트한다면 전 코드보다 테스트하기에 용이하다.

- 위처럼 의존성 주입은 확장 및 테스트 가능한 Android 앱을 만드는 데 유용한 기법이다. 의존성에 필요한 클래스들의 인스턴스를 가지고 있는 Container를 이용해 앱의 다양한 부분에서 필요한 경우에 인스턴스를 공유할 수 있다.

- 단 Container를 제작하는 등 수동적으로 의존성을 주입하는 경우에는 프로젝트가 커지는 경우에는 추천하지 않는다. 프로젝트가 커져 작성해야 하는 코드의 수가 늘어나면 그에 따른 상용구 코드를 많이 작성하게 되는데, 상용구 코드는 오류가 발생하기 쉬운 코드의 일종이기 때문이다.  또한 컨테이너의 범위, 수명 주기를 직접 지정하고 관리한다면, 내부에서 버그나 메모리 누수가 발생할 수 있다.

- 수동적으로 의존성 주입을 한다면 위와 같은 문제점이 발생할 수 있기 때문에, 의존성 주입 프로세스를 자동화해주는 라이브러리들에 대해서 알아볼 것이다.