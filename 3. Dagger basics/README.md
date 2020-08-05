# 3. Dagger Basics

- Android 앱에서 수동으로 의존성 주입을 하는 경우에는 프로젝트의 크기가 커질 수록 문제가 발생할 수 있다. 이런 상황에는 **Dagger**를 이용하여 자동적으로 종속 항목들을 관리해 프로젝트가 커질 경우에도 프로젝트의 복잡성을 줄일 수 있다.

- Dagger는 개발자가 직접 작성했을 코드를 모방하는 코드를 자동적으로 생성해 컴파일 시점에 코드가 생성되므로 Guice와 같은 리플렉션을 이용한 의존성 주입 라이브러리보다 성능이 뛰어나다.

## Dagger 사용 시의 이점

- Dagger 사용 시 AppContainer와 같은 종속 항목을 담아놓은 클래스를 자동으로 생성해 준다. (우리는 `@Inject` 등으로 주입 후 사용만 하면 됨)
- 필요한 클래스의 팩토리를 생성해 줌
- `Scope`를 이용해서 종속 항목을 재사용할 수 있고 재생성을 막아 효율성을 높힘
- `SubComponent`를 이용하여 불필요한 의존성 or 재사용할 의존성을 관리한다.

## Dagger의 간단한 사용 사례 - 팩토리 생성

![img](https://developer.android.com/images/training/dependency-injection/3-factory-diagram.png?hl=ko)

- `UserRepository`는 `UserLocalDataSource`와 `UserRemoteDataSource`에 의존하고 있다. 아래 Snippet처럼 `UserRepository`를 작성한다.

```kotlin
class UserRepository(
	private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) { ... }
```

- Dagger가 `UserRepository`를 생성하는 방법을 알 수 있도록 `UserRepository`생성자에 `@Inject` 어노테이션을 추가한다.

```kotlin
class UserRepository @Inject constructor{  
	private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
} { ... }
```

- 위의 코드 Snippet에서는 `@Inject`를 통해서 `UserRepository`를 제작하는 방법에 대해서 Dagger에 전달해주고, 의존하고 있는 항목이 `UserLocalDataSource`와 `UserRemoteDataSource`임을 알려준다.
- 이제 Dagger가 `UserRepository` 인스턴스를 생성하는 방법은 알게 되지만, 의존하고 있는 `UserLocalDataSource`와 `UserRemoteDataSource`의 인스턴스를 생성하는 방법에 대해서는 알지 못한다.
- 그럼 의존하고 있는 `UserLocalDataSource`와 `UserRemoteDataSource`에도 `@Inject` 어노테이션을 붙혀 Dagger가 만드는 방법에 대해서 알게 만들어야 한다.

```kotlin
class UserLocalDataSource @Inject constructor() { ... }
class UserRemoteDataSource @Inject constructor() { ... }
```

