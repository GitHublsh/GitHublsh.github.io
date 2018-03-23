---
title: Android四大组件之Service
date: 2018-03-23 10:09:43
tags: [Service]
---
### 一、Service简介

##### Service 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。


###### 服务基本上分为两种形式：
	
* 启动

 * 当应用组件（如 Activity）通过调用 startService() 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方。例如，它可能通过网络下载或上传文件。 操作完成后，服务会自行停止运行。

* 绑定

 * 当应用组件通过调用 bindService() 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。 仅当与另一个应用组件绑定时，绑定服务才会运行。 多个组件可以同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

 两种形式是可以共存的，无论应用是处于启动状态还是绑定状态，抑或处于启动并且绑定状态，任何应用组件均可像使用 Activity 那样通过调用 Intent 来使用服务（即使此服务来自另一应用）。 不过，可以通过清单文件将服务声明为私有服务，并阻止其他应用访问。
 
 --
 
 需要注意的是：
 
 服务在其托管进程的主线程中运行，它既不创建自己的线程，也不在单独的进程中运行（除非另行指定）。 这意味着，如果服务将执行任何 CPU 密集型工作或阻止性操作（例如 MP3 播放或联网），则应在服务内创建新线程来完成这项工作。通过使用单独的线程，可以降低发生“应用无响应”(ANR) 错误的风险，而应用的主线程仍可继续专注于运行用户与 Activity 之间的交互。

 --
 
 
### 二、如何创建

#### 1.基础

要创建服务，必须创建 Service 的子类（或使用它的一个现有子类）。在实现中，需要重写一些回调方法，以处理服务生命周期的某些关键方面并提供一种机制将组件绑定到服务（如适用）。 应重写的最重要的回调方法包括：

 * onStartCommand()

  * 当另一个组件（如 Activity）通过调用 startService() 请求启动服务时，系统将调用此方法。一旦执行此方法，服务即会启动并可在后台无限期运行。 如果实现此方法，则在服务工作完成后，需要通过调用 stopSelf() 或 stopService() 来停止服务。（如果只想提供绑定，则无需实现此方法。）

 * onBind()
 
  * 当另一个组件想通过调用 bindService() 与服务绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，必须通过返回 IBinder 提供一个接口，供客户端用来与服务进行通信。请务必实现此方法，但如果并不希望允许绑定，则应返回 null。
  
 * onCreate()

  * 首次创建服务时，系统将调用此方法来执行一次性设置程序（在调用 onStartCommand() 或 onBind() 之前）。如果服务已在运行，则不会调用此方法。

* onDestory()

 * 当服务不再使用且将被销毁时，系统将调用此方法。服务应该实现此方法来清理所有资源，如线程、注册的侦听器、接收器等。 这是服务接收的最后一个调用。


######如果组件通过调用 startService() 启动服务（这会导致对 onStartCommand() 的调用），则服务将一直运行，直到服务使用 stopSelf() 自行停止运行，或由其他组件通过调用 stopService() 停止它为止。

######如果组件是通过调用 bindService() 来创建服务（且未调用 onStartCommand()，则服务只会在该组件与其绑定时运行。一旦该服务与所有客户端之间的绑定全部取消，系统便会销毁它。


仅当内存过低且必须回收系统资源以供具有用户焦点的 Activity 使用时，Android 系统才会强制停止服务。如果将服务绑定到具有用户焦点的 Activity，则它不太可能会终止；如果将服务声明为在前台运行（稍后讨论），则它几乎永远不会终止。或者，如果服务已启动并要长时间运行，则系统会随着时间的推移降低服务在后台任务列表中的位置，而服务也将随之变得非常容易被终止；如果服务是启动服务，则您必须将其设计为能够妥善处理系统对它的重启。 如果系统终止服务，那么一旦资源变得再次可用，系统便会重启服务


#### 2.清单文件中声明服务

如同 Activity（以及其他组件）一样，必须在应用的清单文件中声明所有服务。

要声明服务，请添加 <service> 元素作为 <application> 元素的子元素。例如：

	<manifest ... >
	  ...
	  <application ... >
	      <service android:name=".ExampleService" />
	      ...
	  </application>
	</manifest>


还可将其他属性包括在 <service> 元素中，以定义一些特性，如启动服务及其运行所在进程所需的权限。android:name 属性是唯一必需的属性，用于指定服务的类名。

为了确保应用的安全性，始终使用显式 Intent 启动或绑定 Service，且不要为服务声明 Intent 过滤器。 启动哪个服务存在一定的不确定性，而如果对这种不确定性的考量非常有必要，则可为服务提供 Intent 过滤器并从 Intent 中排除相应的组件名称，但随后必须使用 setPackage() 方法设置 Intent 的软件包，这样可以充分消除目标服务的不确定性。

此外，还可以通过添加 android:exported 属性并将其设置为 "false"，确保服务仅适用于应用。这可以有效阻止其他应用启动服务，即便在使用显式 Intent 时也如此。

#### 3. 创建启动服务

启动服务由另一个组件通过调用 startService() 启动，这会导致调用服务的 onStartCommand() 方法。

服务启动之后，其生命周期即独立于启动它的组件，并且可以在后台无限期地运行，即使启动服务的组件已被销毁也不受影响。 因此，服务应通过调用 stopSelf() 结束工作来自行停止运行，或者由另一个组件通过调用 stopService() 来停止它。

应用组件（如 Activity）可以通过调用 startService() 方法并传递 Intent 对象（指定服务并包含待使用服务的所有数据）来启动服务。服务通过 onStartCommand() 方法接收此 Intent。

例如，假设某 Activity 需要将一些数据保存到在线数据库中。该 Activity 可以启动一个协同服务，并通过向 startService() 传递一个 Intent，为该服务提供要保存的数据。服务通过 onStartCommand() 接收 Intent，连接到互联网并执行数据库事务。事务完成之后，服务会自行停止运行并随即被销毁。



--

注意：默认情况下，服务与服务声明所在的应用运行于同一进程，而且运行于该应用的主线程中。 因此，如果服务在用户与来自同一应用的 Activity 进行交互时执行密集型或阻止性操作，则会降低 Activity 性能。 为了避免影响应用性能，应在服务内启动新线程。

--


两种方式：

 * Service

  * 这是适用于所有服务的基类。扩展此类时，必须创建一个用于执行所有服务工作的新线程，因为默认情况下，服务将使用应用的主线程，这会降低应用正在运行的所有 Activity 的性能。

 * IntentService

  * 这是 Service 的子类，它使用工作线程逐一处理所有启动请求。如果不要求服务同时处理多个请求，这是最好的选择。 只需实现 onHandleIntent() 方法即可，该方法会接收每个启动请求的 Intent，就能够执行后台工作。

#### 4.IntentService

由于大多数启动服务都不必同时处理多个请求（实际上，这种多线程情况可能很危险），因此使用 IntentService 类实现服务也许是最好的选择。

IntentService 执行以下操作：

 * 创建默认的工作线程，用于在应用的主线程外执行传递给 onStartCommand() 的所有 Intent。

 * 创建工作队列，用于将 Intent 逐一传递给 onHandleIntent() 实现，这样就永远不必担心多线程问题。

 * 在处理完所有启动请求后停止服务，因此永远不必调用 stopSelf()。

 * 提供 onBind() 的默认实现（返回 null）。

 * 提供 onStartCommand() 的默认实现，可将 Intent 依次发送到工作队列和 onHandleIntent() 实现。

综上所述，只需实现 onHandleIntent() 来完成客户端提供的工作即可。

举个例子：

	public class HelloIntentService extends IntentService {
	
	  /**
	   * A constructor is required, and must call the super IntentService(String)
	   * constructor with a name for the worker thread.
	   */
	  public HelloIntentService() {
	      super("HelloIntentService");
	  }
	
	  /**
	   * The IntentService calls this method from the default worker thread with
	   * the intent that started the service. When this method returns, IntentService
	   * stops the service, as appropriate.
	   */
	  @Override
	  protected void onHandleIntent(Intent intent) {
	      // Normally we would do some work here, like download a file.
	      // For our sample, we just sleep for 5 seconds.
	      try {
	          Thread.sleep(5000);
	      } catch (InterruptedException e) {
	          // Restore interrupt status.
	          Thread.currentThread().interrupt();
	      }
	  }
	}

只需要一个构造函数和一个 onHandleIntent() 实现即可。
