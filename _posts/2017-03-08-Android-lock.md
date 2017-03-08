---
layout: post
title: "Android应用锁"
categories: android
tags: [documentation,android]
image:
  feature: forest.jpg
  teaser: forest-teaser.jpg
  credit: Death to Stock Photo
  creditlink: 
---
出于隐私保护的需求，应用置于后台立刻或一段时间后返回，需要加锁处理。  

### 实现原理
  将应用可见可操作时为前台状态(foreground)，不可见视为后台状态(backround)，从前台切回后台时，显示锁界面。 <br/>
##### 状态切换的场景
1. 按home键返回桌面<br/>
2. 按back键返回桌面或者退出<br/>
3. 按任务键切换到其他app<br/>
4. 当前app下手机锁屏黑屏<br/>

返回前台时，在Activity的生命周期onReusme上锁
    
    @Override
    protected void onResume() {
        super.onResume();
        if(!mIsForeground){
            lock();
        }
    }

home键退回桌面

    @Override
    public void onTrimMemory(int level) {
        super.onTrimMemory(level);
        if (level >= ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN) {
            mIsForeground = false;
        }
    }

back键退出，mIsForeground初始值为false，显示闪屏页后即可lock()  
back键返回桌面，使用moveTaskToBack，处理同home键返回  
任务键切换到其他app，可在onStop和onTrimMemory中判断
锁屏黑屏，动态注册广播监听Intent.ACTION_SCREEN_OFF


  
### 注意事项
##### 1.多个进程
自身app使用到了多进程，Activity指定了process，这时前后不同进程的Activity切换，需要判断当前RunningTask是否是你的进程。
##### 2.通知栏点进入app  
展示app时，接收到新通知，如微信消息，跳转至微信后返回，由于判断当前进程方法无效，暂时无法解决，参考手机QQ(V6.6.93060)锁屏暂未实现。
##### 3.图库跳转
本质上类似多个进程，由于RunningTask不好判断进程名字，无法确定状态。所以在onActivityResult先解除锁，在onResume恢复锁。避免选图或者裁剪图片时返回上锁的问题。

#### 参考资料
[http://blog.csdn.net/lsmfeixiang/article/details/42212979](http://blog.csdn.net/lsmfeixiang/article/details/42212979 "仿QQ黑屏，锁屏，程序切换之后的手势密码锁定，加强版")  
[https://github.com/wenmingvs/AndroidProcess](https://github.com/wenmingvs/AndroidProcess "判断App位于前台或者后台的6种方法")  
[http://blog.csdn.net/u010983881/article/details/48549123](http://blog.csdn.net/u010983881/article/details/48549123 "onActivityResult和onResume的调用顺序问题")