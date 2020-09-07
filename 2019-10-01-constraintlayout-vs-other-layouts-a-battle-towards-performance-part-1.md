<div style="text-align:center">
<h1> Constraint vs Other Layouts- A Battle towards performance Part-1
</h1>
<br/>
Blog by <a href="http://thedroidlady.com/">theDroidLady</a>
</div>
<br/>
<br/>
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/process.gif">
</div>
<br/>
<br/>
Imagine, you’ve just joined a new job and you need to shop for a whole new wardrobe that goes with your workplace culture and outlook (formals in most cases).
You walk into an apparel store, head straight to the formals section, and you notice that some of the items hanging there are quite impractical and uncomfortable.
Now, would you rather just buy the items listed there because the store tells you to or would you rather walk around the other sections and see what’s best for you?

<b>Choosing a layout</b> for your app and website is also a similar process. But before we head to that, it’s important that we understand how our UI is created.

If you look at the <b>image</b> below, it shows you the <b>steps</b> that go into creating a UI.

1) The CPU takes the high-level object(<i>button, textView, etc…</i>) and turns it into a <b>display list</b> — a set of commands for drawing it.

2) GPU (Graphics Processing Unit) is a hardware piece which takes these <b>high-level objects</b> through OpenGL-ES API and <b>turns it into pixels in a texture or on the screen</b>.

>Once uploaded, the display list object is <b>cached</b>. That way, should we need to draw the display list again? Instead of creating it, we can just redraw the existing one- which is much cheaper probably like that not-so-quirky shirt hanging in the casual section.

So, creating this display list through <b>CPU and GPU</b> uploading might be expensive. And that’s why we should reduce the number of times these actions are performed.

### Let's understand the Rendering process-

1) <b>Measure</b> -The system completes a top-down traversal of the view tree to determine how large each ViewGroup and View element should be. When a ViewGroup is measured, it also measures its children.

2) <b>Layout</b> -Another top-down traversal occurs, with each ViewGroup determining the positions of its children using the sizes determined in the measure phase.

3) <b>Draw</b> -The system performs yet another top-down traversal. For each object in the view tree, Canvas object is created to send a list of drawing commands to the GPU. These commands include the ViewGroup and View objects' sizes and positions, which the system determined during the previous 2 phases.

### Manage complexity: Layouts matter

Android [Layouts](https://developer.android.com/guide/topics/ui/declaring-layout.html) allow you to nest UI objects in the view hierarchy.
This nesting can also impose a layout cost.
When your app processes an object for layout, the app performs the same process on all children of the layout as well.
For a complicated layout, sometimes a cost only arises the first time the system computes the layout.
For instance, when your app recycles a complex list item in a [RecyclerView](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.html) object, the system needs to lay out all of the objects. In another example, trivial changes can propagate up the chain towards the parent until they reach an object that doesn’t affect the size of the parent.

### Double taxation

Typically, the framework executes the [layout](https://developer.android.com/guide/topics/ui/declaring-layout.html) or measure stage in a single pass and quite quickly.

However, with some more complicated layout cases, the framework may have to iterate multiple times on parts of the hierarchy that require multiple passes to resolve before ultimately positioning the elements.

Having to perform more than one layout-and-measure iteration is what we call double taxation.
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/heirarchy.jpeg">
</div>
For example, when you use the [RelativeLayout](https://developer.android.com/reference/android/widget/RelativeLayout.html) container, which allows you to position [View](https://developer.android.com/reference/android/view/View.html) objects with respect to the positions of other View objects, the framework performs the following actions:

1) It executes a layout-and-measure pass, during which the framework calculates each child object’s position and size, based on each child’s request.
and then, it uses this data, also taking object weights into account, to figure out the proper position of correlated views.

2) After that, it performs a second layout pass to finalize the objects’ positions.

3) Finally, goes on to the next stage of the rendering process.

The more levels your view hierarchy has, the greater the potential performance penalty.

Containers other than [RelativeLayout](https://developer.android.com/reference/android/widget/RelativeLayout.html) may also give rise to double taxation. For example:

1) A [LinearLayout](https://developer.android.com/reference/android/widget/LinearLayout.html) view could result in a double layout-and-measure pass if you make it horizontal.

A double layout-and-measure pass may also occur in a vertical orientation if you add [measureWithLargestChild](https://developer.android.com/reference/android/widget/LinearLayout.html#attr_android:measureWithLargestChild), in which case the framework may need to do a second pass to resolve the proper sizes of objects.

2) The [GridLayout](https://developer.android.com/reference/android/widget/GridLayout.html) has a similar issue.

While this container also allows relative positioning, it normally avoids double taxation by pre-processing the positional relationships among child views.

However, if the layout uses weights or fills with the [Gravity](https://developer.android.com/reference/android/view/Gravity.html) class, the benefit of that preprocessing is lost, and the framework may have to perform multiple passes if the container were a RelativeLayout.

3) [FrameLayout](https://developer.android.com/reference/android/widget/FrameLayout) is designed to host a single child. It stacks its children one over another which is helpful to create overlay UI. Also, It’s a very good option for parent view in case of custom views. But FrameLayout can only position child views by applying gravity relative to their parent.

4) [ConstraintLayout](http://tools.android.com/tech-docs/layout-editor) has dual power of both Relative Layout as well as Linear layout: Set relative positions of views ( like Relative layout ) and also set weights for dynamic UI (which was only possible in Linear Layout). Despite the fact that it’s awesome, it fails to serve the purpose with simple UI layouts. Much like the formal section in your apparel store.

Multiple layout-and-measure passes are not, in themselves, a performance burden. But they can become so if they’re in the wrong spot. Just like shopping from the casual isn’t harmful until and unless you don’t know what you are looking for.

You should be wary of situations where one of the following conditions applies to your container:

 1) It is a root element in your view hierarchy.

 2) It has a deep view hierarchy beneath it.

 3) There are many instances of it populating the screen, similar to children in a [ListView](https://developer.android.com/reference/android/widget/ListView.html) object.

I hope you enjoyed this blog and learned something! In my next blog, I have covered the tools I used for measuring various layout performances.
