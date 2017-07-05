---
title: iOS消息转发机制-- 动态方法解析实现@dynamic
date: 2017-02-02 11:31:09
categories: 
    - iOS合集
---

> 假设要编写一个类似于“字典”的对象，它里面可以容纳其他对象，只不过开发者要通过属性来存取其中的数据，这个类的设计思路：由开发者来添加属性定义，并将其申明为@dynamic,而类则会自动处理相关属性值的存放和获取操作。

#### 实现代码：

`.h文件`

```objectivec
#import <Foundation/Foundation.h>

@interface MyDictionary : NSObject
@property (nonatomic, strong) NSDate *date;
@end
```

> 本例中，属性具体设置什么无关紧要，只是掩饰此功能。在类的内部，每个属性的值还会存放在字典中，所以先在类中编写如下代码，并将属性申明为@dynamic,这样编译器就不会为其自动生成实例变量的存取方法了。


`.m文件`

```objectivec
#import "MyDictionary.h"
#import <objc/runtime.h>

@interface MyDictionary ()
@property (nonatomic, strong) NSMutableDictionary *storeDictionary;

@end

@implementation MyDictionary

@dynamic date;

- (instancetype)init
{
    if (self = [super init]) {
        _storeDictionary = [NSMutableDictionary dictionary];
    }
    return self;
}

// 通过消息转移实现运行时动态添加get 和 set 方法
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    NSString *selectorStr = NSStringFromSelector(sel);
    if ([selectorStr hasPrefix:@"set"]) {
        
        class_addMethod(self, sel, (IMP)autoDictionarySetter, "v@:@");
        
    }else{
        
        class_addMethod(self, sel, (IMP)autoDictionaryGetter, "@@:");
    }
    return YES;
}

// get 方法的实现
id autoDictionaryGetter(id self, SEL _cmd) {
    MyDictionary *typeSelf = (MyDictionary *)self;
    NSMutableDictionary *backingStore = typeSelf.storeDictionary;
    
    // the key is simply the selector name
    NSString *key = NSStringFromSelector(_cmd);
    
    // return the value
    return [backingStore objectForKey:key];
}

// set 方法的实现
void autoDictionarySetter(id self, SEL _cmd, id value){
    // get the backstore from the object
    MyDictionary *typeSelf = (MyDictionary *)self;
    NSMutableDictionary *backingStore = typeSelf.storeDictionary;
    
    // get method name from _cmd
    NSString *selectorString = NSStringFromSelector(_cmd);
    NSMutableString *key = [selectorString mutableCopy];
    
    // remove ":" at the end
    [key deleteCharactersInRange:NSMakeRange(key.length - 1, 1)];
    
    // remove "set" at the prefix
    [key deleteCharactersInRange:NSMakeRange(0, 3)];
    
    // lowercase the first charactor
    NSString *firstLowercaseString = [[key substringToIndex:1] lowercaseString];
    [key replaceCharactersInRange:NSMakeRange(0, 1) withString:firstLowercaseString];
    
    // set value
    if (value) {
        
        [backingStore setObject:value forKey:key];
    }else{
        [backingStore removeObjectForKey:key];
    }
    
    
}
@end
```

#### 调用：


```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyDictionary *dic = [[MyDictionary alloc] init];
    dic.date = [NSDate date];
    NSLog(@"date = %@",dic.date);
}
```

#### 输出：
```objectivec
date = 2017-07-02 03:26:08 +0000
```

