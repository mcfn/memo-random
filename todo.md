# Memo todo-list



## About Core Foundation
http://limboy.me/ios/2013/06/07/core-foundation.html

这里就牵扯到了一个问题，如何让旧有的系统（Mac OS 9）和NeXTSTEP合成为一个新系统？这就需要一个更为底层的核心库可以供Mac Toolbox和OPENSTEP双方调用。CF就这么诞生了。

CF是由C语言实现的，而不是Objective-C，所以如果用到了CF，就需要手动管理内存，ARC是无能为力的。当然因为CF和Foundation之间的友好关系，它们之间的管理权也是可以移交的，这个后面再说。

CF提供了基础功能，如CFString,CFDate,CFNumber等等，以CFString为例，CFString和NSString之间是什么关系？NSString其实是一个「类簇」，也就是抽象接口，所以String Objects并不是NSString实例，而是实现了NSString方法的私有类的实例，也就是CFString。

同时NSStrings和CFStrings之间可以自由转换，也就是「toll free bridging」。比如：
```objc
CFStringRef aCFString = (CFStringRef)aNSString;  
NSString *aNSString = (NSString *)aCFString;
```

因为编译器无法自动管理CF的内存，所以CF对象在使用完后，需要手动释放（CFRelease）。如果使用ARC来管理内存，苹果提供了3种方法来处理：




# MRC -> ARC

## dispatch_retain
After ios6, ARC will manage you queue for you.

http://stackoverflow.com/questions/8618632/does-arc-support-dispatch-queues
##
