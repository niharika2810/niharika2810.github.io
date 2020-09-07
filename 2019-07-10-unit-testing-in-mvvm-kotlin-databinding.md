<div style="text-align:center">
<h1> Unit Testing in MVVM+Kotlin+DataBinding
</h1
</div>
<br/>
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/testing.png">
</div>
<br/>
<br/>
## Introduction

I love Kotlin and MVVM. When I began writing the application with it here at 1mg, I instantly went, “Wow”! It’s clean, it’s easy to read (sometimes 😁) and helps you write better code.

However, while writing tests, I encountered certain problems.

In this article, let us resolve some of the common issues that can be faced during mocking and unit testing our features.

## Mockito Troubles: the Right Dependencies

1) In Kotlin

```
org.mockito.exceptions.base.MockitoException:
 Cannot mock/spy class com.example.authentication.login.LoginRepository
 Mockito cannot mock/spy because :
 — final class
 ```

### Reason -

When Mockito mocks an object it extends the requested class to create an instance.

However, all classes in Kotlin are final by default. The rule: “closed for modifications, open for extension” is baked into the language.

So they can’t be extended.

### Solution -

Either Replace your mockito-core gradle with the following:

```
testImplementation 'org.mockito:mockito-inline:latestVersion'
```

It is the configuration to make Mockito work with final classes.

Or

Use the Mockito extension mechanism by creating the file <b>org.mockito.plugins.MockMaker</b> in your <b>test</b> folder -

### test/resources/mockito-extensions/org.mockito.plugins.MockMaker

Write this line in the file -

<b>mock-maker-inline</b>

2) Another problem was of AndroidSchedulers -

```
java.lang.ExceptionInInitializerError at io.reactivex.android.schedulers.AndroidSchedulers$1.call(AndroidSchedulers.java:35)

at io.reactivex.android.schedulers.AndroidSchedulers$1.call(AndroidSchedulers.java:33)

at io.reactivex.android.plugins.RxAndroidPlugins.callRequireNonNull(RxAndroidPlugins.java:70)

at io.reactivex.android.plugins.RxAndroidPlugins.initMainThreadScheduler(RxAndroidPlugins.java:40)

at io.reactivex.android.schedulers.AndroidSchedulers.<clinit>(AndroidSchedulers.java:32)
```

### Reason -

Default Scheduler returned by AndroidSchedulers.mainThread() is an instance of <b>HandlerScheduler</b> which relies on Android dependencies which are not available to be instantiated on the <b>JVM</b>.

However, we want to override the default <b>AndroidSchedulers.mainThread()</b> scheduler and return an instance which does not have these Android dependencies and can be safely instantiated.

### Solution -

Use <b>RxAndroid’s RxAndroidPlugins</b> class which provides some hooks for overriding RxAndroid’s schedulers.

```
@Before
public static void setup() {
    RxAndroidPlugins.setInitMainThreadSchedulerHandler(
        __ -> Schedulers.trampoline());
}
```

3) Error while mocking Repository

```
private var loginRepository = mock<LoginRepository>()
```

After running the test, I saw the test failed as the actual repository was getting called.

### Reason -

Here you can see, I am initializing the repository inside the ViewModel class, which means when I initialize my ViewModel class in Test,

those mock repositories will get replaced with the actual implementation.

```
class LoginViewModel : ViewModel() {
    private val loginRepository: LoginRepository = LoginRepository()
    private val truecallerRepository: TruecallerRepository = TruecallerRepository()
    //other logic
    }
```

### Solution -

Pass these repositories from Factory to the ViewModel, you will always get the same instance you want to get invoked.

<b>ViewModelFactory</b>: It’s a class that implements <b>ViewModelProvider.Factory</b> and it will create the ViewModel from a parameter .class.

```
class LoginViewModelFactory constructor() : ViewModelProvider.Factory {


    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass != LoginViewModel::class.java) {
            throw IllegalArgumentException("Wrong Arguments")
        }
        val loginRepository = LoginRepository()
        val truecallerRepository = TruecallerRepository()
        return LoginViewModel(loginRepository, truecallerRepository) as T
    }
}
```

In your ViewModel -

```
class LoginViewModel(private val loginRepository: LoginRepository,
                     private val truecallerRepository: TruecallerRepository) : ViewModel() {
// other logic
}
```

In Fragment or Activity -

```
val loginViewModel = ViewModelProviders.of(this, this.viewModeFactory).get(MyViewModel::class.java)
```

4) Error while mocking LiveData Observers -

```
private val mockObserver = mock<Observer<AuthenticationState>>
```

### Solution -

When testing LiveData, [InstantTaskExecutorRule](https://developer.android.com/reference/android/arch/core/executor/testing/InstantTaskExecutorRule) is needed in addition to [RxImmediateSchedulerRule](https://stackoverflow.com/questions/43356314/android-rxjava-2-junit-test-getmainlooper-in-android-os-looper-not-mocked-runt/43356315#43356315) if the class being tested has both background thread and LiveData.

Use this with RxImmediateSchedulerRule -

```
@Rule
@JvmField
var instantExecutorRule = InstantTaskExecutorRule()
```

That's all from my learnings!!








