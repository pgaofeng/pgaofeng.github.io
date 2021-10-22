## CoordinateLayout

CoordinateLayout，又称协调布局。主要作为一个调控者，用于控制它的子View之间的依赖关系和滑动关系。

### 使用

> CoordinatorLayout is a super-powered FrameLayout

在CoordinatorLayout类的注释中，第一行就是上面这句话，即CoordinateLayout是一个超级强大的FrameLayout。也就是说，在没有其他配置的情况下(未设置anchor和Behavior等)，它的行为是和FrameLayout保持一致的，直接可以当成一个FrameLayout来使用。

当然，正常情况下肯定是不会把他来当做FrameLayout来使用的。而前面之所以说这一点，是因为对CoordinateLayout不熟而对FrameLayout很熟，因此把他们放在一起联想，就能很清楚的知道CoordinateLayout中的子View的测量过程和布局过程了。

#### Behavior

Behavior是CoordinateLayout的一个抽象的静态内部类，CoordinateLayout就是通过它来实现各种炫酷的操作，从而变得super-powered。虽然Behavior是一个抽象类，但是它并没有抽象方法，所有的方法都有默认的方法体，方便开发者仅关注他们感兴趣的地方。

Behavior只能作用在CoordinateLayout的子View上，并且一个Behavior只能对应一个子View。可以通过三种方式去给子View设置Behavior：

1，可以在xml中通过app:layout_behavior="xxx"来设置，xxx的值对应的是Behavior的完整类名，例如可以设置为com.study.androidbehavior.behavior.BelowBehavior，当然也可以使用省略包名的缩略类名如.behavior.BelowBehavior。推荐最好是使用完整的类名，因为有时候可能会出现包名不一致的情况。

2，若是没有在xml中设置Behavior，还可以通过子View的CoordinateLayout.LayoutParams#setBehavior来设置，这时候需要手动去new一个Behavior然后设置给子View。

3，也可以使用注解的方式设置，在需要设置Behavior的View类上使用注解@DefaultBehavior(BelowBehavior.class)。当然这种方式就要求必须自定义View，很不灵活，所以几乎不会使用这种方式。

用的最多的是第一种方式，也就是在xml中设置。

##### 构造Behavior

Behavior有两个构造方法，一个是空参数的构造方法public Behavior()，一个是两个参数的构造方法public Behavior(Context context, AttributeSet attrs)，默认下两个构造方法都是空实现的。

其中，空参数的构造方法一般用于在代码中设置，很方便直接new一个Behavior；而两个参数的构造方法常用于通过xml设置Behavior的方式，这种情况下，会将xml中的对应的属性传递到attrs中。因此，可以在这个方法中去获取到xml设置的属性以及自定义属性，例如下面的例子就是获取xml中设置的属性：

- 1，先在attrs.xml中定义属性名称和类型，这步和自定义view时是一样的

```xml
<resources>
    <declare-styleable name="MovableButton_Behavior">
        <attr name="sex" format="string" />
    </declare-styleable>
</resources>
```

- 2，在xml中使用这个属性

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_behavior=".behavior.AttrBehavior"
        app:sex="aaaa" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

- 3，在对应的Behavior中获取设置的属性：

```kotlin
class AttrBehavior(
    context: Context,
    attributeSet: AttributeSet
) : CoordinatorLayout.Behavior<View>(context, attributeSet) {

    init {
        context.obtainStyledAttributes(attributeSet, R.styleable.MovableButton_Behavior).use {
            val sex = it.getString(R.styleable.MovableButton_Behavior_sex)
            println("xml中设置的app:sex属性值为：$sex")
        }
    }

}
```

通过如上三步操作，即可在Behavior中获取到xml中自定义的属性了。需要注意的是，自定义的属性，必须与对应的Behavior设置在同一个view上。例如上例中的app:sex属性与AttrBehavior都设置在了ImageView上，因此AttrBehavior才能获取到这个属性值。

Behavior还有一个泛型，这个泛型是用于设置当前的Behavior可以作用在什么View上的。例如上例中泛型参数设置成了View，代表在CoordinateLayout的所有子View上都能应用这个Behavior。若是设置为ImageView，则表示只能在ImageView上设置并使用这个Behavior，设置在其他View上则会是无效的。

##### 依赖关系

Behavior主要有两种能力，处理view之间的依赖关系以及处理view之间的嵌套滑动关系。 依赖关系就是某个view依赖于另一个view，当另一个view发生变化的时候，该view也会随之发生变化。可以在Behavior中去定义这种依赖关系。

###### layoutDependsOn 确定依赖关系

```java
public boolean layoutDependsOn(
	@NonNull CoordinatorLayout parent, 
	@NonNull V child,
    @NonNull View dependency
)
```

在Behavior中，使用layoutDependsOn来确定依赖关系。这个方法有三个参数，第一个参数是父布局parent；第二个参数是设置了Behavior的那个View，被称为child；第三个参数就是依赖的view。我们需要做的就是在这个方法中去判断child与这个view是否需要存在依赖关系，若是存在依赖的话，则返回true，否则返回false。Behavior会遍历除了child外的其他view，然后将其带入这个方法去判断是否是依赖的对象。

> 在Behavior中，都是使用parent来代表父布局CoordinateLayout，child来代表设置了Behavior的子view，dependency代表依赖的view。

###### onDependentViewChanged 处理依赖行为

```java
public boolean onDependentViewChanged(
	@NonNull CoordinatorLayout parent, 
	@NonNull V child,
   	@NonNull View dependency
)
```

依赖的view发生变化后（位置发生变化或者尺寸发生变化），就会调用这个方法。因此，若是定义了依赖关系，则需要在这个方法中去处理具体的行为。这个方法还有个boolean的返回值，用于判断是否发生了依赖行为。也就是当child发生了位置或者尺寸的变化时，表明发生了依赖行为，此时应该返回true。

注意，依赖是可以存在多个依赖的。也就是一个child可以依赖多个dependency，这个关系是layoutDependsOn方法所确定的，当它返回true的时候就是确定了依赖关系。因此不论是哪个依赖的view发生变化，都会触发onDependentViewChanged的调用。

###### onDependentViewRemoved 移除依赖

```java
public void onDependentViewRemoved(
	@NonNull CoordinatorLayout parent, 
	@NonNull V child,
    @NonNull View dependency
)
```

onDependentViewRemoved 方法是在被依赖的view从父布局中移除的时候调用，该方法用于当依赖消失的时候（被依赖的view从父布局中移除，也就是关于它的依赖关系消失了）child的行为。



###### 小例子

下面用一个小例子说明这种依赖关系，在CoordinatorLayout中定义了两个view，其中一个view依赖于另一个view，并且一直处于这个view的下方。

- 1，首先定义一个可移动的按钮MovableButton作为被依赖的View，因为依赖的条件是被依赖的View发生位置或者尺寸的变化，所以定义这个view是为了能满足这个要求。

  ```kotlin
  // 一个简单view，可以跟随手指而动
  class MovableButton @JvmOverloads constructor(
      context: Context,
      attributeSet: AttributeSet? = null
  ) : AppCompatButton(context, attributeSet) {
  
      private var mInitX = 0F
      private var mInitY = 0F
      private var mEventX = 0F
      private var mEventY = 0F
  
      override fun onTouchEvent(event: MotionEvent?): Boolean {
          when (event?.actionMasked) {
              MotionEvent.ACTION_DOWN -> {
                  mInitX = x
                  mInitY = y
                  mEventX = event.rawX
                  mEventY = event.rawY
              }
              MotionEvent.ACTION_MOVE -> {
                  x = mInitX + event.rawX - mEventX
                  y = mInitY + event.rawY - mEventY
              }
          }
          return super.onTouchEvent(event)
      }
  
  }
  ```

- 2，定义一个BelowBehavior，声明这种依赖关系并且处理具体的依赖行为。

  ```kotlin
  class BelowBehavior(
      context: Context,
      attributeSet: AttributeSet
  ) : CoordinatorLayout.Behavior<View>(context, attributeSet) {
  
      override fun layoutDependsOn(
          parent: CoordinatorLayout,
          child: View,
          dependency: View
      ): Boolean {
      	// 设置只有MovableButton才能作为被依赖View
          return dependency is MovableButton
      }
  
      override fun onDependentViewChanged(
          parent: CoordinatorLayout,
          child: View,
          dependency: View
      ): Boolean {
      	// 让child一直处在被依赖的View下面
          child.y = dependency.height + dependency.translationY
          return true
      }
      
       override fun onDependentViewRemoved(
          parent: CoordinatorLayout, child: View,
          dependency: View
      ) {
          // 当被依赖的view被移除的时候，将child的位置重置在界面顶部
          child.y = 0F
      }
  
  }
  ```

在BelowBehavior中，定义依赖关系的时候并没有太严格的要求，仅仅是判断依赖的view是一个MovableButton就行。实际中，这个依赖关系的确立应该更加严格一些，如本例中，若是在布局中存在多个MovableButton的话，那么就会确立多个依赖关系。不论哪个MovableButton位置发生了变化，设置了Behavior的view都会立即跑到它的下面。

- 3，在xml中使用Behavior

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:id="@+id/parent"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  
      <com.study.androidbehavior.widget.MovableButton
          android:id="@+id/movable_button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content" />
  
      <ImageView
          android:id="@+id/image"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:src="@mipmap/bg"
          app:layout_behavior=".behavior.BelowBehavior" />
      
      <Button
          android:id="@+id/button_remove"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_gravity="end"
          android:text="移除按钮" />
  
  </androidx.coordinatorlayout.widget.CoordinatorLayout>
  ```

在xml中，CoordinatorLayout一共有三个子view，并且给ImageView设置BelowBehavior。那么这种情况下，就是ImageView会一直跟在MovableButton的下面，MovableButton的位置发生变化的时候，ImageView也会跟着一起变，总之就是一直在它的下面。

- 4，在MainActivity中设置点击按钮移除，用于触发onDependentViewRemoved事件

  ```kotlin
  class MainActivity : AppCompatActivity() {
  
      private lateinit var mParent: CoordinatorLayout
      private lateinit var mButtonMovable: MovableButton
      private lateinit var mButtonRemove: Button
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
  
          mParent = findViewById(R.id.parent)
          mButtonMovable = findViewById(R.id.movable_button)
          mButtonRemove = findViewById(R.id.button_remove)
          mButtonRemove.setOnClickListener {
              mParent.removeView(mButtonMovable)
          }
      }
  }
  ```
  
  在MainActivity中给被依赖的view添加了一个点击事件，当点击这个按钮的时候，就会将可移动的按钮从父布局中移除，此时会触发依赖移除的方法，ImageView就会被移动到界面顶部。

![Behavior的依赖行为](../images/behavior-dependency.gif)



##### 自定义布局

前面有说过，CoordinateLayout可以当成一个FrameLayout来使用，它的测量和布局过程都是和FrameLayout一样的。但是，这只是它默认的行为，我们实际上可以通过Behavior来实现自定义的测量和布局过程。

###### onLayoutChild

```java
boolean onLayoutChild(
	@NonNull CoordinatorLayout parent, 
	@NonNull V child,
    int layoutDirection
)
```

在Behavior中，通过onLayoutChild方法去自定义布局。他有一个返回值，处理完自定义布局后，需要返回true来标识确定要自定义布局，否则是无效的。在这个方法中，我们应该只去对child进行布局，而不应该对所有的view进行布局。第三个参数layoutDirection我们基本上不用去关注，它是用于判断布局的方向的，从左到右或者从右到左，因为大部分的国家的书写顺序都是从左到右的。

若是在onDependentViewChanged中若是修改了view的位置(layout)，则应该重写onLayoutchild方法，去保证每个状态的位置的稳定。

###### onMeasureChild

```java
boolean onMeasureChild(
	@NonNull CoordinatorLayout parent, 
	@NonNull V child,
    int parentWidthMeasureSpec,
    int widthUsed,
    int parentHeightMeasureSpec, 
    int heightUsed
)
```

与之类似，也可以使用这个方法去对child进行自定义的测量。在没有设置Behavior或者Behavior中没有去干涉测量和布局的情况下，默认的测量布局过程都是与FrameLayout一致的。因此可以根据需求去决定是否需要自定义这个过程。

实际上，自定义测量过程实际用的并不多，毕竟默认的测量过程基本上已经满足我们的需求了，更多用的是布局过程。默认的布局过程是像FrameLayout那样全部堆在左上角的，因此需要我们重写布局过程去实现自己想要的效果。

###### 小例子

定义一个Behavior，重写他的onLayoutChild方法，将其child设置到MovableButton的右侧。

```kotlin
class LayoutBehavior(
    context: Context,
    attr: AttributeSet
) : CoordinatorLayout.Behavior<View>(context, attr) {

    override fun onLayoutChild(
        parent: CoordinatorLayout,
        child: View,
        layoutDirection: Int
    ): Boolean {
        // 去查找是否有MovableButton
        var target: MovableButton? = null
        for (i in 0 until parent.childCount) {
            if (parent.getChildAt(i) is MovableButton) {
                target = parent.getChildAt(i) as MovableButton
                break
            }
        }
        target ?: return false
        // 将child放置在movableButton的右侧
        child.layout(
            target.right,
            target.top,
            target.right + child.measuredWidth,
            target.top + child.measuredHeight
        )
        return true
    }

}
```

注意的是onLayoutChild只是接管了parent对child的布局过程，也就是说只有父View去layout的时候才会调用到这个方法。因此上面的例子虽然实现了在布局的时候将child置于MovableButton的右侧，但当MovableButton移动的时候，child并不会随之而动的。



##### 拦截事件

发生在CoordinateLayot中的事件都可以被Behavior所拦截，拦截方法也是和正常的事件拦截机制是一样的。首先通过onInterceptTouchEvent来判断是否需要拦截，是的话返回true，此时所有的事件都会分发到onTouch方法中。

##### 嵌套滑动

嵌套滑动指的是多个view嵌套在一起，并且都可以滚动。这时候当发生滚动的时候，就会出现问题，最主要的就是滑动不连贯的问题。

因为在传统的View的滑动中，当一个view决定要消耗滑动事件的时候，这个系列的滑动事件（Down-Move-Move... -Up/Cancel）都会交给它。那么当它已经滑动到底部不能再滑动的时候，剩下的事件相当于是浪费了。

而嵌套滑动主要就是解决的这个问题，可以将一套事件分发给多个view，让每个view都有机会参与这个滑动过程。例如内部的view滑动到底部后，剩下的事件就交给外部view去进行滑动，从而实现一种滑动的连续性。

当然Behavior只是利用了View的嵌套滑动机制而已，当发生了嵌套滑动事件的时候，会先将事件依次分发给Behavior，然后才是滑动对象。

###### onStartNestedScroll

```java
public boolean onStartNestedScroll(
    @NonNull CoordinatorLayout coordinatorLayout,
    @NonNull V child, 
    @NonNull View directTargetChild, 
    @NonNull View target,
    @ScrollAxis int axes, 
    @NestedScrollType int type
)
```

这是嵌套滑动开始的方法，当发生嵌套滑动的时候，会先调用这个方法，判断Behavior是否需要参与此次的滑动，返回true表示参与这个活动，才会有后续的方法调用，否则后续的事件都不会再回调到这个Behavior中。

其中参数coordinatorLayout表示父布局，child表示设置了Behavior的子View。directTargetChild代表滑动view在CoordinateLayout中直接子布局，target代表滑动的子布局。axes表示滑动的方向ViewCompat#SCROLL_AXIS_HORIZONTAL和ViewCompat#SCROLL_AXIS_VERTICAL。type表示滑动的类型ViewCompat#TYPE_TOUCH和ViewCompat#TYPE_NON_TOUCH。

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".view.TouchActivity">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
            tools:itemCount="10"
            tools:listitem="@layout/item_recycler" />

    </FrameLayout>

    ...

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

如上这个布局文件，在RecyclerView发生滑动的时候，directTargetChild就是FrameLayout，而target就是RecyclerView。

###### onNestedScrollAccepted

```java
public void onNestedScrollAccepted(
	@NonNull CoordinatorLayout coordinatorLayout,
    @NonNull V child, 
    @NonNull View directTargetChild, 
    @NonNull View target,
    @ScrollAxis int axes, 
    @NestedScrollType int type
)
```

onNestedScrollAccepted方法是在接收滚动事件后触发的，也就是onStartNestedScroll返回true后就会触发这个方法。可以将它看做是对onStartNestedScroll的补充，也就是开发者需要在onStartNestedScroll中判断是否要参与此次的滑动，而在决定参与后在onNestedScrollAccepted中去做滑动的前置准备，比如初始化状态等。

###### onNestedPreScroll和onNestedScroll

```java
public void onNestedPreScroll(
	@NonNull CoordinatorLayout coordinatorLayout,
    @NonNull V child, 
    @NonNull View target, 
    int dx, 
    int dy, 
    @NonNull int[] consumed,
    @NestedScrollType int type
)

public void onNestedScroll(
	@NonNull CoordinatorLayout coordinatorLayout, 
	@NonNull V child,
    @NonNull View target, 
    int dxConsumed, 
    int dyConsumed, 
    int dxUnconsumed,
    int dyUnconsumed, 
    @NestedScrollType int type, 
    @NonNull int[] consumed
)
```

onNestedPreScroll和onNestedScroll也是一组的，他们会在滚动的时候持续被调用，并且是成对出现的。

当发生滑动事件的时候，会先调用onNestedPreScroll方法，这个方法是用于处理滑动前的操作的，也就是在滑动前会将滑动的距离(参数dx和dy)传递进来，Behavior可以通过这个方法来提前消耗这些距离，消耗的距离需要放在参数consumed中(参数数组，consumed[0]是dx的消耗，consumed[1]是dy上的消耗)。例如本来RecyclerView要滑动100的距离，然后Behavior在这个方法中选择消耗了20的距离，那么实际传递到RecyclerView的距离则只有80。

在onNestedPreScroll之后，滑动view会处理剩下的滑动，当滑动view处理完后，会将剩下的滑动再通过onNestedScroll传递过来，然后Behavior就可以在这个方法中继续处理滑动事件了。

总结一下过程就是**子view发生了滑动事件->先传递到Behavior的onNestedPreScroll进行消耗->Behavior消耗剩下的事件才传给子view消耗->子view消耗后再剩下的事件传给Behavior的onNestedScroll消耗**。



###### onStopNestedScroll

这个方法是一个系列的事件发生结束后调用的，可以用于处理一些收尾工作。



###### 补充

在传统的嵌套滑动中，是通过NestedScrollingParent3和NestedScrollingChild3来将父View与子View进行区分的。要实现嵌套滑动，必须存在两个view，一个是外层的parent，一个是内部的child。

CoordinateLayout中的嵌套滑动也是通过传统的嵌套滑动实现的，其中CoordinateLayout实现了NestedScrollingParent3接口，它是可以作为parent的。因此若要实现嵌套滑动，触发滑动的view必须实现NestedScrollingChild3接口。其中NestedScrollView和RecyclerView都是实现了Child接口的。

而CoordinateLayout并没有去作为parent去处理嵌套滑动逻辑，而是将其都转发给了Behavior，可以看到Behavior的这些方法和NestedScrollingParent3中的方法基本上是一致的。所以设置了Behavior的子view都可以看做parent。

###### 示例

写一个示例来应用嵌套滑动的功能：
