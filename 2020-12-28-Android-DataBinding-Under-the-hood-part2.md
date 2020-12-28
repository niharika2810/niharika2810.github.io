<div style="text-align:center">
<h1> Android Data Binding: Under the Hood (Part 2)
</h1>
<h3> Let's talk about the magic happening behind the dynamic data updates and click actions when we use data binding for our views
</h3>
Blog by <a href="http://thedroidlady.com/">theDroidLady</a>
</div>
<br/>
<div style="text-align:center">
<img align="center" width="500" height="300" src="/Images/Article/data_binding_two.jpg">
</div>
<br/>
<br/>

# Introduction

Hola folks 👋 , I’m back with part 2 of learning & understanding how Data Binding works behind the scene.

In the [previous blog](https://proandroiddev.com/android-data-binding-under-the-hood-part-1-33b8c7adfb7c), we saw different alternatives to [findViewById](https://stackoverflow.com/questions/60141973/what-does-findviewbyid-means-in-android-studio), their pros & cons and how [DataBinding](https://developer.android.com/topic/libraries/data-binding) works under the hood with static data. If you haven’t read it yet, I would recommend to quickly go, read and come back to know more.

In this article, let's talk about the magic happening behind the background data updates and click actions.

Well, I am sure if you are reading this article, you have already set your mind to know the magic, implementation and the players behind the scene. So let’s get started and deep dive into these aspects.

<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/happy_emoji.gif">
</div>

<br/>
<br/>
## Click Events & Data Update

Let's say I have a textView and a button in my XML. On button click, I want to update my text. Here comes the role of [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) (If you are working with [MVVM](https://www.raywenderlich.com/34-design-patterns-by-tutorials-mvvm#:~:text=Model%2DView%2DViewModel%20(MVVM,re%20typically%20subclasses%20of%20UIView%20), you are already aware of it.), where we write all our business logic. This is my XML:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="viewmodel"
            type="com.example.myapplication.MainViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/count_textview"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{Integer.toString(viewmodel.count)}"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            tools:text="0" />

        <Button
            android:id="@+id/action_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="30dp"
            android:onClick="@{() -> viewmodel.onIncrement()}"
            android:text="Increment"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/count_textview" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

And, This is my MainViewModel,
 
```
class MainViewModel : ViewModel() {

    private val _count = MutableLiveData<Int>()

    val count: LiveData<Int> = _count

    fun onIncrement() {
        _count.value = (_count.value ?: 0) + 1
    }
}
```

I updated my MainActivity to use ViewModel,

```
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: MainViewModel
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding: ActivityMainBinding =
            DataBindingUtil.setContentView(this, R.layout.activity_main)
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
        binding.lifecycleOwner = this
        binding.viewmodel = viewModel
    }
}
```

>Note: To use a [LiveData](https://developer.android.com/topic/libraries/architecture/livedata) object with your binding class, you need to specify a lifecycle owner to define the scope of the LiveData object.

Now, Whenever a user clicks the button, the count is incremented and as this count is a LiveData and has been observed in XML by the `count_textView`, you will always see the latest updated value. 

## Wow, So Easy it is. But, How exactly is this happening? How does DataBinding class know that this view needs to be updated?

Well, We have the answer,

In our ActivityMainBinding class, you will find this 

```
 protected ActivityMainBinding(Object _bindingComponent, View _root, int _localFieldCount,
      Button actionButton, TextView countTextview) {
    super(_bindingComponent, _root, _localFieldCount);
    this.actionButton = actionButton;
    this.countTextview = countTextview;
  }
```

You can see a field named `_localFieldCount`. This parameter now holds a value of 1 because now the XML is having one view which is observing for any changes.

> In the case of static data, where we just rendered the model values, this `_localFieldCount` was 0 as no view was observing for any changes. (Check the previous article for reference.) 

And this class ActivityMainBinding extends [ViewDataBinding](https://android.googlesource.com/platform/frameworks/data-binding/+/refs/heads/mirror-goog-studio-master-dev/extensions/library/src/main/java/androidx/databinding/ViewDataBinding.java), so it calls the Parent class constructor, where we have a WeakListener array which is nothing but an array of Weak reference objects which do not prevent their referents from being made finalizable, finalized, and then reclaimed. The best part is we don’t need to worry about their cleanups. :-)

```
 /**
 * The observed expressions.
 */
 private WeakListener[] mLocalFieldObservers;

protected ViewDataBinding(DataBindingComponent bindingComponent, View root, int localFieldCount) {
        mBindingComponent = bindingComponent;
        mLocalFieldObservers = new WeakListener[localFieldCount];
        ........
}
```

So whenever an activity sets a [lifecycle owner](https://developer.android.com/reference/android/arch/lifecycle/LifecycleOwner), which is in our case is Activity Scope, ViewDataBinding class registers for a listener i.e onStartListener. Whenever the activity comes in onStart state, binding class starts executing the bindings.

```
@OnLifecycleEvent(Lifecycle.Event.ON_START)
public void onStart() {
    ViewDataBinding dataBinding = mBinding.get();
    if (dataBinding != null) {
        dataBinding.executePendingBindings();
    }
}
```

In our ActivityMainBindingImpl, we have our executeBindings, which takes the count value from ViewModel and passes to updateLiveDataRegistration method. 

```
  @Override
    protected void executeBindings() {
    ......
    
     if (viewmodel != null) {
        // read viewmodel.count
        viewmodelCount = viewmodel.getCount();
     }
     updateLiveDataRegistration(0, viewmodelCount);
     ........
  } 
 ```

This <b>updateLiveDataRegistration</b> asks to create a <b>weak listener</b> for the livedata variable we passed and then start listening for the updates. (This internally sets the same lifecycleOwner for the liveData as well) This localFieldId is set to 0 in this case.

```
   protected boolean updateLiveDataRegistration(int localFieldId, LiveData<?> observable) {
        mInLiveDataRegisterObserver = true;
        try {
            return updateRegistration(localFieldId, observable, CREATE_LIVE_DATA_LISTENER);
        } finally {
            mInLiveDataRegisterObserver = false;
        }
    }
```

Now when the user clicks the button, onChanged() method from our LiveDataListener gets called,

```
@Override
public void onChanged(@Nullable Object o) {
    ViewDataBinding binder = mListener.getBinder();
    if (binder != null) {
        binder.handleFieldChange(mListener.mLocalFieldId, mListener.getTarget(), 0);
    }
}
```

This eventually calls our own implementation of `onFieldChange()`, which looks like this, 

```
@Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
            case 0 :
                return onChangeViewmodelCount((androidx.lifecycle.LiveData<java.lang.Integer>) object, fieldId);
        }
        return false;
  }
```

Inside this onChangeViewModelCount() method, we have `mDirtyFlags`(a long value) which is getting updated and it will request for rebinding which will call execute pending bindings and get views updated on the screen.

```
synchronized(this) {
        mDirtyFlags |= 0x1L;
}
```

>This process keeps on repeating every time we click our button.
 
So this is how the click events and updates are handled by DataBinding class. 

For reference, you can find my DataBinding sample [here](https://github.com/niharika2810/DataBindingSample)

This is published on [ProAndroidDev](https://proandroiddev.com/android-data-binding-under-the-hood-part-2-fdcbb0f54700)

Enjoy reading!!
 