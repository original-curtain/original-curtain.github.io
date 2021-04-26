---
title: JNI
date: 2021-04-26 11:16:57
tags: Java
---
JNI（Java Native Interface，Java本地接口），用于打通Java层与Native(C/C++)层。

Android系统在启动启动过程中，先启动Kernel创建init进程，紧接着由init进程fork第一个横穿Java和C/C++的进程，即Zygote进程。Zygote启动过程中会AndroidRuntime.cpp中的startVm创建虚拟机，VM创建完成后，紧接着调用startReg完成虚拟机中的JNI方法注册。
<!--more-->
```
int AndroidRuntime::startReg(JNIEnv* env)
{
    //设置线程创建方法为javaCreateThreadEtc
    androidSetCreateThreadFunc((android_create_thread_fn) javaCreateThreadEtc);

    env->PushLocalFrame(200);
    //进程JNI方法的注册
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}
static int register_jni_procs(const RegJNIRec array[], size_t count, JNIEnv* env) {
    for (size_t i = 0; i < count; i++) {
        if (array[i].mProc(env) < 0) {
            return -1;
        }
    }
    return 0;
}
static const RegJNIRec gRegJNI[] = {
    REG_JNI(register_android_os_MessageQueue),
    REG_JNI(register_android_os_Binder),
    ...
};
```
register_jni_procs(gRegJNI, NELEM(gRegJNI), env)这行代码的作用就是就是循环调用gRegJNI数组成员所对应的方法。

gRegJNI数组，有100多个成员变量，定义在AndroidRuntime.cpp：

JNI注册的两种时机：
- Android系统启动过程中Zygote注册，可通过查询AndroidRuntime.cpp中的gRegJNI，看看是否存在对应的register方法；
- 调用System.loadLibrary()方式注册

# JNI原理分析
Java层与Native层方法是如何注册并映射的，以MediaPlayer为例。 文件MediaPlayer.java中调用System.loadLibrary(“media_jni”)，把libmedia_jni.so动态库加载到内存。以loadLibrary为起点展开JNI注册流程的过程分析
## loadLibrary
```
//System.java
public static void loadLibrary(String libName) {
    //调用Runtime方法
    Runtime.getRuntime().loadLibrary(libName, VMStack.getCallingClassLoader());
}

//Runtime.java
void loadLibrary(String libraryName, ClassLoader loader) {
    if (loader != null) {
        String filename = loader.findLibrary(libraryName);
        //加载库
        String error = doLoad(filename, loader);
        if (error != null) {
            throw new UnsatisfiedLinkError(error);
        }
        return;
    }

    //loader为空，则会进入该分支
    String filename = System.mapLibraryName(libraryName);
    List<String> candidates = new ArrayList<String>();
    for (String directory : mLibPaths) {
        String candidate = directory + filename;
        candidates.add(candidate);
        if (IoUtils.canOpenReadOnly(candidate)) {
             //加载库
            String error = doLoad(candidate, loader);
            if (error == null) {
                return; //加载成功
            }
        }
    }
    ...
}
```
其真正加载的工作是由doLoad()实现

## doLoad
```
private String doLoad(String name, ClassLoader loader) {
    ...
    synchronized (this) {
        return nativeLoad(name, loader, ldLibraryPath);
    }
}
```
nativeLoad()这是一个native方法，再进入ART虚拟机的java_lang_Runtime.cc，最终的核心功能工作：
- 调用dlopen函数，打开一个so文件并创建一个handle；
- 调用dlsym()函数，查看相应so文件的JNI_OnLoad()函数指针，并执行相应函数。

## JNI_OnLoad
```
// android_media_MediaPlayer.cpp
jint JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env = NULL;
    if (register_android_media_MediaPlayer(env) < 0) {
        goto bail;
    }
    ...
}
```

## register_android_media_MediaPlayer
```
//android_media_MediaPlayer.cpp
static int register_android_media_MediaPlayer(JNIEnv *env) {
    return AndroidRuntime::registerNativeMethods(env,
                "android/media/MediaPlayer", gMethods, NELEM(gMethods));
}
```
其中gMethods，记录java层和C/C++层方法的一一映射关系。
```
static JNINativeMethod gMethods[] = {
    {"prepare",      "()V",  (void *)android_media_MediaPlayer_prepare},
    {"_start",       "()V",  (void *)android_media_MediaPlayer_start},
    {"_stop",        "()V",  (void *)android_media_MediaPlayer_stop},
    {"seekTo",       "(I)V", (void *)android_media_MediaPlayer_seekTo},
    {"_release",     "()V",  (void *)android_media_MediaPlayer_release},
    {"native_init",  "()V",  (void *)android_media_MediaPlayer_native_init},
    ...
};
```
这里涉及到结构体JNINativeMethod，其定义在jni.h文件：
```
typedef struct {
    const char* name;  //Java层native函数名
    const char* signature; //Java函数签名，记录参数类型和个数，以及返回值类型
    void*       fnPtr; //Native层对应的函数指针
} JNINativeMethod;
```

## registerNativeMethods
```
//AndroidRuntime.cpp
int AndroidRuntime::registerNativeMethods(JNIEnv* env,
    const char* className, const JNINativeMethod* gMethods, int numMethods){
    return jniRegisterNativeMethods(env, className, gMethods, numMethods);
}
```
jniRegisterNativeMethods该方法是由Android JNI帮助类JNIHelp.cpp来完成

## jniRegisterNativeMethods
```
// JNIHelp.cpp
extern "C" int jniRegisterNativeMethods(C_JNIEnv* env, const char* className, const JNINativeMethod* gMethods, int numMethods) {
    JNIEnv* e = reinterpret_cast<JNIEnv*>(env);
    scoped_local_ref<jclass> c(env, findClass(env, className));
    if (c.get() == NULL) {
        e->FatalError("");//无法查找native注册方法
    }
    if ((*env)->RegisterNatives(e, c.get(), gMethods, numMethods) < 0) {
        e->FatalError("");//native方法注册失败
    }
    return 0;
}
```

## RegisterNatives