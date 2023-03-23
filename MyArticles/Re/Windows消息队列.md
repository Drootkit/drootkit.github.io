# 一、消息队列

消息队列是在什么时候创建的？
在线程被第一次创建的时候，当线程第一次被创建时，系统假设他不会被用于与任何用户界面所相关的任务，这样可以有效减少系统资源的使用。当检测到他调用一个和与图形相关的函数的时候，系统为他分配额外的资源



Windows下的应用是基于事件驱动的，他等待系统向其传递输入；dos程序是顺序的、过程驱动的。

## Windows消息传递的过程：

用户或应用程序的某些行为会产生一些事件。操作系统找到事件所属的应用程序，然后向该应用程序发送条相应的消息。然后，该消息就被加入到该引用程序的消息队列中。之后，应用程序不断地检杳消息队列，每当接收到一条消息时，应用程序就将该消息分发给与该消息所属窗口相关的窗口过程。最后，窗口过程执行与当前消息对应的指令。

## Windows消息队列：

Windows操作系统的内核空间中有一个系统消息队列（system message queue），在内核空间中还为每个UI线程分配各自的线程消息队列(Thread message queue)。在发生输入事件之后，Windows操作系统的输入设备驱动程序将输入事件转换为一个“消息”投寄到系统消息队列；操作系统的一个专门线程从系统消息队列取出消息，分发到各个UI线程的输入消息队列中。
Windows的事件驱动模式，并不是操作系统把消息主动分发给应用程序；而是由应用程序的每个UI线程通过“消息循环”代码从UI线程消息队列获取消息

## Windows为什么使用句柄：

为什么说句柄是一种指向指针的指针。
由于windows是一种以虚拟内存为基础的操作系统，其内存管理器经常会在内存中来回的移动对象，以此来满足各种应用程序对内存的需求。而对象的移动意味着对象内存地址的变化，正是因为如此，如果直接使用指针，在内存地址被改变后，系统将不知道到哪里去再调用这个对象。windows系统为了解决这个问题，系统专门为各种应用程序腾出了一定的内存地址（句柄）专门用来记录这些变化的地址（这些内存地址就是指向指针的指针），这些内存地址本身是一直不变化的。windows内存管理器在移动某些对象之后，他会将这些对象新的内存地址传给句柄，告诉他移动后对象去了哪里

## 死锁：Message Deadlocks

原因：发送的消息被处理时被”丢弃”了，而发送与接收的线程是同一队列，这就会导致该线程”死”了。

其实可以看做一个相互等待的场景：

- a线程发消息1给b线程
- b线程处理消息1，回调函数中发了消息2给a
- a接到消息2，但因为b对消息1的处理结果还没回来而等待
- b因为消息2的处理结果还没回来而等待

# 二、相关API

```c
postMessage //消息进入消息队列中后立即返回，消息可能不被处理。
PostThreadMessage //消息放入指定线程的消息队列中后立即返回，消息可能不被处理。
SendMessage //消息进入消息队列中，处理后才返回，如果消息不被处理，发送消息的线程将一直处于阻塞状态，等待消息返回。
SendNotifyMessage//如果消息进入本线程，则为SendMessage()，不是则采取postMessage()，当目标线程仍然依send处理
SendMessageTimeout //消息进入消息队列，处理或超时则返回，实际上SendMessage()就是建立在该函数上的
SendMessageCallback //在本线程再指定一个回调函数，当处理完后再次处理
BroadcastSystemMessage //发送目标为系统组件，比如驱动程序
```

## windows编程

c语言的程序至少有一个主函数main，Windows编程中存在两个主函数

```c
int WINAPI WinMain(
 HINSTANCE hInstance, // handle to current instance
 HINSTANCE hPrevInstance, // handle to previous instance
 LPSTR lpCmdLine,   // pointer to command line
 int nCmdShow     // show state of window
);

LRESULT CALLBACK WindowProc(
 HWND hwnd,   // handle to window
 UINT uMsg,   // message identifier
 WPARAM wParam, // first message parameter
 LPARAM lParam  // second message parameter
);
```

第二个是个callback函数，Windows必须至少一个callback函数

第一个winmain函数用来从消息队列中不断的发现消息，并处理消息（发送给对应的窗口函数）

```c
MSG msg; //定义消息名
while (GetMessage (&msg, NULL, 0, 0))
{
   TranslateMessage (&msg) ; //翻译消息
   DispatchMessage (&msg) ;  //撤去消息
}
return msg.wParam;
```

关于msg的结构定义，Windows中绝大多数都是基于结构体的体系。

```c
typedef struct tagMSG {   // msg 
  HWND  hwnd;  	// 要将消息发送到的目标句柄  
  UINT  message;		// 一个消息数字，对应一个消息类型
  WPARAM wParam;
  LPARAM lParam;
  DWORD time;			// 消息放入队列的时间（相对于Windows的时间，不是物理时间）
  POINT pt;				// 消息放队列的鼠标位置
} MSG;
```

## sendmessage和postmessage

前者发送消息之后需要等到返回才能返回、后者发送之后直接返回，不需要等待。
因为前者直接调用WndProc消息处理函数，所以需要等待返回之后才能返回。
后者是直接把消息放到消息队列中，所以可以直接返回。

# 三、实际运用

在开发层面运用很多，可以手动创建窗口，也可以利用MFC或者是利用其他的现成的框架。

## 细节

程序调用CreateWindowEx函数，将窗口的样式被设置成为WS_EX_TOOLWINDOW，该属性的窗口有以下特点：

1. 不在任务栏显示。
2. 不显示在Alt+Tab的切换列表中。
3. 在任务管理器的窗口管理Tab中不显示。

相当于创建了一个隐形窗口。通过消息机制调用回调函数实现创建子程序，利用回调函数可以自己根据消息执行的特点，可以规避调试。

文章https://www.anquanke.com/post/id/176079#h2-7既没有给出hash，图片还寄了，但是提供了一种恶意代码执行的思路，类似于Windows进程注入中的回调注入方式。