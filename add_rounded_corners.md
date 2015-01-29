# 为控件添加圆角边框

控件的圆角边框可以使你的App看起来更美观，其实实现起来也很简单，以创建一个灰色的带圆角边框的Button为例：
![灰色的带圆角边框的Button](images/rounderd_corner.jpeg)

## 一、创建一个ShapeDrawable作为背景

在drawable目录下创建一个button_rounded_background.xml文件：

```xml
<shape xmlns:android = "http://schemas.android.com/apk/res/android"  
    android:shape= "rectangle" >  
    <solid android:color= "#AAAAAA" />  
    <corners android:radius= "15dp" />  
</shape> 
```

见名知意，除了`<solid/>`,`<corners/>`标签外，`<shape/>`还支持很多不同功能标签，更多介绍请移步Android官方文档：
http://developer.android.com/guide/topics/resources/drawable-resource.html#Shape

## 二、在Button中应用ShapeDrawable

创建main.xml文件，代码如下

```xml
<RelativeLayout xmlns:android = "http://schemas.android.com/apk/res/android"  
    android:layout_width= "fill_parent"  
    android:layout_height= "fill_parent"  
    android:gravity= "center" >  
  
    <Button  
        android:id ="@+id/button"  
        android:layout_width ="wrap_content"  
        android:layout_height ="wrap_content"  
        android:background ="@drawable/button_rounded_background"  
        android:padding ="10dp"  
        android:text ="@string/hello"  
        android:textColor ="#000000" />  
  
</RelativeLayout>
```
至此就已经构建完成了一个带圆角边框的Button。
ShapeDrawable不仅可以利用在Button中，它还可以应用与所有带背景的控件，例如ListView中的Item等。
