# Android

## 控件
### Notification

*状态通知栏主要涉及到2个类：Notification 和 NotificationManager*

**Notification**：通知信息类，它里面对应了通知栏的各个属性

**NotificationManager**：是状态栏通知的管理类，负责发通知、清除通知等操作。

**使用的基本流程：**

- **Step 1.** 获得NotificationManager对象： NotificationManager mNManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

- **Step 2.** 创建一个通知栏的Builder构造类： Notification.Builder mBuilder = new Notification.Builder(this);

- **Step 3.** 对Builder进行相关的设置，比如标题，内容，图标，动作等！

- **Step 4.**调用Builder的build()方法为notification赋值

- **Step 5**.调用NotificationManager的notify()方法发送通知！

- **PS:**另外我们还可以调用NotificationManager的cancel()方法取消通知

  

**设置相关的一些方法：**

```java
private Notification notification = new NotificationCompat.Builder(this, "test")
                .setContentTitle("官方通知")
                .setContentText("世界那么大，想去走走吗")
                .setSmallIcon(R.drawable.ic_baseline_account_box_24)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.android_1))
                .setColor(Color.parseColor("#ff0000"))
                .setContentIntent(pendingIntent)
                .setAutoCancel(true)
                .build();
```



调用下述的相关的方法进行设置：(官方API文档：[Notification.Builder](https://developer.android.google.cn/reference/androidx/core/app/NotificationCompat.Builder)) 常用的方法如下：

- **setContentTitle**(CharSequence)：设置标题
- **setContentText**(CharSequence)：设置内容
- **setSubText**(CharSequence)：设置内容下面一小行的文字
- **setTicker**(CharSequence)：设置收到通知时在顶部显示的文字信息
- **setWhen**(long)：设置通知时间，一般设置的是收到通知时的System.currentTimeMillis()
- **setSmallIcon**(int)：设置右下角的小图标，在接收到通知的时候顶部也会显示这个小图标
- **setLargeIcon**(Bitmap)：设置左边的大图标
- **setAutoCancel**(boolean)：用户点击Notification点击面板后是否让通知取消(默认不取消)
- **setDefaults**(int)：向通知添加声音、闪灯和振动效果的最简单、 使用默认（defaults）属性，可以组合多个属性，
  **Notification.DEFAULT_VIBRATE**(添加默认震动提醒)；
  **Notification.DEFAULT_SOUND**(添加默认声音提醒)；
  **Notification.DEFAULT_LIGHTS**(添加默认三色灯提醒)
  **Notification.DEFAULT_ALL**(添加默认以上3种全部提醒)
- **setVibrate**(long[])：设置振动方式，比如：
  setVibrate(new long[] {0,300,500,700});延迟0ms，然后振动300ms，在延迟500ms， 接着再振动700ms，关于Vibrate用法后面会讲解！
- **setLights**(int argb, int onMs, int offMs)：设置三色灯，参数依次是：灯光颜色， 亮持续时间，暗的时间，不是所有颜色都可以，这跟设备有关，有些手机还不带三色灯； 另外，还需要为Notification设置flags为Notification.FLAG_SHOW_LIGHTS才支持三色灯提醒！
- **setSound**(Uri)：设置接收到通知时的铃声，可以用系统的，也可以自己设置，例子如下:
  .setDefaults(Notification.DEFAULT_SOUND) //获取默认铃声
  .setSound(Uri.parse("file:///sdcard/xx/xx.mp3")) //获取自定义铃声
  .setSound(Uri.withAppendedPath(Audio.Media.INTERNAL_CONTENT_URI, "5")) //获取Android多媒体库内的铃声
- **setOngoing**(boolean)：设置为ture，表示它为一个正在进行的通知。他们通常是用来表示 一个后台任务,用户积极参与(如播放音乐)或以某种方式正在等待,因此占用设备(如一个文件下载, 同步操作,主动网络连接)
- **setProgress**(int,int,boolean)：设置带进度条的通知 参数依次为：进度条最大数值，当前进度，进度是否不确定 如果为确定的进度条：调用setProgress(max, progress, false)来设置通知， 在更新进度的时候在此发起通知更新progress，并且在下载完成后要移除进度条 ，通过调用setProgress(0, 0, false)既可。如果为不确定（持续活动）的进度条， 这是在处理进度无法准确获知时显示活动正在持续，所以调用setProgress(0, 0, true) ，操作结束时，调用setProgress(0, 0, false)并更新通知以移除指示条
- **setContentIntent**(PendingIntent)：PendingIntent和Intent略有不同，它可以设置执行次数， 主要用于远程服务通信、闹铃、通知、启动器、短信中，在一般情况下用的比较少。比如这里通过 Pending启动Activity：getActivity(Context, int, Intent, int)，当然还可以启动Service或者Broadcast PendingIntent的位标识符(第四个参数)：
  **FLAG_ONE_SHOT** 表示返回的PendingIntent仅能执行一次，执行完后自动取消
  **FLAG_NO_CREATE** 表示如果描述的PendingIntent不存在，并不创建相应的PendingIntent，而是返回NULL
  **FLAG_CANCEL_CURRENT** 表示相应的PendingIntent已经存在，则取消前者，然后创建新的PendingIntent， 这个有利于数据保持为最新的，可以用于即时通信的通信场景
  **FLAG_UPDATE_CURRENT** 表示更新的PendingIntent
  使用示例：

```java
//点击后跳转Activity
Intent intent = new Intent(context,XXX.class);  
PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);  
mBuilder.setContentIntent(pendingIntent)  
```

### AlertDialog

1. **基本使用流程**

    - **Step 1**：创建**AlertDialog.Builder**对象；
    - **Step 2**：调用**setIcon()**设置图标，**setTitle()**或**setCustomTitle()**设置标题；
    - **Step 3**：设置对话框的内容：**setMessage()**还有其他方法来指定显示的内容；
    - **Step 4**：调用**setPositive/Negative/NeutralButton()**设置：确定，取消，中立按钮；
    - **Step 5**：调用**create()**方法创建这个对象，再调用**show()**方法将对话框显示出来；

示例：

```java
public void dialogClick(View view) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setIcon(R.mipmap.ic_launcher)
                .setTitle("我是对话框")
                .setMessage("今天天气怎么样")
                .create()
                .show();

    }
```

2. **常用方法**（官方API文档：[AlertDialog官方文档](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog)）
    - [setButton](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setButton(int, java.lang.CharSequence, android.os.Message))(int whichButton, CharSequence text, [Message](https://developer.android.google.cn/reference/android/os/Message.html) msg)：设置按下按钮时要发送的消息

    - [setButton](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setButton(int, java.lang.CharSequence, android.graphics.drawable.Drawable, android.content.DialogInterface.OnClickListener))(int whichButton, CharSequence text, [Drawable](https://developer.android.google.cn/reference/android/graphics/drawable/Drawable.html) icon, [DialogInterface.OnClickListener](https://developer.android.google.cn/reference/android/content/DialogInterface.OnClickListener.html) listener)：设置要与按钮文本一起显示的图标以及在按下对话框的确定按钮时要调用的监听器。

    - [setIcon](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setIcon(android.graphics.drawable.Drawable))([Drawable](https://developer.android.google.cn/reference/android/graphics/drawable/Drawable.html) icon)：设置要在标题中使用的图标。

    - [setIconAttribute](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setIconAttribute(int))(int attrId)：设置图标属性。

    - [setMessage](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setMessage(java.lang.CharSequence))(CharSequence message)：设置对话框提示信息。

    - [setTitle](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setTitle(java.lang.CharSequence))(CharSequence title)：设置标题。

    - [setView](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setView(android.view.View, int, int, int, int))([View](https://developer.android.google.cn/reference/android/view/View.html) view, int viewSpacingLeft, int viewSpacingTop, int viewSpacingRight, int viewSpacingBottom)：设置要在对话框中显示的视图，指定在该视图周围显示的间距。

    - [setView](https://developer.android.google.cn/reference/androidx/appcompat/app/AlertDialog#setView(android.view.View))([View](https://developer.android.google.cn/reference/android/view/View.html) view)：设置自定义布局。

    - setPositiveButton(CharSequence text, final OnClickListener listener)：设置确定按钮，参数依次为：按钮显示文字，监听事件。

    - setNegativeButton(CharSequence text, final OnClickListener listener)：设置取消按钮，参数依次为：按钮显示文字，监听事件。

    - setNeutralButton(CharSequence text, final OnClickListener listener)：设置中间按钮，参数依次为：按钮显示文字，监听事件。

### PopupWindow

1. **几个常用的构造方法**
   - public PopupWindow (Context context)
   - public PopupWindow(View contentView, int width, int height)
   - public PopupWindow(View contentView)
   - public PopupWindow(View contentView, int width, int height, boolean focusable)

2. **常用的一些方法**
    - **setContentView**(View contentView)：设置PopupWindow显示的View
    - **getContentView**()：获得PopupWindow显示的View
    - **showAsDropDown(View anchor)**：相对某个控件的位置（正左下方），无偏移
    - **showAsDropDown(View anchor, int xoff, int yoff)**：相对某个控件的位置，有偏移
    - **showAtLocation(View parent, int gravity, int x, int y)**： 相对于父控件的位置（例如正中央Gravity.CENTER，下方Gravity.BOTTOM等），可以设置偏移或无偏移 PS:parent这个参数只要是activity中的view就可以了！
    - **setWidth/setHeight**：设置宽高，也可以在构造方法那里指定好宽高， 除了可以写具体的值，还可以用WRAP_CONTENT或MATCH_PARENT， popupWindow的width和height属性直接和第一层View相对应。
    - **setFocusable(true)**：设置焦点，PopupWindow弹出后，所有的触屏和物理按键都由PopupWindows 处理。其他任何事件的响应都必须发生在PopupWindow消失之后，（home 等系统层面的事件除外）。 比如这样一个PopupWindow出现的时候，按back键首先是让PopupWindow消失，第二次按才是退出 activity，准确的说是想退出activity你得首先让PopupWindow消失，因为不并是任何情况下按back PopupWindow都会消失，必须在PopupWindow设置了背景的情况下 。
    - **setAnimationStyle(int)：**设置动画效果
    - **[dismiss](https://developer.android.google.cn/reference/kotlin/android/widget/PopupWindow#dismiss())()：**关闭弹窗

## 布局

### LinearLayout

1. **XML attributes**
   - **gravity：**控制组件所包含的子元素的对齐方式，可多个组合
   - **layout_gravity：**控制该组件在父容器里的对齐方式
   - **background：**为该组件设置一个背景图片，或者是直接用颜色覆盖
   - **divider：**分割线
   - **showDividers：**设置分割线所在的位置，none（无），beginning（开始），end（结束），middle（每两个组件间）
   - **dividerPadding：**设置分割线的padding
   - **layout_weight（权重）：**该属性是用来等比例的划分区域

### RelativeLayout

![RelativeLayout 属性](https://note.youdao.com/yws/api/personal/file/WEB73eac672be8bf8d40bd107f02b0d5795?method=download&shareKey=e6a1acca8290a539c0d8ad6789a9f589)

### FrameLayout

1. **XML atrribute**
   - **android:foreground: **设置改帧布局容器的前景图像
   - **android:foregroundGravity: **设置前景图像显示的位置

### TableLayout

1. **XML atrribute**
   - **android:collapseColumns: **设置需要被隐藏的列的序号
   - **android:shrinkColumns: **设置允许被收缩的列的列序号
   - **android:stretchColumns: **设置运行被拉伸的列的列序号

> 以上这三个属性的列号都是从0开始算的,比如shrinkColunmns = "2",对应的是第三列！
> 可以设置多个,用逗号隔开比如"0,2",如果是所有列都生效,则用"\*"号即可

2. 子控件属性

    - **android:layout_column**：从第几列开始显示
    - **android:layout_span**：表示合并n个单元格,也就说这个组件占n个单元格

### GridLayout

![GridLayout](https://note.youdao.com/yws/api/personal/file/WEB68e088ad5f3e46a2d7c86106e7dab103?method=download&shareKey=568d3758dd3957713058aa295074aff9)

## 动画

### 补间动画

**1. 补间动画的分类和Interpolator**

- **AlphaAnimation：**透明度渐变效果，创建时许指定开始以及结束透明度，还有动画的持续 时间，透明度的变化范围(0,1)，0是完全透明，1是完全不透明；对应<**alpha**/>标签！
- **ScaleAnimation**：缩放渐变效果，创建时需指定开始以及结束的缩放比，以及缩放参考点， 还有动画的持续时间；对应<**scale**/>标签！
- **TranslateAnimation**：位移渐变效果，创建时指定起始以及结束位置，并指定动画的持续 时间即可；对应<**translate**/>标签！
- **RotateAnimation**：旋转渐变效果，创建时指定动画起始以及结束的旋转角度，以及动画 持续时间和旋转的轴心；对应<**rotate**/>标签
- **AnimationSet**：组合渐变，就是前面多种渐变的组合，对应<**set**/>标签

**2. 各种动画的详细讲解**

+ **AlphaAnimation(透明度渐变)**

> anim_alpha.xml

```xml
<alpha xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"  
    android:fromAlpha="1.0"  
    android:toAlpha="0.1"  
    android:duration="2000"/>
```

> 属性解释

**fromAlpha** :起始透明度
**toAlpha**:结束透明度
透明度的范围为：0-1，完全透明-完全不透明

+ **ScaleAnimation(缩放渐变)**

> anim_scale.xml

```xml
<scale xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/accelerate_interpolator"  
    android:fromXScale="0.2"  
    android:toXScale="1.5"  
    android:fromYScale="0.2"  
    android:toYScale="1.5"  
    android:pivotX="50%"  
    android:pivotY="50%"  
    android:duration="2000"/>
```

>  属性解释：

**fromXScale**/**fromYScale**：沿着X轴/Y轴缩放的起始比例
**toXScale**/**toYScale**：沿着X轴/Y轴缩放的结束比例
**pivotX**/**pivotY**：缩放的中轴点X/Y坐标，即距离自身左边缘的位置，比如50%就是以图像的 中心为中轴点
