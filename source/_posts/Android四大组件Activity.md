---
title: Android四大组件Activity
date: 2021-05-13 20:34:32
tags: [Android,Activity]
---
四大组件之Activity
<!--more-->

# Activity生命周期
## 在A的onCreate里打开B，然后finish掉A的生命周期
```
A onCreate
B onCreate
B onStart
B onResume
A onCreate
```
变种：A的onStart打开
```
A onCreate
A onStart
B onCreate
B onStart
B onResume
A onStop
A onCreate
```
A的onResume打开
```
A onCreate
A onStart
A onResume
A onPause
B onCreate
B onStart
B onResume
A onStop
A onCreate
```