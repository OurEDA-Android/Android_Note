# ListView的构成 

## Adapter接口:
提供了数据抽象和视图抽象。

## BaseAdapter:

经常用于自定义Adapter的抽象类，原来数据是通过观察者模式搞定的啊，但是和View相关的操作的接口并没有看到，但是它实现的接口ListAdapter的父类Adapter出现了熟悉的东西。这个抽象类中只实现了数据相关的，所以我们在新建View 的时候才需要继承3个View操作的方法。

ListAdapter中增加了两个判断是否eable的方法。

## ArrayAdapter：
我们在使用一种更简单直有string类型的listView中曾经使用过这种写法，现在我们来看一下里面具体的实现过程。
看完了觉得没啥说的，还是有一个过滤首字母的filter，其余的更简单。


## SimpleAdapter:

这是我们很熟知的一个东西啦，和我们平时写的自定义Adapter的写法差不多。区别是通过一个ViewBinder判断数据和View的绑定关系，如果确认了绑定关系的话，就会进行赋值，就代码来看，SimpleAdapter只支持三种类型的子View。

``` java

                if (!bound) {
                    if (v instanceof Checkable) {
						...
                    } else if (v instanceof TextView) {
						...
                    } else if (v instanceof ImageView) {
						...
                    } else {
                        throw new IllegalStateException(v.getClass().getName() + " is not a " +
                                " view that can be bounds by this SimpleAdapter");
                    }
                }


```

另外里面出现了一个Filter，这个过滤器的作用是获取匹配的前缀字符，不匹配就移除。

[Filter阅读](Filter相关.md)

## 抽象类AdapterView:
指使用了Adapter进行数据填充的View类型。

`OnItemClickListener` \ `OnItemLongClickListener`接口处理每一项的点击事件。


``` java


    public boolean performItemClick(View view, int position, long id) {
        final boolean result;
        if (mOnItemClickListener != null) {
        	// 播放音效
            playSoundEffect(SoundEffectConstants.CLICK);
            mOnItemClickListener.onItemClick(this, view, position, id);
            result = true;
        } else {
            result = false;
        }

        if (view != null) {
        	// 设定view的状态
            view.sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        }
        return result;
    }


```

eventType容后分析

``` java 

    public void sendAccessibilityEvent(int eventType) {
        if (mAccessibilityDelegate != null) {
            mAccessibilityDelegate.sendAccessibilityEvent(this, eventType);
        } else {
            sendAccessibilityEventInternal(eventType);
        }
    }


```

获取选中item 的方法：

``` java

    /**
     * @return The data corresponding to the currently selected item, or
     * null if there is nothing selected.
     */
    public Object getSelectedItem() {
        T adapter = getAdapter();
        int selection = getSelectedItemPosition();
        if (adapter != null && adapter.getCount() > 0 && selection >= 0) {
            return adapter.getItem(selection);
        } else {
            return null;
        }
    }



```

判断View在ViewGroup的位置：

``` java 

    public int getPositionForView(View view) {
        View listItem = view;
        try {
            View v;
            // 递归到最外层
            while ((v = (View) listItem.getParent()) != null && !v.equals(this)) {
                listItem = v;
            }
        } catch (ClassCastException e) {
        	// -1
            // We made it up to the window without find this list view
            return INVALID_POSITION;
        }

        if (listItem != null) {
            // Search the children for the list item
            final int childCount = getChildCount();
            for (int i = 0; i < childCount; i++) {
                if (getChildAt(i).equals(listItem)) {
                    return mFirstPosition + i;
                }
            }
        }

        // Child not found!
        return INVALID_POSITION;
    }


```

中间出现了一个`AdapterContextMenuInfo`（上下文菜单）能获取上下文菜单的信息，
中间出现了`dispatchSaveInstanceState`（View中）防止创建冻结的View.

* DataSetObserver


``` java 

    /**
     * Receives call backs when a data set has been changed, or made invalid. The typically data sets
     * that are observed are {@link Cursor}s or {@link android.widget.Adapter}s.
     * DataSetObserver must be implemented by objects which are added to a DataSetObservable.
    */
 	
	public abstract class DataSetObserver {
	
    /*
      This method is called when the entire data set has changed,
      most likely through a call to {@link Cursor#requery()} on a {@link Cursor}.
     */
    public void onChanged() {
        // Do nothing
    }

    /**
     * This method is called when the entire data becomes invalid,
     * most likely through a call to {@link Cursor#deactivate()} or {@link 		Cursor#close()} on a
     * {@link Cursor}.
     */
    public void onInvalidated() {
        // Do nothing
    }
	}



```

* Parcelable序列化接口
* AdapterDataSetObserver
onChange() 与 onInvalidated() 方法，就能对比得出，前者会对当前位置的状态进行同步，而后者会重置所有位置的状态。从代码的注释里面还可以获取得到更多的信息。