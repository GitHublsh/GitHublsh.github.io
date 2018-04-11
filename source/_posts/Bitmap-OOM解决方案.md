---
title: Bitmap OOM解决方案
date: 2018-04-11 18:11:28
tags: [内存优化]
---

### Bitmap OOM常用解决方案
---

1. 在Android 2.3.3以及之前，建议使用Bitmap.recycle()方法，及时释放资源。

2. 在Android 3.0开始，可设置BitmapFactory.options.inBitmap值，(从缓存中获取)达到重用Bitmap的目的。如果设置，则inPreferredConfig属性值会被重用的Bitmap该属性值覆盖。

3. 通过设置Options.inPreferredConfig值来降低内存消耗：
     默认为ARGB_8888: 每个像素4字节. 共32位。
     Alpha_8: 只保存透明度，共8位，1字节。
     ARGB_4444: 共16位，2字节。
     RGB_565:共16位，2字节。
     如果不需要透明度，可把默认值ARGB_8888改为RGB_565,节约一半内存。
     
4. 通过设置Options.inSampleSize 对大图片进行压缩，可先设置Options.inJustDecodeBounds，获取Bitmap的外围数据，宽和高等。然后计算压缩比例，进行压缩。

5. 设置Options.inPurgeable和inInputShareable：让系统能及时回收内存。
 	* inPurgeable:
 		* 设置为True,则使用BitmapFactory创建的Bitmap用于存储Pixel的内存空间，在系统内存不足时可以被回收，当应用需要再次访问该Bitmap的Pixel时，系统会再次调用BitmapFactory 的decode方法重新生成Bitmap的Pixel数组。
 		* 设置为False时，表示不能被回收。
   		
 	* inInputShareable：设置是否深拷贝，与inPurgeable结合使用，inPurgeable为
 		* false，该参数无意义；
 		* True,share  a reference to the input data(inputStream, array,etc) 。 False ：a deep copy。
 		                               
6. 使用decodeStream代替其他decodeResource,setImageResource,setImageBitmap等方法来加载图片。
     
     
区别： 
decodeStream直接读取图片字节码，调用nativeDecodeAsset/nativeDecodeStream来完成decode。无需使用Java空间的一些额外处理过程，节省dalvik内存。但是由于直接读取字节码，没有处理过程，因此不会根据机器的各种分辨率来自动适应，需要在hdpi,mdpi和ldpi中分别配置相应的图片资源，否则在不同分辨率机器上都是同样的大小(像素点数量)，显示的实际大小不对。

decodeResource会在读取完图片数据后，根据机器的分辨率，进行图片的适配处理，导致增大了很多dalvik内存消耗。

       decodeStream调用过程：
             decodeStream(InputStream,Rect,Options) -> nativeDecodeAsset/nativeDecodeStream
       decodeResource调用过程：即finishDecode之后，调用额外的Java层的createBitmap方法，消耗更多dalvik内存。
             decodeResource(Resource,resId,Options)  -> decodeResourceStream (设置Options的inDensity和inTargetDensity参数)  -> decodeStream() (在完成Decode后，进行finishDecode操作)
             finishDecode() -> Bitmap.createScaleBitmap()(根据inDensity和inTargetDensity计算scale) -> Bitmap.createBitmap()

以上方法的组合使用，合理避免OOM错误。
