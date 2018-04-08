---
title: Android开发中Handler的经典总结
tags: ['java', 'android', 'handler']
categories: ['java', 'android']
copyright: true
---
当应用程序启动时，Android首先会开启一个主线程(也就是UI线程)，主线程为管理界面中的UI控件，进行事件分发。

  

** 一、Handler的定义： **

主要接受子线程发送的数据， 并用此数据配合主线程更新UI。

解释：当应用程序启动时，Android首先会开启一个主线程 (也就是UI线程) ， 主线程为管理界面中的UI控件， 进行事件分发， 比如说， 你要是点击一个
Button ，Android会分发事件到Button上，来响应你的操作。  如果此时需要一个耗时的操作，例如: 联网读取数据，
或者读取本地较大的一个文件的时候，你不能把这些操作放在主线程中，如果你放在主线程中的话，界面会出现假死现象，
如果5秒钟还没有完成的话，会收到Android系统的一个错误提示  "强制关闭"。
这个时候我们需要把这些耗时的操作，放在一个子线程中，因为子线程涉及到UI更新，，Android主线程是线程不安全的，
也就是说，更新UI只能在主线程中更新，子线程中操作是危险的。 这个时候，Handler就出现了。，来解决这个复杂的问题
，由于Handler运行在主线程中(UI线程中)，  它与子线程可以通过Message对象来传递数据，
这个时候，Handler就承担着接受子线程传过来的(子线程用sedMessage()方法传弟)Message对象，(里面包含数据)  ，
把这些消息放入主线程队列中，配合主线程进行更新UI。

** 二、Handler一些特点 **

handler可以分发Message对象和Runnable对象到主线程中，
每个Handler实例，都会绑定到创建他的线程中(一般是位于主线程)，它有两个作用：

(1)安排消息或Runnable 在某个主线程中某个地方执行；

(2)安排一个动作在不同的线程中执行。

Handler中分发消息的一些方法

post(Runnable)

postAtTime(Runnable，long)

postDelayed(Runnable long)

sendEmptyMessage(int)

sendMessage(Message)

sendMessageAtTime(Message，long)

sendMessageDelayed(Message，long)

以上post类方法允许你排列一个Runnable对象到主线程队列中，

sendMessage类方法， 允许你安排一个带数据的Message对象到队列中，等待更新。

** 三、Handler实例 **

子类需要继承Hendler类，并重写handleMessage(Message msg) 方法， 用于接受线程数据。

以下为一个实例，它实现的功能为：通过线程修改界面Button的内容

    
    
    public class MyHandlerActivity extends Activity { 
        Button button; 
        MyHandler myHandler; 
     
        protected void onCreate(Bundle savedInstanceState) { 
            super。onCreate(savedInstanceState); 
            setContentView(R。layout。handlertest); 
     
            button = (Button) findViewById(R。id。button); 
            myHandler = new MyHandler(); 
            // 当创建一个新的Handler实例时， 它会绑定到当前线程和消息的队列中，开始分发数据 
            // Handler有两个作用， (1) : 定时执行Message和Runnalbe 对象 
            // (2): 让一个动作，在不同的线程中执行。 
     
            // 它安排消息，用以下方法 
            // post(Runnable) 
            // postAtTime(Runnable，long) 
            // postDelayed(Runnable，long) 
            // sendEmptyMessage(int) 
            // sendMessage(Message); 
            // sendMessageAtTime(Message，long) 
            // sendMessageDelayed(Message，long) 
          
            // 以上方法以 post开头的允许你处理Runnable对象 
            //sendMessage()允许你处理Message对象(Message里可以包含数据，) 
     
            MyThread m = new MyThread(); 
            new Thread(m)。start(); 
        } 
     
        /** 
        * 接受消息，处理消息 ，此Handler会与当前主线程一块运行 
        * */ 
    class MyHandler extends Handler { 
            public MyHandler() { 
            } 
     
            public MyHandler(Looper L) { 
                super(L); 
            } 
     
            // 子类必须重写此方法，接受数据 
            @Override 
            public void handleMessage(Message msg) { 
                // TODO Auto-generated method stub 
                Log。d("MyHandler"， "handleMessage。。。。。。"); 
                super。handleMessage(msg); 
                // 此处可以更新UI 
                Bundle b = msg。getData(); 
                String color = b。getString("color"); 
                MyHandlerActivity。this。button。append(color); 
     
            } 
        } 
     
        class MyThread implements Runnable { 
            public void run() { 
     
                try { 
                    Thread。sleep(10000); 
                } catch (InterruptedException e) { 
                    // TODO Auto-generated catch block 
                    e。printStackTrace(); 
                } 
     
                Log。d("thread。。。。。。。"， "mThread。。。。。。。。"); 
                Message msg = new Message(); 
                Bundle b = new Bundle();// 存放数据 
                b。putString("color"， "我的"); 
                msg。setData(b); 
     
                MyHandlerActivity。this。myHandler。sendMessage(msg); // 向Handler发送消息，更新UI 
     
            } 
        } 
    } 

  
转载地址： [ 点击打开链接 ](http://mobile.51cto.com/aprogram-442833.htm)  

