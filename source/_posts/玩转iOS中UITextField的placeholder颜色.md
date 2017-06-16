---
title: 玩转iOS中UITextField的placeholder颜色
date: 2017-05-23 13:59:59
categories: 
	- iOS合集
---
![珍惜时间](http://oqepgj2jp.bkt.clouddn.com/wallpaper-2572384.jpg)
>`UITextField`是iOS开发中经常使用到的控件，它有一个`placeholder`属性，也就是占位文字。默认占位文字颜色是` 70% gray`,但有时我们可能需要修改其占位文字的颜色，下文中将为大家介绍三中修改方法,并就动态改变颜色做相关说明（关于动态改变：当UITextField为第一响应者时为一种颜色，取消第一响应者时为另一种颜色）。

##方法1
- 设置 `attributedPlaceholder`属性
- 说明：此种方式对于无需动态改变`placeholder`颜色较为方便。

---
- 代码1（单色）

```objectivec
NSMutableDictionary *attrs = [NSMutableDictionary dictionary]; // 创建属性字典
  attrs[NSFontAttributeName] = [UIFont systemFontOfSize:17]; // 设置font
  attrs[NSForegroundColorAttributeName] = [UIColor greenColor]; // 设置颜色
  NSAttributedString *attStr = [[NSAttributedString alloc] initWithString:@"夏虫不可以语冰" attributes:attrs]; // 初始化富文本占位字符串
  self.textField.attributedPlaceholder = attStr;
```
- 效果1
![单色placeholder](http://upload-images.jianshu.io/upload_images/2115041-b0b505d18f2888a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 代码2（复色）

```objectivec
NSMutableAttributedString *attStr = [[NSMutableAttributedString alloc] initWithString:@"夏虫不可以语冰"];
 [attStr setAttributes:@{NSForegroundColorAttributeName : [UIColor redColor],
                            NSFontAttributeName : [UIFont systemFontOfSize:15.0]} range:NSMakeRange(0, 2)];
 [attStr setAttributes:@{NSForegroundColorAttributeName : [UIColor greenColor],
                            NSFontAttributeName : [UIFont systemFontOfSize:17.0]} range:NSMakeRange(2, 3)];
 [attStr setAttributes:@{NSForegroundColorAttributeName : [UIColor blueColor],
                            NSFontAttributeName : [UIFont systemFontOfSize:15.0]} range:NSMakeRange(5, 2)];
    self.textField.attributedPlaceholder = attStr;
```
- 效果
![复色placeholder](http://upload-images.jianshu.io/upload_images/2115041-d9430dd8f3e57343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##方法2
- 自定义UITextField,重写`- (void)drawPlaceholderInRect:(CGRect)rect;`
- 说明：此种方式只能设置一次状态,不能动态的改变`placeholder`的颜色,但可以设置`placeholder`所在位置。

---
- 代码

```objectivec
- (void)drawPlaceholderInRect:(CGRect)rect
{
    [self.placeholder drawInRect:CGRectMake(0, 2, rect.size.width, 25) withAttributes:@{ NSFontAttributeName: [UIFont systemFontOfSize:16.0],
                                        NSForegroundColorAttributeName : [UIColor blueColor],
                                     }];
}
```
- 效果
![placeholder](http://upload-images.jianshu.io/upload_images/2115041-cf46f67c604c2c9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##方法3
- 自定义UITextField,利用runTime找出UITextFiled内部隐藏的成员变量和属性，利用KVC进行修改。
- 说明：此种方式对于动态改变`placeholder`颜色较为方便。

- **拓展代码**（利用runTime找出成员变量和属性，程序中无需使用，只是帮助我们看清UITextField内部结构，知道其中的相关成员变量和属性，然后赋值即可）。

```objectivec
#import "SJTextField.h"
#import <objc/runtime.h>
@implementation SJTextField

// 初始化调用一次 用于查看UITextField中的成员属性和变量
+ (void)initialize
{
    [self getIvars];
    // [self getProperties];
}

// 获取所有属性
+ (void)getProperties
{
    unsigned int count = 0;
    objc_property_t *properties = class_copyPropertyList([UITextField class], &count);
    for (int i = 0; i<count; i++) {
        // 取出属性
        objc_property_t property = properties[i];
        
        // 打印属性名字
        NSLog(@"%s<---->%s", property_getName(property), property_getAttributes(property));
    }
    free(properties);
}

// 获取所有成员变量
+ (void)getIvars
{
    unsigned int count = 0;
    // 拷贝出所有的成员变量列表
    Ivar *ivars = class_copyIvarList([UITextField class], &count);
    for (int i = 0; i<count; i++) {
        // 取出成员变量
        //        Ivar ivar = *(ivars + i);
        Ivar ivar = ivars[i];
        
        // 打印成员变量名字
        NSLog(@"%s %s", ivar_getName(ivar), ivar_getTypeEncoding(ivar));
    }
    // 释放
    free(ivars);
}
@end
```
- **拓展点** 可以查到有两个关于`placeholder`的属性和变量，分别是`_placeholderLabel.textColor`和`_placeholderLabel`，故下面就是用来设置动态改变`placeholder`颜色的代码。
- 代码（在自定义UITextField中）

```objectivec
#import "SJTextField.h"
#import <objc/runtime.h>
@implementation SJTextField
static NSString * const SJPlacerholderColorKeyPath = @"_placeholderLabel.textColor";
- (void)awakeFromNib
{
// 设置placeholder开始颜色（方式一）
//    UILabel *placeholderLabel = [self valueForKeyPath:@"_placeholderLabel"];
//    placeholderLabel.textColor = [UIColor redColor];   
   // 设置placeholder开始颜色（方式二）
    [self setValue:[UIColor greenColor] forKeyPath:SJPlacerholderColorKeyPath];
    // 不成为第一响应者
    [self resignFirstResponder];
}

/**
 * 当前文本框聚焦时就会调用
 */
- (BOOL)becomeFirstResponder
{
    // 修改占位文字颜色
    [self setValue:[UIColor redColor] forKeyPath:SJPlacerholderColorKeyPath];
    return [super becomeFirstResponder];
}

/**
 * 当前文本框失去焦点时就会调用
 */
- (BOOL)resignFirstResponder
{
    // 修改占位文字颜色
    [self setValue:[UIColor greenColor] forKeyPath:SJPlacerholderColorKeyPath];
    return [super resignFirstResponder];
}
@end
```
- 效果


![未进入编辑状态](http://upload-images.jianshu.io/upload_images/2115041-9c0ca6ff0ec7c261.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![进入编辑状态](http://upload-images.jianshu.io/upload_images/2115041-6c9aacd3d30f3655.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


End.


