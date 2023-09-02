### Fragment



#### 基础

##### Fragment 回调函数

 <img src="F:\Typora\Nodes\Android\Base\Image-1615973299463.png" alt="Image" style="zoom: 80%;" />

- **onAttach()：**   fragment与activity关联时调用，activity传入到此方法中；
- **onCreate()：** 系统在创建fragment时调用之；若想保留组件， 则在此初始化；
- **onCreateView()**：创建与fragment相关联的视图；
- **onActivityCreated()**：activity中的onCreated()方法已返回时调用
- **onStart():** 
- onResume(): 若原 fragment 无finish，那么 返回时不重新create 而是 resume
- onPause():  **用户离开片段的第一个信号；**
- onStop():
- onDestroyView()：**移除与fragment相关联的视图；**
- onDestroy():
- onDetach()： 取消fragment与activity相关联时调用；



Fragment切换记录：

存在两个fragment，fgm1，fgm2：

1. 先加载fgm1：

   **onAttach() --> onCreate() --> onCreateView() --> onActivityCreated() --> onStart() --> onResume()**

2. 此时切换到 fgm2：

   1. **fgm2：onAttach() --> onCreate();**  
   2. **fgm1：onPause() --> onStop() --> onDestroyView() --> onDestroy() --> onDetach()**
   3. **fgm2: onCreateView() --> onActivityCreated() --> onStart() --> onResume()**



#### Activity 与 Fragment

##### Activity 与 Fragment 通信

```java
getActivity().findViewById(R.id.xxx);//在fragment使用，获取Activity中view
```

也可以使用接口回调；



##### Activity 加载 Fragment

当Activity含有一个至多个fragment时候：

```java
/**
 *  container：fragment将要插入到的父集ViewGroup（来自Activity）
 *  savedInstanceState：恢复片段时，提供上一Fragment下相关数据Bundle
**/
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState) {
    //是否将获取view附加到ViewGroup中:
    // false：插入container；true：会在container中添加多余的ViewGroup    
    return inflater.inflate(R.layout.example_fragment, container, false);
}
```

静态使用fragment时候：可以使用android:id 或者 android:tag 来唯一标识fragment;

动态：

```java
// 动态添加Fragment
FragmentManager fragmentManager = getSupportFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

// 使用add; 也可以使用replace
fragmentTransaction.add(R.id.fragment_container, new ExampleFragment());
fragmentTransaction.commit();
```

**FragmentManager:**

- findFragmentById() (静态获取)
- findFragmentByTag()（在使用replace时候，可以传入第三个参数，即String tag）
- popBackStack(): 使fragment从栈中弹出
- addBackStackChangedListener(): 注册监听栈变化的监听器

**FragmentTransaction:**

- add() ：

  - 可重复添加相同Fragment：java.lang.IllegalStateException: Fragment already added；

  - 当前无fragment显示时，add可将新添加的fragment显示；当前有，只添加新fragment而已，不遮盖当前显示的fragment；   

  **通过add新添加的Fragment会执行到onResume()，如果当前有fragment显示，则新添加的fragment被覆盖没有机会显示！**

  ```java
  FragmentTransaction add(int containerViewId, Fragment fragment);
  
  // 其中tag可通过：FragmentManager.findFragmentByTag(tag)获取fragment
  FragmentTransaction add(int containerViewId, Fragment fragment, String tag);
  ```

  

- replace():

  replace方法的实质就是将待添加fragment对应的容器中添加的fragment 全部删除，再调用add(int,Fragment,String)将新fragment添加进去！

  replace方法在删除之前的Fragment时候，会检测当前replace的Fragment，如果Fragment已被add，则不删除，而是删除其他的fragment（如果有的话），再直接显示replace的Fragment。

  ```java
  FragmentTransaction replace(int containerViewId, Fragment fragment);
  FragmentTransaction replace(int containerViewId,  Fragment fragment,String tag);
  ```

  > This is essentially the same as calling remove for all currently added fragments that were added with the same containerViewIdand then add with the same arguments given here.
  >
  > public abstract FragmentTransaction replace(@IdRes int containerViewId,@NonNull Fragment fragment, @Nullable String tag);

addToBackStack(String)：允许用户通过按返回键返回上一fragment；参数可以传入null;



- deatch()：移除Fragment的View

  执行detach, 相应的fragment会相继执行onPause() -> onStop() -> onDestroyView()

  此时，fragment并未销毁，可以fragment.isDetached() 方法来判断fragment是否从viewTree中删除：

  如果想重新显示detach 的 fragment，则可以执行：

  ```java
  transaction.attach(fragment)//注这里不是调用fragment的回调方法onAttach哦！！！
  	.show(fragment)
  	.commit();
  Log.d(TAG,"判断 attach 之后的 fragment是否 isAdded ： " + fragment.isAdded());//false
  ```

  如果是删除，则执行remove即可；

  注：这里有个易错点，当执行add -> detach -> attach 时候，fragment.isAdded() 显示为false；

  当fragment执行完回调函数时候，再判断isAdded，那么就是true；



#### 实例化

1.使用构造函数；
2.使用静态工厂；

#####  注意

仅仅实例化的时候，不会执行onCreate，onCreateView等类似的回调方法！比如：仅仅new ExampleFragment()时候，并不会调用onCreate等回调方法，

而是直到activity添加fragment时候，才会执行相应的回调；因此在静态工厂方法中，newInstance中执行new Fragment() 并不会调用onCreate；

其次，不推荐使用带有参数的构造函数：当屏幕旋转的时候，fragment会重新从onCreate执行回调，这个时候会通过反射调用默认的构造函数（因此在使用newInstance时候也**不要将默认的空参数的构造方法设置为private**，这样会实例化失败）。那么需要的参数自然不会获取到，因此会出现旋转屏幕内容丢失的情况！

```java
//实例
public static ArgumentsFragment newInstance(String info) {
    Bundle args = new Bundle();
    ArgumentsFragment fragment = new ArgumentsFragment();
    args.putString(ARG,info);
    fragment.setArguments(args);
    return fragment;
}

public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if(getArguments() != null){
        Bundle args = getArguments();
        String info = args.getString(ARG);
    }
}
```

onHiddenChanged:

```java
//当add() + show()/hide()时候，旧Fragment会调用此方法
@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    if(activity!=null){
        if(hidden){
            //当该页面隐藏时
        }else {
            //当页面展现时
        }
    }
}
```

##### Adapter

- FragmentPagerAdapter

```java
public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
            + " v=" + ((Fragment)object).getView());
    mCurTransaction.detach((Fragment)object);
}
```

- FragmentStatePagerAdapter：

```java
public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    Fragment fragment = (Fragment) object;
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
            							+ " v=" + ((Fragment)object).getView());
    while (mSavedState.size() <= position) {
        mSavedState.add(null);
    }
    mSavedState.set(position, fragment.isAdded()
            ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
    mFragments.set(position, null);

    mCurTransaction.remove(fragment);
}
```

**注意区分:**

**前者只是将fragment detach，后者则是remove fragment；**

**是故FragmentStatePagerAdapter并不保存其他fragment状态，适合大量fragment；**

**相反前者，则适合少量的，需要来回滚动的fragment！**



#### 模板

```java
    public RecordInputFragment() {

    }


    public static RecordInputFragment newInstance(int studentId) {
        RecordInputFragment fragment = new RecordInputFragment();
        Bundle args = new Bundle();
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            studentId = getArguments().getInt(ARG_STUDENT_ID);
        }
    }
        @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        mContext = context;

        // Activity 实现监听接口
        if (context instanceof OnFragmentInteractionListener) {
            mListener = (OnFragmentInteractionListener) context;
        } else {
            throw new RuntimeException(context.toString() + " must implement 
                                       OnFragmentInteractionListener");
        }
    }
```