#### android view 体系

 ![1612781654146](F:\Typora\Nodes\Android\view\1612781654146.png)

- **Activity** 

  Activity包含一个Window，该Window在Activity的attach方法中通过调用PolicyManager.makeNewWindo创建

- **Window**

  表示顶层窗口，管理界面的显示和事件的响应；每个Activity 均会创建一个 **PhoneWindow**对象，是Activity和整个View系统交互的接口

- **DecorView**

  **extends FrameLayout;   FrameLayout extends ViewGroup；ViewGroup extends View**

  作为 PhoneWindow 内部变量

- **WindowManager**

  主要用来 管理窗口的一些状态、属性、view增加、删除、更新、窗口顺序、消息收集和处理等。



##### 创建 activity

 ![1612781814585](F:\Typora\Nodes\Android\view\1612781814585.png)

- performLacunchActivity

  完成被调用Activity的创建并启动，和其对manifest文件的读取

- addView

  进行**addView**时候，会触发ViewRoot中的**scheduleTraversals**-->**performTraversals()**,并在后者启动绘制view流程；



### view 绘制流程

 ![1612783360167](F:\Typora\Nodes\Android\view\1612783360167.png)

view绘制一般经过 测量（measure）+ 布局（layout）+ 绘图（draw）等流程

#### view绘制

- measure

  1. view 的 父view调用**view.measure(父要求尺寸**),view通过measure()来进行相应前置和优化

  2. view调用**onMeasure()**来进行相应的逻辑处理
  3. 通过setMeasuredDimension() 通知父view期望尺寸；父view会调用view的layout方法来通知其实际尺寸

- layout

  1. 父view通过view的layout将实际尺寸传给view，view通过layout中的setFrame保存，并通过onSizeChanged()告知view尺寸更改;
  2. view便会调用onLayout（若无子view，则为空实现。但是在ViewGroup中，通过onLayout 通知子view来做相应上述实现）

- draw：总调用方法 onDraw()

  - 绘制背景：drawBackground(),        //不可重写，只可通过setBackground来修改/设置背景

  - 绘制主体：onDraw(),                       //自定义一般修改这个方法

  - **绘制子view**：view的dispatchDraw(),   //若无子view，则是空实现；

    	若是ViewGroup，则调用ViewGroup.drawChild()->子view.draw()

  - 绘制前景：drawForeground()         //主体内容之上显示内容重写此

  **如果非要重写 draw(), 务必 super.draw()**



##### measure细节

- 关键函数

  measure() 调用 onMeasure()

  ```java
  final void measue(int widthMeasureSpec, int heightMeasureSpec); //final 不可重写
  void onMeasure(int widthMeasureSpec, int heightMeasureSpec);
  
  void measureChildren(int widthMeasureSpec, int heightMeasureSpec);
  void measureChild(View child, int parentWidthMeasureSpec,
                                int parentHeightMeasureSpec);
  void measureChildWithMargins(View child,int parentWidthMeasureSpec, int widthUsed, 
                                          int parentHeightMeasureSpec, 
                                          int heightUsed);
  ```

- MeasureSpec:SpecMode

  - UNSPECIFIED: 父view不对子view施加任何限制，子view可以获取任何想要获取的大小（一般用于系统内部）
  - AT_MOST: 针对于wrap_content属性，子元素至多达到父view指定大小
  - EXACTLY: 精确值，针对于match_parent,由父布局指定size



#### viewgroup 绘制

- 测量
  - measure()：做一些前置和优化工作
  - onMeasure(): 递归调用子view的measure(),并通过measure()传递父要求子view的尺寸参数（通过xml中参数计算得出）；最后调用setMeasuredDimension()告知自己的父view 期望的值



- 布局

  - layout：调用setFrame(),保存ViewGroup实际尺寸
  - onLayout(): 递归调用子view的layout方法，将计算结果传给view让其保存

- 绘制
  onDraw() 为空实现；

  方法与view类似，但是ViewGroup通常作为一个透明的容器，来盛放各式各样的View;

  通过dispatchView() 来绘制子 view；



**自定义VIewGroup**:

```java
public class MultiViewGroup extends ViewGroup {
    private static final String TAG = "MultiViewGroup";
    private Context mContext;
    public MultiViewGroup(Context context){
        super(context);
        mContext = context;
        init();
    }

    public MultiViewGroup(Context context, AttributeSet attrs){
        super(context,attrs);
        mContext = context;
        init();
    }

    private void init(){
        LinearLayout oneLL = new LinearLayout(mContext);
        oneLL.setBackgroundColor(Color.RED);

        LinearLayout twoLL = new LinearLayout(mContext);
        twoLL.setBackgroundColor(Color.GREEN);

        LinearLayout thirdLL = new LinearLayout(mContext);
        thirdLL.setBackgroundColor(Color.BLUE);

        addView(oneLL);  // ViewGroup 中 addView()
        addView(twoLL);
        addView(thirdLL);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = MeasureSpec.getSize(heightMeasureSpec);
        setMeasuredDimension(width,height);

        //下列虽然对子view进行measure，但是真正影响位置的是onLayout中对子view的layout。也就是说下面这段没用
        for(int i = 0;i < getChildCount();i++){
            View child = getChildAt(i);
            child.measure(width/2,height/2);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int startLeft = 25;
        int marginTop = 25;
        int tX = this.getWidth() - 2 * startLeft;
        int tY = this.getHeight() - 2 * marginTop;


        Log.d(TAG,"getWidth = " + getWidth() +" ;getHeight = " + getHeight());

        for(int i =0;i < getChildCount();i++){
            View child = getChildAt(i);
            child.layout(startLeft, marginTop, startLeft + tX, marginTop + tY);
            startLeft += (getWidth());
        }
    }
}
```