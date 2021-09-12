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

