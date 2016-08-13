---
layout: post
title: 使用LayoutInflater布局显示异常
description: 你使用LayoutInflater布局显示异常过吗？
category: blog
---

在实际开发中一下代码会被经常使用。
    
    LayoutInflater.from(this).inflate(R.layout.popup_choose_type, null);
可能我们习惯性的写法就是上面酱紫啦，但是在`AS`中，上面的这段话会被报警告的。
这个都不是重点，重点是大家一定遇到过在inflate某个布局时，如果这样写了之后，那么它的父布局上面的padding或者margin的属性一定会没有效果的！这个是为什么？还有AS为什么不建议我们使用null?这个才是今天要说的重点。

    inflate(int resource, ViewGroup root, boolean attachToRoot) 
这个方法一共可以使用三个，啦啦啦，第二个root的参数，通常我们就用了null，最后一个boolean的值那估计也用的比较少了。

还有一个场景，在Fragment填充布局的时候，通常又是这样写的：

     @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_flow1, container, false);

    }
其实都是使用的那个`inflate`的方法。

接下来重点说说这个方法的最后两个参数的意思：

**Root**:
>Optional view to be the parent of the generated hierarchy (if attachToRoot is true), or else simply an object that provides a set of LayoutParams values for root of the returned hierarchy (if attachToRoot is false.)

>如果第三个参数`attachToRoot`是`true`的话，那么这个View就是孩纸它爹了，但是如果是`false`的话，那么就仅仅作为一个提供`LayoutParams`的东西了。

**attachToRoot**
>Whether the inflated hierarchy should be attached to the root parameter? If false, root is only used to create the correct subclass of LayoutParams for the root view in the XML.

>false，那么直接返回我们填充的那个view,root就只作为layoutParams!


两个参数的方法最终回去调用三个参数的方法：

     public View inflate(int resource, ViewGroup root) {
        return inflate(resource, root, root != null);
    }



    // Temp is the root view that was found in the xml
    final View temp = createViewFromTag(root, name, attrs, false);

    ViewGroup.LayoutParams params = null;

    if (root != null) {
        if (DEBUG) {
            System.out.println("Creating params from root: " +
                    root);
        }
        // Create layout params that match root, if supplied
        params = root.generateLayoutParams(attrs);
        if (!attachToRoot) {
            // Set the layout params for temp if we are not
            // attaching. (If we are, we use addView, below)
            temp.setLayoutParams(params);
        }

    // We are supposed to attach all the views we found (int temp)
    // to root. Do that now.
    if (root != null && attachToRoot) {
        root.addView(temp, params);
    }

    // Decide whether to return the root that was passed in or the
    // top view found in xml.
    if (root == null || !attachToRoot) {
        result = temp;
    }

    ...
    return result;

在`LayouInflater`的代码中,我们可以看到相关的代码，大致流程是通过xml将对应的view先创建出来，然后创建出一个`LayoutParams`。如果`root`不为空，先通过`root`得到一个`LayoutParams`但是`attachToRoot`为false的话，那么就直接给该`View`设置`LayoutParamas`,另外一种情况，那么就是直接让将`view`添加给`root`,然后将`root`返回！



解决刚刚的一些问题：


当我们不使用一个`root`时，那就是没有设置对应的`Layoutparams`。那么默认滴就会走默认的`LayoutParams`...


* 解决方案1：
每次指定一个`root`,这个是优先考虑的，其实也很简单，无论是在`listView`或者`Fragment`填充布局时，其实都有一个`ViewGroup parent`的参数供你使用的！  所以，不要偷懒写成最开始的那个样子了！


        @Override
        public View getView(final int position, View convertView, ViewGroup parent) {
            final SubViewHolder h;
            if (null == convertView) {
                convertView = LayoutInflater.from(mContext).inflate(R.layout.item_city_select_name, parent, false);
                h = new SubViewHolder();
                h.mCityName = (TextView) convertView.findViewById(R.id.city_name);
                convertView.setTag(h);
            } else {
                h = (SubViewHolder) convertView.getTag();
            }


        @Override
         public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_flow, container, false);
        }

这个也告诉了我们一个道理：没有爸爸的孩纸是多么滴可怕。。。哈哈哈！！

* 解决方案2：

在自己的布局基础上，**再包裹一层父布局**。这个其实当然不好咯！！大家都懂！！