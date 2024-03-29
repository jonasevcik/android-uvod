# Testing

Testing is a vital part of development process. Their purpose is not just to verify your code's correctness, but also give you means of validation when you change your code. When refactoring a code base, tests give you confidence, that you didn't break the logic of your code, you just change its structure.

There are also approaches like **TDD** - test driven development - that place tests on the first place. That's not a negligible factor. Making your code testable involves structuring it the way, you can easily mock classes, pass dependencies - to be substituted during the time of testing.

## Mocking

The term _mocking_ refers to replacing an object with a different one. Based on the use case, we distinguish [following types](https://stackoverflow.com/questions/3459287/whats-the-difference-between-a-mock-stub):

*   **Dummy** - just bogus values to satisfy the `API`.

    > _Example_: If you're testing a method of a class which requires many mandatory parameters in a constructor which _have no effect_ on your test, then you may create dummy objects for the purpose of creating new instances of a class.
*   **Fake** - create a test implementation of a class which may have a dependency on some external infrastructure. (It's good practice that your unit test does **NOT** actually interact with external infrastructure.)

    > _Example_: Create fake implementation for accessing a database, replace it with `in-memory` collection.
*   **Stub** - override methods to return hard-coded values, also referred to as `state-based`.

    > _Example_: Your test class depends on a method `Calculate()` taking 5 minutes to complete. Rather than wait for 5 minutes you can replace its real implementation with stub that returns hard-coded values; taking only a small fraction of the time.
*   **Mock** - very similar to `Stub` but `interaction-based` rather than state-based. This means you don't expect from `Mock` to return some value, but to assume that specific order of method calls are made.

    > Example: You're testing a user registration class. After calling `Save`, it should call `SendConfirmationEmail`.

## Tests in Android

We distinguish 2 types of tests in Android. They are separated in the project structure under `test` and `androidTest` directories.

![Test Structure](.gitbook/assets/10-structure.png)

### Unit Testing

Unit tests test basic code functionality. They ran on JVM, so they rely solely on Java SDK. Any Android specific classes or functionality must be omitted or mocked. Their advantage is that they can run without an Android device, so they can run locally, thus being super fast to execute. They are located under `test` directory.

```kotlin
class ExampleUnitTest {
    @Test
    fun addition_isCorrect() {
        assertEquals(4, 2 + 2)
    }
}
```

### Instrumentation Testing

Instrumentation tests are usually integration or end to end tests. They use Android APIs, thus must be executed on an Android device - being that a real device or an emulator. They rely on a runner - component that handles context in which the test will be ran.

Instrumentation testing is valuable, since it mimics real world usage of your application. Unfortunately, due to the fact it requires Android runtime for execution, it's also slower than unit testing.

```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        // Context of the app under test.
        val appContext = InstrumentationRegistry.getInstrumentation().targetContext
        assertEquals("com.example.app", appContext.packageName)
    }
}
```

## Start Testing

### Dependencies

Writing tests usually requires a different set of dependencies, then it's needed for application runtime, and you don't need these dependencies to be present in you production code. Therefore, you can specify, which dependencies are `test` or `androidTest` specific in the following manner:

```groovy
// basic framework for unit testing in Java
testImplementation 'junit:junit:4.13.2'
// framework for easy mocking
testImplementation 'org.mockito:mockito-inline:3.8.0'
testImplementation 'androidx.arch.core:core-testing:2.1.0'

androidTestImplementation 'androidx.test:core-ktx:1.4.0'
androidTestImplementation 'androidx.test:core:1.4.0'
androidTestImplementation 'androidx.test:runner:1.4.0'
androidTestImplementation 'androidx.test:rules:1.4.0'
// framework for UI testing
androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
```

###  Mocking

Mocking is usually done (and recommended to be done) using Mockito library. In the following example, you can see creation of a stub of `Context` class. Note the usage of `RunWith(MockitoJUnitRunner::class)`, which provides initialization of objects annotated with `@Mock`.

```kotlin
class UnitTestSample {

    @Mock
    private lateinit var mockContext: Context

    @Test
    fun readStringFromContext_LocalizedString() {
        // Given a mocked Context injected into the object under test...
        `when`(mockContext.getString(R.string.hello_word))
                .thenReturn(FAKE_STRING)
        val myObjectUnderTest = ClassUnderTest(mockContext)

        // ...when the string is returned from the object under test...
        val result: String = myObjectUnderTest.getHelloWorldString()

        // ...then the result should be the expected one.
        assertThat(result, `is`(FAKE_STRING))
    }
}
```

### Testing LiveData

When trying to use LiveData inside unit tests, you end up with:

```bash
java.lang.RuntimeException: Method getMainLooper in android.os.Looper not mocked.
```

This happens because LiveData enforce their code to be executed on the Main Thread. Android (and not only Android) has a concept of one main thread. It runs operations related to UI and this way it ensures actions performed on it are ordered.

JVM doesn't understand this concept and more importantly, doesn't include objects related to Android's Main Thread. This gets more complicated with calling `livedata.postValue()`. Posting a value is not synchronized and can happen in random order, so that it can be executed after you test's asserts.

Thankfully, the following rule solves the two problems.

1. the code is not executed on the Main Thread
2. any value posting happens immediately

```kotlin
@get:Rule
val rule = InstantTaskExecutorRule()
```

{% hint style="info" %}
**Note:** Rules are applied to every test case.
{% endhint %}

### Test Activities

In instrumentation testing, you need to control the context you run your test in. This includes invoking the right `Activity`. This can be achieved using `ActivityScenario`. By using this class, you can place your `Activity` in states that simulate the lifecycle events, like Activity recreation. You can use it like this:

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
    }
}
```

Or if you want it to be executed for every test case, use it with a rule, like so:

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @get:Rule var activityScenarioRule = activityScenarioRule<MyActivity>()

    @Test fun testEvent() {
        val scenario = activityScenarioRule.scenario
    }
}
```

### Test UI

UI testing is don by Espresso APIs. Espresso provides framework to work with views and UI, find and match views, perform actions over views and make assertions over views.

```kotlin
// withId(R.id.my_view) is a ViewMatcher
// click() is a ViewAction
// matches(isDisplayed()) is a ViewAssertion
onView(withId(R.id.my_view))
    .perform(click())
    .check(matches(isDisplayed()))
```

#### Test Annotations

You may encounter `@SmallTest`, `@MediumTest`, `@LargeTest` annotations for test classes. Their meaning is summarized in the figure below:

![Test size annotations](.gitbook/assets/10-annotations.png)

#### Getting Data From Views

```kotlin
private fun getText(matcher: Matcher<View?>?): String? {
    val stringHolder = arrayOf<String?>(null)
    onView(matcher).perform(object : ViewAction {
        override fun getConstraints(): Matcher<View> {
            return isAssignableFrom(TextView::class.java)
        }
        override fun getDescription(): String {
            return "getting text from a TextView"
        }

        override fun perform(uiController: UiController?, view: View) {
            val tv = view as TextView //Save, because of check in getConstraints()
            stringHolder[0] = tv.text.toString()
        }
    })
    return stringHolder[0]
}
```
