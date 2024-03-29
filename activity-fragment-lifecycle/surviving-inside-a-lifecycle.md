# Surviving Inside a Lifecycle

## Saving state

`Activity` is very likely to be terminated, almost by anything. Today's phones have usually locked rotation of the screen, but still, screen rotation is one of the external factors leading to `Activity's` "death".

#### What to save?

* data that are hard to recover (time or resource wise)
  * i.e. long running calculations
* bigger collections of data, that are valid for longer periods of time
  * this is usually a good case for using a DB

### State changes

Notice that the transition from portrait mode to landscape cleared the state of UI elements.

&#x20;![Rotace displeje - vyplněná data](../.gitbook/assets/4-screen-rotation-1.png) ![Rotace displeje - chybějící data](../.gitbook/assets/4-screen-rotation-2.png)

#### How not to handle screen rotation

* ignore it
* force portrait/landscape mode only (what do you think happens on the phone below)
* declare `android:configChanges="orientation"` in `AndroidManifest.xml` and then ignore any manual handling of the changes

![HW keyboard](../.gitbook/assets/4-kyocera-rise.jpg)

{% hint style="info" %}
Setting`android:configChanges="orientation",`means that for some reason, you want to handle changes in the UI yourself in[`onConfigurationChanged()`](https://developer.android.com/reference/android/app/Activity#onConfigurationChanged\(android.content.res.Configuration\)) callback. It's not meant for dealing with state loss.
{% endhint %}

### Saving View state

System UI elements handle state persistence automatically. It's necessary to assign `View` an ID. Assigned **ID must be unique** in given context. Using duplicate IDs results in incorrect state restoration.

![View state saving](../.gitbook/assets/4-save-state.gif)

### Saving Object state

Java way of serialising object state is usually using `Serializable` interface. Android uses it's very own `Parcelable` implementation.

#### Parcelable

[`Parcelable`](http://developer.android.com/reference/android/os/Parcelable.html) is very fast. It uses binary data and native code implementation to provide the fast response times. Object properties are serialised into a Parcel, where order in which the individual values are written and read is important. It can handle storing of primitive data types and Kotlin's equivalents + String + onother `Objects` implementing `Parcelable`.

{% hint style="info" %}
Don't implement `Parcelables` manually. Use `kotlin-parcelize` plugin to handle it for you. The only thing you need is to annotate your class with `@Parcelize`.
{% endhint %}

#### **Bundle**

`Bundle` is a special kind of `Parcelable`, which uses `Map` like interface. It is used across the system to serialise state.

### Saving Activity state

Saving state of an `Activity` can be done using `onSaveInstanceState` function. It provides a `Bundle` to be filled with values. This `Bundle` can be later retrieved either in `onCreate` or in `onRestoreInstanceState` callback. Today, the recommended way to retain data is to do so in a `ViewModel`.

Note that, when an `Activity` is created for the first time, `Bundle` passed as `onCreate` functions's argument is null, in subsequent calls it is non null, even though it was never used to persist data.

![Bundle state](<../.gitbook/assets/4-activity-state (1).png>)

{% tabs %}
{% tab title="Kotlin" %}
```kotlin
class MainActivity : AppCompatActivity() {
    private val STATE_OUT = "state-out"

    private var someData: Any? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (savedInstanceState != null) {  // Activity is recreated
            someData = savedInstanceState.getParcelable(STATE_OUT)
        }
    }

    // Guaranteed to be called before destroying the Activity
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putParcelable(STATE_OUT, someData)
    }
} 
```
{% endtab %}

{% tab title="Java" %}
```java
public class SaveStateActivity extends AppCompatActivity {

    private static final String STATE_OUT = "state-out";

    private Object someData;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (savedInstanceState != null) {  // Activity is recreated
            someData = savedInstanceState.gerParcelable(STATE_OUT);
        }
    }

    // Guaranteed to be called before destroying the Activity
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putParcelable(STATE_OUT, someData);
    }
}
```
{% endtab %}
{% endtabs %}

### Saving Fragment state

Saving `Fragment` state is very similar to the way `Activities` handle it. The only thing one must watch for is the implementation of `onSaveInstanceState`, which must be implemented in both `Fragment` and its parent `Activity` as well. And like for `Activities` the recommended way to retain data is to do so in a `ViewModel`.

{% hint style="warning" %}
Don't use [retained](https://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html) `Fragment` for persisting state. Even if you use the headless version. It's won't recover from all states.
{% endhint %}

## How can you tell, you are doing it right?

Navigate to _Developer options_, and here activate the option: _Don't keep Activities_. This way the system will destroy any activity you leave invisible. This behavior is hard to encounter when using a high-end device.&#x20;
