# Layout and Views

## View Types

### View

View is a single UI component which represents the basic building block for user interface components. Its behavior and look is defined programatically by extending a [View](https://developer.android.com/reference/android/view/View) class.

#### Widget

Widget is a special type of a View, which main purpose is not only to display information, but also to provide additional functionality. Typical widget is a [VideoView](https://developer.android.com/reference/android/widget/VideoView).

### ViewGroup

ViewGroup is a subclass, that is the base class for _layouts_, which are invisible containers that hold other Views \(or other ViewGroups\) and define their layout properties. Views inside a ViewGroup are called Child Views.

## Layout Types

### RelativeLayout

[`RelativeLayout`](https://developer.android.com/guide/topics/ui/layout/relative) is a view group that displays child views in relative positions. The position of each view can be specified as relative to sibling elements \(such as to the left-of or below another view\) or in positions relative to the parent [`RelativeLayout`](https://developer.android.com/reference/android/widget/RelativeLayout) area \(such as aligned to the bottom, left or center\).

{% tabs %}
{% tab title="Code" %}
```markup
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <EditText
        android:id="@+id/title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Title"/>

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/title"
        android:layout_alignParentRight="true"
        android:text="Button" />

</RelativeLayout>
```
{% endtab %}

{% tab title="Visual" %}
![](../.gitbook/assets/relativelayout.png)
{% endtab %}
{% endtabs %}

### LinearLayout

[`LinearLayout`](https://developer.android.com/guide/topics/ui/layout/linear) is a view group that aligns all children in a single direction, vertically or horizontally. You can specify the layout direction with the [`android:orientation`](https://developer.android.com/reference/android/widget/LinearLayout#attr_android:orientation) attribute.

{% tabs %}
{% tab title="Code" %}
```markup
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button 1" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button 2" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button 3" />

</LinearLayout>
```
{% endtab %}

{% tab title="Visual" %}
![](../.gitbook/assets/linearlayout.png)
{% endtab %}
{% endtabs %}

### FrameLayout

[`FrameLayout`](https://developer.android.com/reference/android/widget/FrameLayout) is designed to block out an area on the screen to display a single item. Generally, `FrameLayout` should be used to hold a single child view, because it can be difficult to organize child views in a way that's scalable to different screen sizes without the children overlapping each other. You can, however, add multiple children to a `FrameLayout` and control their position within the `FrameLayout` by assigning gravity to each child, using the [`android:layout_gravity`](https://developer.android.com/reference/android/widget/FrameLayout.LayoutParams#attr_android:layout_gravity) attribute.

Child views are drawn in a stack, with the most recently added child on top.

{% tabs %}
{% tab title="Code" %}
```markup
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Hello World!" />

</FrameLayout>
```
{% endtab %}

{% tab title="Visual" %}
![](../.gitbook/assets/framelayout.png)
{% endtab %}
{% endtabs %}

### ConstraintLayout

Available since Android Studio v 2.2. It allows creation of flat view hierarchy, which can improve view rendering latencies. It was created to replace RelativeLayout or LinearLayout using wight attributes. These both group layouts have computationally expensive onMeasure phase, which can be simplified by mathematical expressions in ConstraintLayout. This way the dimensions can be computed with constant complexity. It's being shipped as independent support library, so it can be used regardless the version on OS.

![Constraint layout](../.gitbook/assets/2-constraint-layout.png)

### CoordinatorLayout

Enhanced FrameLayout. It allows for creation of UI with animations. CoordinatorLayout's child views can be animated together based on defined behaviors from state A to state B. It's one of the fundamental parts of the Material Design language.

![CoordinatorLayout](../.gitbook/assets/2-coordinator-layout.gif)

## Gotchas

### Don't use RelativeLayout as a parent view of a complex view structure

RelativeLayout is suitable for simple view hierarchy only. Complex structure leads to slow rendering and can introduce a jank.

RelativeLayout uses 2 passes in its onMeasure phase. First phase computes size of its child views, the second pass sets its own dimensions.

### Efficient Layout Inflation

Using the following layout:

```markup
<ViewGroup android:id="@+id/root">
    <View android:id="@+id/leaf" />
    <ViewGroup android:id="@+id/inner_group">
        <View android:id="@+id/inner_leaf" />
    </ViewGroup>
</ViewGroup>
```

**What's the difference between** _**A**_ **and** _**B**_**?**

A\)

```kotlin
val vg = findViewById<View>(R.id.inner_group) as ViewGroup
val v: View = findViewById(R.id.inner_leaf)
```

B\)

```kotlin
val vg = findViewById<View>(R.id.inner_group) as ViewGroup
val v = vg.findViewById<View>(R.id.inner_leaf)
```

Calling `findViewById()` on the root view results in subsequent calls of this function on all of its children, or until the ID is found. Example A finds the ID following these steps:

1. root-&gt;leaf  
2. root-&gt;inner\_group  
3. inner\_group-&gt;inner\_leaf

Example B has just one step:

1. inner\_group-&gt;inner\_leaf

### Minimising Overdraw

Placing views on top of each other \(usually when using FrameLayout\) introduces overdraw. The UI is rendered in layers, and if you have views overlapping each other, some pixels might be overdrawn multiple times, before the whole layout is finally rendered.

See the the image below. The background o window is rendered first, then the background of a ToolBar, then the wide image and finally the poster image on top of it. Pixels in the top poster area were overdrawn as much as 4 times!

![Overdraw Example](../.gitbook/assets/2-overdraw1.png)

Overdrawing existing pixels is a redundant operation and slows down the whole process of rendering, because we compute an information which is then discarded.

{% hint style="info" %}
You can activate overdraw visualisation in Developer options.
{% endhint %}

![Activate Overdraw Debugging](../.gitbook/assets/2-overdraw0.png)

Overdraw can be introduced just by applying a redundant background color:

 ![Overdraw example 2](../.gitbook/assets/2-overdraw2.png) ![Overdraw example 3](../.gitbook/assets/2-overdraw3.png)

