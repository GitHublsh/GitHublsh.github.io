---
title: IPC基础概念介绍
date: 2018-03-22 09:09:50
tags: [Android进阶]
---

## IPC基础概念介绍

#### 一、简介

* IPC是 Inter-Process Communication的缩写，意为进程间通信或跨进程通信，是指两个进程之间进行数据交换的过程。

* 线程是CPU调度的最小单元，同时线程是一种有限的系统资源。进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。一个进程可以包含多个线程，因此进程和线程是包含与被包含的关系。最简单的情况下，一个进程中只可以有一个线程，即主线程，在Android中也叫UI线程。


#### 二、Android中的多进程模式

 Android中开启多进程的方法：
 
	 <activity android:name=".BinderOneActivity"
	            android:process="com.neil.binder.one"/>
	 <activity android:name=".BinderTwoActivity"
	            android:process="com.neil.binder.two"/>
	 <activity android:name=".BinderThreeActivity"
	            android:process="com.neil.binder.three"/>
	            
在AndroidManifest.xml指定android:process属性，若没有指定process属性，默认的进程中，默认进程的进程名是包名。

日志输出：

	u0_a60    4080  1207  1255464 42888 ffffffff b74c94f5 S com.example.liushihan.glidedemo
	u0_a60    4113  1207  1256320 42124 ffffffff b74c94f5 S com.neil.binder.one
	u0_a60    4144  1207  1254256 42088 ffffffff b74c94f5 S com.neil.binder.two
	u0_a60    4170  1207  1254256 41744 ffffffff b74c94f5 S com.neil.binder.three

Android系统为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据。两个应用通过ShareUID跑在同一个进程中是有要求的，需要这两个应用有相同的ShareUID并且签名相同才可以。在这种情况下，它们可以互相访问对方的私有数据，比如data目录，组件信息等。


	            
#### 三、序列化和反序列化

1. 序列化

由于存在于内存中的对象都是暂时的，无法长期驻存，为了把对象的状态保持下来，这时需要把对象写入到磁盘或者其他介质中，这个过程就叫做序列化。

指把堆内存中的 Java 对象数据，通过某种方式把对象存储到磁盘文件中或者传递给其他网络节点（在网络上传输）。这个过程称为序列化。通俗来说就是将数据结构或对象转换成二进制串的过程

2. 反序列化

反序列化恰恰是序列化的反向操作，也就是说，把已存在在磁盘或者其他介质中的对象，反序列化（读取）到内存中，以便后续操作，而这个过程就叫做反序列化。

  概括性来说序列化是指将对象实例的状态存储到存储媒体（磁盘或者其他介质）的过程。在此过程中，先将对象的公共字段和私有字段以及类的名称（包括类所在的程序集）转换为字节流，然后再把字节流写入数据流。在随后对对象进行反序列化时，将创建出与原对象完全相同的副本。
  
  把磁盘文件中的对象数据或者把网络节点上的对象数据，恢复成Java对象模型的过程。也就是将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程
  
#### 四、Serializable VS  Parcelable

##### 1. Serializable

Serializable是java提供的一个序列化接口，它是一个空接口，专门为对象提供标准的序列化和反序列化操作，使用Serializable实现类的序列化比较简单，只要在类声明中实现Serializable接口即可。

举个例子：

	import java.io.Serializable;
	
	public class Person implements Serializable {
	
		**
	     * 生成序列号标识
	     */
	    private static final long serialVersionUID = -2083503801443301445L;
	    
	    private String firstName;
	    private String lastName;
	    private int age;
	
	
	    public Person(String firstName, String lastName, int age) {
	        this.firstName = firstName;
	        this.lastName = lastName;
	        this.age = age;
	    }
	
	
	    public String getFirstName() {
	        return firstName;
	    }
	
	    public void setFirstName(String firstName) {
	        this.firstName = firstName;
	    }
	
	    public String getLastName() {
	        return lastName;
	    }
	
	    public void setLastName(String lastName) {
	        this.lastName = lastName;
	    }
	
	    public int getAge() {
	        return age;
	    }
	
	    public void setAge(int age) {
	        this.age = age;
	    }
	}


实现的Serializable接口并声明了序列化标识serialVersionUID。

那么serialVersionUID有什么作用呢？

实际上我们不声明serialVersionUID也是可以的，因为在序列化过程中会自动生成一个serialVersionUID来标识序列化对象。

那么既然自动生成我们是不是可以不写？

由于serialVersionUID是用来辅助序列化和反序列化过程的，原则上序列化后的对象中serialVersionUID只有和当前类的serialVersionUID相同才能够正常被反序列化，也就是说序列化与反序列化的serialVersionUID必须相同才能够使序列化操作成功。具体过程是这样的：序列化操作的时候系统会把当前类的serialVersionUID写入到序列化文件中，当反序列化时系统会去检测文件中的serialVersionUID，判断它是否与当前类的serialVersionUID一致，如果一致就说明序列化类的版本与当前类版本是一样的，可以反序列化成功，否则失败。不指定的话在反序列化会报错。


例子的实现很简单，当然也是有弊端的，如果在此过程中使用反射，可能会创建大量其他对象，这就可能会导致很多垃圾回收。造成的结果是性能差和电池耗尽。

##### 2. Parcelable

Parcelable是Android SDK中接口。鉴于Serializable在内存序列化上开销比较大，而内存资源属于android系统中的稀有资源（android系统分配给每个应用的内存开销都是有限的），为此android中提供了Parcelable接口来实现序列化操作，Parcelable的性能比Serializable好，在内存开销方面较小，所以在内存间数据传输时推荐使用Parcelable，如通过Intent在activity间传输数据，而Parcelable的缺点就使用起来比较麻烦。

举个例子：

	import android.os.Parcel;
	import android.os.Parcelable;
	
	public class Person implements Parcelable {
	
	    private String firstName;
	    private String lastName;
	    private int age;
	
	
	    public Person(String firstName, String lastName, int age) {
	        this.firstName = firstName;
	        this.lastName = lastName;
	        this.age = age;
	    }
	
	
	    public String getFirstName() {
	        return firstName;
	    }
	
	    public void setFirstName(String firstName) {
	        this.firstName = firstName;
	    }
	    
	    public String getLastName() {
	        return lastName;
	    }
	
	    public void setLastName(String lastName) {
	        this.lastName = lastName;
	    }
	
	    public int getAge() {
	        return age;
	    }
	
	    public void setAge(int age) {
	        this.age = age;
	    }
	
	
	    @Override
	    public int describeContents() {
	        return 0;
	    }
	
	    @Override
	    public void writeToParcel(Parcel dest, int flags) {
	        dest.writeString(this.firstName);
	        dest.writeString(this.lastName);
	        dest.writeInt(this.age);
	    }
	
		protected Person(Parcel in) {
		        this.firstName = in.readString();
		        this.lastName = in.readString();
		        this.age = in.readInt();
		    }
	
	    public static final Parcelable.Creator<Person> CREATOR = new Parcelable.Creator<Person>() {
	        @Override
	        public Person createFromParcel(Parcel source) {
	            return new Person(source);
	        }
	
	        @Override
	        public Person[] newArray(int size) {
	            return new Person[size];
	        }
	    };
	}


相对Serializable稍显复杂。

* Parcelable.Creator

	* 必须实现的接口。

* T createFromParcel (Parcel source)

	* 创建序列化的实例，实现从Parcel容器中读取传递数据值,封装成Parcelable对象返回逻辑层

* T[] newArray (int size)

	* 创建一个类型为T，长度为size的数组，供外部类反序列化本类数组使用。

* void writeToParcel (Parcel dest, 
                int flags)
                

	* 将当前对象写入序列化结构中


简单概括下：

 序列化过程中，需要实现序列化和反序列化以及内容描述。
 
 writeToParcel实现序列化，通过CREATOR内部对象来实现反序列化，其内部通过createFromParcel方法来创建序列化对象并通过newArray方法创建数组，最终利用Parcel的一系列read方法完成反序列化，最后由describeContents完成内容描述功能，该方法一般返回0，仅当对象中存在文件描述符时返回1。
 
 一句话概括，就是通过writeToParcel将我们的对象映射成Parcel对象，再通过createFromParcel将Parcel对象映射成我们的对象。


##### 3. Serializable 和 Parcelable 异同

* 都能序列化，都可用于Intent传递数据

* Serializable是Java中的序列化接口，使用简单但是内存开销大，序列化和反序列化需要大量的I/O操作。

* Parcelable是Android SDK中的序列化接口，更适用Android平台，使用起来稍显复杂，但是效率很高。内存开销较小。

* Parcelable主要用于内存序列化上，因此在序列化到存储设备或者网络传输方面还是尽量选择Serializable接口

