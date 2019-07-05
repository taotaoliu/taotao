---
title: iOS-RunLoop
date: 2019-06-5 14:01:15
tags: iOS
---

Objective-C 版本 RunLoop 源码：
[https://opensource.apple.com/source/CF/CF-635.19/CFRunLoop.c.auto.html](https://opensource.apple.com/source/CF/CF-635.19/CFRunLoop.c.auto.html)

Swift 版本 RunLoop 源码：
[https://github.com/apple/swift-corelibs-foundation/blob/master/CoreFoundation/RunLoop.subproj/CFRunLoop.c](https://github.com/apple/swift-corelibs-foundation/blob/master/CoreFoundation/RunLoop.subproj/CFRunLoop.c)

## 1. RunLoop 概念

运行循环，在程序运行过程中循环处理事件，如果没有 Runloop 程序执行完毕就会立即退出，Runloop 使程序能够一直运行，并且能时时刻刻处理用户的输入操作。RunLoop 可以在需要的时候自己运行，在没有操作的时候停下，充分节省 CPU 资源，提高程序性能。

### (1) Event Loop
#### 1) 概念
在计算机科学里， Event Loop / Run Loop 是一个用于等待和发送消息/事件的程序结构，在程序中等待和派发一系列事件或者消息。它通过向“事件提供者”发出请求来工作，通常会阻塞请求，直到事件到达，然后调用相应的事件处理程序。
#### 2) Event Driven（事件驱动）
为了解决图形界面和用户交互的问题便出现了 Event Driven。通常 GUI 程序的交互事件执行是由用户来控制的，无法预测它发生的节点，对应这样的情况，需要采用 Event-Driven 的编程方法。流程如：启动 ——> 事件循环（即等待事件发生并处理之）。
#### 3) Event Handler
当 Event（事件） 被放到 Event Loop 里进行处理的时候，会调用预先注册过的代码对 Event（事件） 做处理或者回调，这一处理回调就叫做 Event Handler 。对应的一个 Event 并不一定有 Event Handler 。
#### 4) Event Loop 解决的问题
处理事件一般来说分为同步和异步，Event Loop 就是实现了异步处理事件的一种方法。对于有 Event Handler 的 Event 线程不用等待任务的结果出来再去执行下一个，而是等待 Event 被加入到 Event Loop 的时候再去执行，如果一个 Event 也没有，线程就会休眠，避免浪费资源。如果没有 Event Loop 实现异步操作，用户操作就会出现卡顿。
### (2) RunLoop 与线程
1) 每个线程都有唯一的一个与之对应的 RunLoop 对象。
2) RunLoop 保存在一个全局的 Dictionary 里，线程作为 key , RunLoop 作为 value 。
3) 主线程的 RunLoop 已经自动创建好了，子线程的 RunLoop 需要主动创建。
4) RunLoop 在第一次获取时创建，在线程结束时销毁。

上述关系可从下列源代码中看到:
```
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    // 拿到当前 Runloop 调用 _CFRunLoopGet0
    return _CFRunLoopGet0(pthread_self());
}

static CFMutableDictionaryRef __CFRunLoops = NULL;   
static CFSpinLock_t loopsLock = CFSpinLockInit;

// should only be called by Foundation
// t==0 is a synonym for "main thread" that always works
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {
	    t = _CFMainPThread;
    }
    __CFSpinLock(&loopsLock);
    if (!__CFRunLoops) {
        __CFSpinUnlock(&loopsLock);
	    CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 根据传入的主线程获取主线程对应的 RunLoop
	    CFRunLoopRef mainLoop = __CFRunLoopCreate(_CFMainPThread);
         // 保存主线程，将主线程作为 key 和 RunLoop 作为 value 保存到字典中
	    CFDictionarySetValue(dict, pthreadPointer(_CFMainPThread), mainLoop);
	    if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
	        CFRelease(dict);
	    }
	    CFRelease(mainLoop);
            __CFSpinLock(&loopsLock);
    }
    // 从字典里面拿，将线程作为 key 从字典里获取一个 loop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    // 如果 loop 为空，则创建一个新的 loop，所以 Runloop 会在第一次获取的时候创建
    if (!loop) {
        __CFSpinUnlock(&loopsLock);
	    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFSpinLock(&loopsLock);
	    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        // 创建好之后，以线程为 key, Runloop 为 value，一对一存储在字典中，下次获取的时候，则直接返回字典内的 Runloop
	    if (!loop) {
	        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
	        loop = newLoop;
	    }
	    CFRelease(newLoop);
    }
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    __CFSpinUnlock(&loopsLock);
    return loop;
}
```
## 2. RunLoop 结构

### (1) RunLoop Mode

### (2) RunLoop Source

### (3) RunLoop Timer

### (4) RunLoop Observer

## 3. RunLoop 流程

## 3. RunLoop 实现

## 4. RunLoop 应用