# Xlog
> fork Tencent/mars/Xlog（1.2.6版本）

## 背景
改造公司现有日志库，提升其性能指标(主要是CPU、内存)，调研了目前开源项目mmap的代码方案（log4a、xlog、logan），最终选择基于Xlog进行修改

## 目前公司日志库的实现方案
* 日志打印 
  * 使用的原生android.util.Log的api
* 本地日记存储
  * 使用Java_IO的方式，开启线程实时记录日志到本地

### 缺点
* 原生android.util.Log的api打印日志，会频繁触发GC，造成内存抖动
* Java_IO的方式，CPU占用高、内存占用
* 具体性能测试数据，见此文章

## mmap开源方案选型以及对比
1. 性能对比(含测试方法)，见此文章
2. 从源码角度进行方案选型，见此文章

## 定制化需求
1. 因为我们目前C端的产品，是我们自己的硬件设备，开发同学期望能直接通过nginx在线直接查看日志，故需要去掉python解密日志这一步骤
2. 提供jni函数，直接使用__android_log_write，解决"在高qps下，android.util.Log的api打印日志，内存抖动问题"

## Xlog修改点
1. 去掉了日志压缩
2. 解决日志不加密，但直接查看乱码的问题
3. 增加__android_log_write的jni函数，以及Java API， 上层可以直接调用，避免使用原生android.util.Log的api

## TODO 未完成点
1. log to mmap（目前是在主线程完成的）， 修改在线程里面实现， 可以提升函数线程执行时间， 尽量不占用主线程资源
2. mmap to file， 增加分段写（类似美团Logan的实现）， 主要好处是，可以避免存储交大日志文件时， 避免cpu出现峰值，让cpu表现，尽可能平滑