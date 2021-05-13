---
title: 第三方开源框架Glide
date: 2021-04-29 09:43:17
tags: Android
---
glide最大的优势就是对bitmap的管理是跟随生命周期去发生改变的
<https://cloud.tencent.com/developer/article/1621891>

GlideV4的源码分析，其添加依赖导入位为：
```
implementation 'com.github.bumptech.glide:glide:4.9.0'
```
简单使用：
```
// 这里便是与 Glide 3+ 的不同
RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.loading);
// 它需要一个
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```
<!--more-->
# 流程

## with
Glide.with 方法主要是为 Context 构建 RequestManager , 主要流程如下

- 获取进程间单例的 Glide 对象

    - 线程池
    - 缓存池
    - 编解码器

- 获取进程间单例的 RequestManagerRetriever 对象
- 通过 RequestManagerRetriever 创建当前 context 的 RequestManager

    - 若可以绑定 Activity, 则为 Activity 添加一个 RequestManagerFragment, 其内部含有 ReqeustManager 对象, 以便后续直接根据 Activity 的生命周期管控 Glide 请求的处理
    - 若非可绑定 Activity, 则获取一个单例的 applicationManager 专门用于处理这类请求

## load
调用了 asDrawable 构建一个 RequestBuilder, 描述一个目标资源为 Drawable 的图片加载请求

## RequestBuilder.into
RequestBuilder.into 主要做了如下的事情

- 根据 ImageView 构建采样压缩和图像变化的策略保存在 Options 和 Transform 中
- 构建 ViewTarget, 描述这个请求要作用的 View 对象
- 调用 into 重载方法
    - 构建 Request 对象
    - 为 ViewTarget 绑定 Request
    - 交由 RequestManger 执行该请求

到这里我们的 Glide.with().load().into 就走完了, 最终会构建一个 Request 交由 RequestManger 分发执行

## RequestManager 执行 Request 请求
- 任务的构建
    - 构建这个请求的 key 
    - 从内存缓存中查找 key 对应的资源, 若存在直接回 onResourceReady 表示资源准备好了
        - 从 ActiveResources 缓存中查找
        - 从 LruResourceCache 缓存中查找
    - 从缓存中查找 key 对应的任务
        - 若存在则说明无需再次获取资源
    - 构建引擎任务 EngineJob
        - 引擎的任务为解码任务 DecodeJob
        - 将任务添加到缓存, 防止多次构建
        - 执行 EngineJob
- 任务的执行
    - 通过 getNextStage 获取 Decode 场景
    - 构建当前场景的处理器 DataFetcherGenerator
    - 调用 runGenerators 处理任务
        - 优先从 Resource 的磁盘缓存中取数据
        - 次优先从 Data 的磁盘缓存中取数据
        - 最终方案重新获取 Source 数据
    - Source 通过 HttpUrlFetcher 获取网络数据流 InputStream
    - DecodeJob 的 onDataFetcherReady 处理 InputStream
        - 解析 InputStream
            - 通过 Downsampler 将 InputStream 解析成 Bitmap
            - 调用 Transformation.transformation 对 Bitmap 进行变化操作
            - 将 Bitmap 转为 Drawable
        - 资源的缓存与展示
            - Engine 进行内存缓存
            - ViewTarget 展示资源
            - DecodeJob 进行磁盘缓存
                - 原始 Bitmap 对应 SOURCE 类型
                - Transform 之后的 Bitmap 对应 RESOURCE 类型



# 四级缓存
- 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
- 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
- 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
- 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

# OOM解决
- 重写application的onTrimMemory()和onLowMemory()，调用Glide的trimMemory() 和 cleanMemroy() 方法
- 配置 GlideModule

# 自定义transformation

# transition（过渡动画）
