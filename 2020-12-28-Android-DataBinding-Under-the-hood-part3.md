<div style="text-align:center">
<h1> Android Data Binding: Under the Hood (Part 3)
</h1>
<h3> Let's talk about the magic happening behind the dynamic data updates and click actions when we use data binding for our views
</h3>
Blog by <a href="http://thedroidlady.com/">theDroidLady</a>
</div>
<br/>
<div style="text-align:center">
<img align="center" width="500" height="300" src="/Images/Article/data_binding_three.png">
</div>
<br/>
<br/>

# Introduction

Hola folks 👋, I am here with my final part of DataBinding internals series. If you haven't read the previous parts, I would recommend to please go and read them to get a better understanding.

In the previous parts, we covered how [static](https://proandroiddev.com/android-data-binding-under-the-hood-part-1-33b8c7adfb7c) and [dynamic](https://proandroiddev.com/android-data-binding-under-the-hood-part-2-fdcbb0f54700) data rendering is happening in XML through DataBinding & also took a deep dive into click actions.

In this article, we will talk about the usage & magic of [BindingAdapter](https://developer.android.com/reference/android/databinding/BindingAdapter). I am sure many of us have been using it for a while now and if you haven't tried it, please do check it out. It will surely make your life easy. :-)

## BindingAdapter

Sometimes, we want to do something more complex than simply calling a setter on the View like <b>loading images</b> off the UI thread, setting <b>custom text</b> based on some logic. What if all this can be possible through XML only in just a few lines?

# This seems damn easy and cool. Isn't it?? But, How can we acheive all this????


<div style="text-align:center">
<img align="center" width="500" height="300" src="/Images/Article/data_binding_three_one.gif">
</div>

<br/>
<br/>

### Definition:

> The Data Binding Library allows you to specify the method called to set a value, provide your own binding logic, and specify the type of the returned object by using Binding adapters. Like calling your own implementation of `setText()` or `setImage()` methods etc.You'd better choose a name for your attribute that is not already used by the framework.

Now, from above we understand that we can create our own method implementations and call in our XML the same way we do after referencing the view programmatically, but now in an easy way by just calling the method using an attribute.

So, Let's use it and create our own `setText()` like method using BindingAdapter.


```
object DataBindingAdapter {
    @JvmStatic
    @BindingAdapter("countText")
    fun TextView.countText(count: Int) {
        text = if (count < 3) {
            "Updated to $count"
        } else {
            "Updating more, Now it is $count"
        }
    }
}
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

<br/>
<br/>

In Kotlin, you can write your binding adapters as [extension functions](https://www.programiz.com/kotlin-programming/extension-functions#:~:text=Basically,%20an%20extension%20function%20is,already%20available%20in%20String%20class) as well. In that case, you won't need to add a parameter for the view.

Here, I am taking a count and setting the text based on some logic. This logic can be anything.

We apply this <b>@BindingAdapter annotation</b> to manipulate how values with expressions are set to views. If you look inside it, it takes <b>an array of string values</b> which are basically the attributes associated with this binding adapter.

I updated my MainActivity to use ViewModel,

```
@Target(ElementType.METHOD)
public @interface BindingAdapter {
    /**
    * @return The attributes associated with this binding adapter.
    */
  String[] value();
    ......
}
```

<br/>
<br/>

To use in your XML, you just need to use this attribute `countText` like this:


```
<TextView
    ......
    app:countText="@{viewmodel.count}"
    .....    
/>
```

<br/>
<br/>

> For an attribute named `countText`, the library automatically tries to find the setter for our view attribute that accepts compatible types as the argument. The namespace of the attribute isn't considered, only the attribute name and type are used when searching for a method.

We have some predefined binding adapters provided by the framework (Do check this out in your project today):

<div style="text-align:center">
<img align="center" width="500" height="300" src="/Images/Article/data_binding_three_two.png">
</div>

<br/>
<br/>

> Note: Whenever we create our own custom binding adapters, anything annotated with @BindingAdapter is understood by the framework to be taken as a view attribute and it creates a setter for the same to be used later.

Coming back to our explanation, In our <b>ActivityDynamicBindingImpl</b> executeBindings(), we can see this logic added which directly calls our `countText()` method,

<br/>

```
if ((dirtyFlags & 0x7L) != 0) {
            // api target 1

            com.example.myapplication.DataBindingAdapter.countText(this.countTextview, androidxDatabindingViewDataBindingSafeUnboxViewmodelCountGetValue);
   }
```

<br/>

> Whenever a bound value changes, the generated binding class calls this setter method on the view with the binding expression.

So during the initialization, it sets the initial text according to the default value passed to our method. And, whenever the count is incremented(by clicking the button), LiveDataListener calls the `onFieldChange()` method updating mDirtyFlags value the same way, calling executeBindings() to rebind the views the latest value.

<br/>

> This time a new count has been passed to countText() method, we will see the view updated accordingly.

Here, You can see that with [DataBinding](https://developer.android.com/topic/libraries/data-binding), we moved almost everything related to our XML/views in just a few easy steps.

For reference, you can find my DataBinding sample [here](https://github.com/niharika2810/DataBindingSample)

## Resources:

* https://academy.realm.io/posts/droidkaigi-kevin-pelgrims-data-real-world-data-binding/
* https://viblo.asia/p/understanding-data-bindings-generated-code-and-how-does-android-data-binding-compiler-work-Ljy5Vd1yZra
* https://medium.com/@jencisov/androids-data-binding-s-baseobservable-class-and-bindable-annotation-in-kotlin-1a5c6682a3c1
* https://developer.android.com/topic/libraries/data-binding/binding-adapters

This is pretty much about the internals. I hope you learned something from this series and now I am pretty sure that if someone asks you about DataBinding magic, you will surely be able to answer all the queries and also, it will be easier for you to work with DataBinding but of course, after understanding the use-cases and the requirements.

And, If you have any suggestions/feedback, please do leave a comment so that we can all learn together.

Also, You know what? you can clap 50 times if you enjoyed reading it. Stay tuned for more. :)

Please feel free to reach out to me on [Twitter](https://twitter.com/theDroidLady) and [LinkedIn](https://www.linkedin.com/in/thedroidlady/).

This is published on [ProAndroidDev](https://proandroiddev.com/android-data-binding-under-the-hood-part-2-fdcbb0f54700)

Enjoy reading!!
 