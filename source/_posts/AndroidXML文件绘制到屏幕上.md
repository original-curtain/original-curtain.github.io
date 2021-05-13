---
title: AndroidXML文件绘制到屏幕上
date: 2021-05-08 12:43:15
tags: Android
--- 
以下是android 29的分析：
<!--more-->
```
    //AppCompatDelegateImpl
    @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mAppCompatWindowCallback.getWrapped().onContentChanged();
    }
```
- 从setContent()入手，AppCompatDelegateImpl中先判断mSubDecor是否存在，不存在则createSubDecor();
- mSubDecor.findViewById(android.R.id.content)找到父布局来加载view

在LayoutInflater.inflate()中主要调用tryInflatePrecompiled()来生成view
```
private @Nullable
View tryInflatePrecompiled(@LayoutRes int resource, Resources res, @Nullable ViewGroup root,
    boolean attachToRoot) {
    ···
    String pkg = res.getResourcePackageName(resource);
    String layout = res.getResourceEntryName(resource);

    try {
        Class clazz = Class.forName("" + pkg + ".CompiledView", false, mPrecompiledClassLoader);
        Method inflater = clazz.getMethod(layout, Context.class, int.class);
        View view = (View) inflater.invoke(null, mContext, resource);
        ···
    }
    ···
}
```
这其中主要用到反射来生成view

pkg和layout的获取通过AssertManager来获取
```
    //ResourceImpl
    @NonNull
    String getResourcePackageName(@AnyRes int resid) throws NotFoundException {
        String str = mAssets.getResourcePackageName(resid);
        if (str != null) return str;
        throw new NotFoundException("Unable to find resource ID #0x"
                + Integer.toHexString(resid));
    }

    //AssertManager
    @UnsupportedAppUsage
    @Nullable String getResourcePackageName(@AnyRes int resId) {
        synchronized (this) {
            ensureValidLocked();
            return nativeGetResourcePackageName(mObject, resId);
        }
    }
```
mAssert是ResourceImpl中的AssertManager,在获取getResourcePackageName()时调用的就是native的方法