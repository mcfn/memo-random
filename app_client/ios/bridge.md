
# Toll-Free Bridged Types
https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html

## Casting and Object Lifetime Semantics
The compiler does not automatically manage the lifetimes of Core Foundation objects. You tell the compiler about the ownership semantics of objects using either a cast (defined in objc/runtime.h) or a Core Foundation-style macro (defined in NSObject.h):

* __bridge transfers a pointer between Objective-C and Core Foundation with no transfer of ownership.
* __bridge_retained or CFBridgingRetain casts an Objective-C pointer to a Core Foundation pointer and also transfers ownership to you.  
You are responsible for calling CFRelease or a related function to relinquish ownership of the object.
* __bridge_transfer or CFBridgingRelease moves a non-Objective-C pointer to Objective-C and also transfers ownership to ARC.  
ARC is responsible for relinquishing ownership of the object.


http://blog.csdn.net/yiyaaixuexi/article/details/8553659
* `__bridge`  （不改变对象所有权）
* `__bridge_retained / CFBridgingRetain()`  （解除 ARC 所有权）
* `__bridge_transfer / CFBridgingRelease()`  （给予 ARC 所有权）

### `__bridge_retained`

__bridge_retained 或者 CFBridgingRetain()  将Objective-C对象转换为Core Foundation对象，把对象所有权桥接给Core Foundation对象，同时剥夺ARC的管理权，后续需要开发者使用CFRelease或者相关方法手动来释放对象。

```objc
- (void)viewDidLoad  
{  
    [super viewDidLoad];  

    NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];  
    CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;  

    (void)aCFString;  

    //正确的做法应该执行CFRelease  
    //CFRelease(aCFString);
}  
```
CFBridgingRetain()  是 __bridge_retained 的宏方法，下面两行代码等价：
```objc
aNSString = (__bridge_transfer NSString *)aCFString;  
aNSString = (NSString *)CFBridgingRelease(aCFString);  
```

### `__bridge`
__bridge 只做类型转换，不改变对象所有权，是我们最常用的转换符。

从OC转CF，ARC管理内存：
```objc
- (void)viewDidLoad  
{  
    [super viewDidLoad];  

    NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];  
    CFStringRef aCFString = (__bridge CFStringRef)aNSString;  

    (void)aCFString;  
}  
```
从CF转OC，需要开发者手动释放，不归ARC管：
```objc
- (void)viewDidLoad  
{  
    [super viewDidLoad];  

    CFStringRef aCFString = CFStringCreateWithCString(NULL, "test", kCFStringEncodingASCII);  
    NSString *aNSString = (__bridge NSString *)aCFString;  

    (void)aNSString;  

    CFRelease(aCFString);  
}  
```
