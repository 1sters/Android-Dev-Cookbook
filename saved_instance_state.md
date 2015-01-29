# Activity的状态保存


## 问题

关于如何保存activity被意外销毁（被意外你懂的，这个大师，那个助手的，还有那什么...）的状态数据，有多重方式，比如持久化了，还有如下这种方式：

```java
package com.suchangli.demo;

import android.app.Activity;
import android.os.Bundle;

public class MainActivity extends Activity{
        private String mStatu;
         @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                //恢复数据
                if(savedInstanceState ！= null){
                        mStatu = savedInstanceState.getString("statu");
                }
        }
         
        @Override
        protected void onSaveInstanceState(Bundle outState) {
                super.onSaveInstanceState(outState);
                
                outState.putString("mStatu", mStatu);
        }
        
        //也可以在这里恢复数据
        @Override
        protected void onRestoreInstanceState(Bundle savedInstanceState) {
                super.onRestoreInstanceState(savedInstanceState);
                mStatu = savedInstanceState.getString("statu");
        }
}
```

感觉很熟悉对吧，但是如果变量多了，或者是很多Activity都需要保存数据，是不是感觉特别的屎啊，那么问题来了，哪种方式比较好？

## 解决方案

请看下面的实现方式，把这种方式写到基类里面，让所有的Activity继承这个基类

```java
package com.suchangli.demo;

import java.io.Serializable;
import java.lang.annotation.Annotation;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Field;

import android.app.Activity;
import android.os.Bundle;
import android.os.Parcelable;

/**
 * 定义一个注解@SavedInstanceState，用于修饰需要保存的状态， 对于用这个注解修饰的变量或者状态，保存恢复就可以了。
 * 子类如果想保存某个字段的状态，只需要用这个注解修饰就可以了
 * 
 * @author tempadmin
 * 
 */
public class BaseActivity extends Activity {

        @Retention(RetentionPolicy.RUNTIME)
        @Target(ElementType.FIELD)
        public @interface SavedInstanceState {
        }

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                // 恢复数据
                if (savedInstanceState ！= null) {
                        restoreInstanceState(savedInstanceState);
                }
        }
        /**
         * 保存状态
         */
        @Override
        protected void onSaveInstanceState(Bundle outState) {
                Field[] fields = this.getClass().getDeclaredFields();
                Field.setAccessible(fields, true);
                Annotation[] ans;
                for (Field f : fields) {
                        ans = f.getDeclaredAnnotations();
                        for (Annotation an : ans) {
                                if (an instanceof SavedInstanceState) {
                                        try {
                                                Object o = f.get(this);
                                                if (o == null) {
                                                        continue;
                                                }
                                                String fieldName = f.getName();
                                                if (o instanceof Integer) {
                                                        outState.putInt(fieldName, f.getInt(this));
                                                } else if (o instanceof String) {
                                                        outState.putString(fieldName, (String) f.get(this));
                                                } else if (o instanceof Long) {
                                                        outState.putLong(fieldName, f.getLong(this));
                                                } else if (o instanceof Short) {
                                                        outState.putShort(fieldName, f.getShort(this));
                                                } else if (o instanceof Boolean) {
                                                        outState.putBoolean(fieldName, f.getBoolean(this));
                                                } else if (o instanceof Byte) {
                                                        outState.putByte(fieldName, f.getByte(this));
                                                } else if (o instanceof Character) {
                                                        outState.putChar(fieldName, f.getChar(this));
                                                } else if (o instanceof CharSequence) {
                                                        outState.putCharSequence(fieldName,
                                                                        (CharSequence) f.get(this));
                                                } else if (o instanceof Float) {
                                                        outState.putFloat(fieldName, f.getFloat(this));
                                                } else if (o instanceof Double) {
                                                        outState.putDouble(fieldName, f.getDouble(this));
                                                } else if (o instanceof String[]) {
                                                        outState.putStringArray(fieldName,
                                                                        (String[]) f.get(this));
                                                } else if (o instanceof Parcelable) {
                                                        outState.putParcelable(fieldName,
                                                                        (Parcelable) f.get(this));
                                                } else if (o instanceof Serializable) {
                                                        outState.putSerializable(fieldName,
                                                                        (Serializable) f.get(this));
                                                } else if (o instanceof Bundle) {
                                                        outState.putBundle(fieldName, (Bundle) f.get(this));
                                                }
                                        } catch (IllegalArgumentException e) {
                                        } catch (IllegalAccessException e) {
                                        } catch (Exception e) {
                                        }
                                }
                        }
                }

                super.onSaveInstanceState(outState);
        }

        /**
         * 在这里恢复数据
         * @param savedInstanceState
         */
        private void restoreInstanceState(Bundle savedInstanceState) {
                Field[] fields = this.getClass().getDeclaredFields();
                Field.setAccessible(fields, true);
                Annotation[] ans;
                for (Field f : fields) {
                        ans = f.getDeclaredAnnotations();
                        for (Annotation an : ans) {
                                if (an instanceof SavedInstanceState) {
                                        try {
                                                String fieldName = f.getName();
                                                Class cls = f.getType();
                                                if (cls == int.class || cls == Integer.class) {
                                                        f.setInt(this, savedInstanceState.getInt(fieldName));
                                                } else if (String.class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getString(fieldName));
                                                } else if (Serializable.class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getSerializable(fieldName));
                                                } else if (cls == long.class || cls == Long.class) {
                                                        f.setLong(this, savedInstanceState.getLong(fieldName));
                                                } else if (cls == short.class || cls == Short.class) {
                                                        f.setShort(this, savedInstanceState.getShort(fieldName));
                                                } else if (cls == boolean.class || cls == Boolean.class) {
                                                        f.setBoolean(this, savedInstanceState.getBoolean(fieldName));
                                                } else if (cls == byte.class || cls == Byte.class) {
                                                        f.setByte(this, savedInstanceState.getByte(fieldName));
                                                } else if (cls == char.class || cls == Character.class) {
                                                        f.setChar(this, savedInstanceState.getChar(fieldName));
                                                } else if (CharSequence.class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getCharSequence(fieldName));
                                                } else if (cls == float.class || cls == Float.class) {
                                                        f.setFloat(this, savedInstanceState.getFloat(fieldName));
                                                } else if (cls == double.class || cls == Double.class) {
                                                        f.setDouble(this, savedInstanceState.getDouble(fieldName));
                                                } else if (String[].class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getStringArray(fieldName));
                                                } else if (Parcelable.class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getParcelable(fieldName));
                                                } else if (Bundle.class.isAssignableFrom(cls)) {
                                                        f.set(this, savedInstanceState.getBundle(fieldName));
                                                }
                                        } catch (IllegalArgumentException e) {
                                        } catch (IllegalAccessException e) {
                                        } catch (Exception e) {
                                        }
                                        
                                }
                        }
                }
        }
}

```


这样就ok了，以后妈妈再也不用关心怎么保存状态了，就是这么任性。。。。

贡献者：[Com360](https://github.com/com360)
