---
layout: post
title:  "Android IPC简介"
date:   2015-04-06 15:14:54
categories: android
---
##序言

我们知道线程间通信很简单，只要通过内存数据共享就可以实现通信了，但是进程间怎样进行通信呢？在Android中一个进程指一个程序或一个应用，多进程通信，通常也是指应用间通信。**Android每个进程运行在不同的虚拟机中，不同的虚拟机在内存上拥有不同的地址空间，所以进程间的通信就不能通过内存地址进程共享数据了。**    

刚才说了，Android中**通常**一个应用对应一个程序，我们也可以给组件 (Activity, Service, Receiver, ContentProvider) 在 AndroidManifest.xml 中声明组件时候指定 *android:process* 就可以开启多进程模式。
	
	<activity
		android:name="io.github.jtlyuan.ipcdemo.IPCAndroidActivity"
		<!-- 用“:”方式是省略了包名，表示私有的，只能同一个app间共享 -->
		android:process=":second_process" />
		

*注意:1、一个App开启多进程模式，应用就拥有了多个虚拟机，他们之间内容是不共享的，只是公用同一份代码和资源文件。
2、会创建多个Application，也就是Application会多次创建调用。*

 
##进程通信方式

常见实现跨进程通信方式有，通过Intent传递数据、共享sdcard文件、AIDL、Socket通信...

*	**使用Intent方式**：这种方式很常见，比如：图片分享，QQ授权登录等。我们只要通过Intent就可以把数据传递给系统中其他应用的Activity或Service,这就可以达到进程间通信的目的了。
	>
	进程间传递的数据必须是基本类型数据或者是实现序列化接口（Serializable/Parcelable）的数据。

*	**共享文件方式**：各个应用通过文件读写通信。

##对象序列化

### 实现Serializable接口
*  只要指定 serialVersionUID = 82389323676232344L 值即可. 
	>
	注意:1、系统序列号的时候这这个值写到文件中，这个值是用来系统在反序列化的时候,判断文件中记录的序列号和当前类的序列号是否一样,一样才有可能序列号成功,否则失败.
	2、如果序列号不一样，反序列化直接失败。如果一样，但是类名或者变量类型改变也会失败。
	3、如果序列号一样，添加了变量和删除了变量，序列号是成功的。
	
* 经典的序列号和反序列化例子。

	//对象序列号
	Student student = new Student("wps", 5);
	ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("student.txt")));
	out.writeObject(student);
	out.close()
	
	//反序列化
	ObjectInputStream oin = new ObjectInputStream(new FileInputStream("student.txt"));
	Student student = (Student) oin.readObject();
	oin.close
	
###	Parcelable
先看个官方的demo

	public class MyParcelable implements Parcelable {
       private int mData;
       
       @Override
       public int describeContents() {
           return 0;//对象含有文件描述符返回1，否则为0
       }
       
       @Override
       public void writeToParcel(Parcel out, int flags) {
           out.writeInt(mData);
       }
  
       public static final Parcelable.Creator<MyParcelable> CREATOR
               = new Parcelable.Creator<MyParcelable>() {
               
           @Override
           public MyParcelable createFromParcel(Parcel in) {
               return new MyParcelable(in);
           }
           
           @Override
           public MyParcelable[] newArray(int size) {
               return new MyParcelable[size];
           }
       };
       
       private MyParcelable(Parcel in) {
           mData = in.readInt();
       }
    }
   
 
 总结：1、Serializable和Parcelable的比较： Serialiable比Parcelable实现序列号简单，但是Parcelable效率更好。通常用于进程间通信一般都用Parceable,用于文件读写用Serialiable。
	2、系统提供的一些已经实现Parcleable的常用类有Intent、Bundle、Bitmap
	
	
##IPC方式

##总结





























