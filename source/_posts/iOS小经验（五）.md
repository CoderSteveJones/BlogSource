---
title: iOS小经验（五）
date: 2016-03-14 09:53:06
categories: 
	- iOS小经验
---


#### 1、让UILabel在指定的地方换行
```objectivec
// 换行符为\n,在需要换行的地方加上这个符号即可，如 
label.numberOfLines = 0;
label.text = @"此处\n换行";
```

#### 2、摇一摇功能
```objectivec
1、打开摇一摇功能
    [UIApplication sharedApplication].applicationSupportsShakeToEdit = YES;
2、让需要摇动的控制器成为第一响应者
[self becomeFirstResponder];
3、实现以下方法

// 开始摇动
- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event
// 取消摇动
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event
// 摇动结束
- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event
```

#### 3、获取图片大小
```objectivec
CGFloat imageWidth = image.size.width;
CGFloat imageHeight = imageWidth * image.scale;
```

#### 4、获取view的坐标在整个window上的位置
```objectivec
// v上的(0, 0)点在toView上的位置
CGPoint point = [v convertPoint:CGPointMake(0, 0) toView:[UIApplication sharedApplication].windows.lastObject];
或者
CGPoint point = [v.superview convertPoint:v.frame.origin toView:[UIApplication sharedApplication].windows.lastObject];
```

#### 5、提交App Store审核程序限制
```objectivec
您的应用程序的未压缩大小必须小于4GB。每个Mach-O可执行文件（例如app_name.app/app_name）不能超过这些限制：
对于MinimumOSVersion小于7.0的应用程序：TEXT二进制文件中所有部分的总数最多为80 MB 。
对于MinimumOSVersion7.x到8.x的应用程序：TEXT对于二进制文件中每个体系结构片段的每个片段，最大为60 MB 。
对于MinimumOSVersion9.0或更高版本的应用程序：__TEXT二进制文件中所有部分的总数最多为500 MB 。参阅：iTunes Connect开发者指南
```

#### 6、修改UISegmentedControl的字体大小
```objectivec
[segment setTitleTextAttributes:@{NSFontAttributeName : [UIFont systemFontOfSize:15.0f]} forState:UIControlStateNormal];
```

#### 7、在非ViewController的地方弹出UIAlertController对话框
```objectivec
//  最好抽成一个分类
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
//...
id rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
if([rootViewController isKindOfClass:[UINavigationController class]])
{
    rootViewController = ((UINavigationController *)rootViewController).viewControllers.firstObject;
}
if([rootViewController isKindOfClass:[UITabBarController class]])
{
    rootViewController = ((UITabBarController *)rootViewController).selectedViewController;
}
[rootViewController presentViewController:alertController animated:YES completion:nil
```
#### 8、获取一个view所属的控制器
```objectivec
// view分类方法
- (UIViewController *)belongViewController {
    for (UIView *next = [self superview]; next; next = next.superview) {
        UIResponder* nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)nextResponder;
        }
    }
    return nil;
}
```

#### 9、UIImage和base64互转
```objectivec
// view分类方法
- (NSString *)encodeToBase64String:(UIImage *)image {
 return [UIImagePNGRepresentation(image) base64EncodedStringWithOptions:NSDataBase64Encoding64CharacterLineLength];
}

- (UIImage *)decodeBase64ToImage:(NSString *)strEncodeData {
  NSData *data = [[NSData alloc]initWithBase64EncodedString:strEncodeData options:NSDataBase64DecodingIgnoreUnknownCharacters];
  return [UIImage imageWithData:data];
}
```

#### 10、UIWebView设置背景透明
```objectivec
[webView setBackgroundColor:[UIColor clearColor]];
[webView setOpaque:NO];
```

#### 11、判断NSDate是不是今天
```objectivec
NSDateComponents *otherDay = [[NSCalendar currentCalendar] components:NSCalendarUnitEra | NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay fromDate:aDate];
NSDateComponents *today = [[NSCalendar currentCalendar] components:NSCalendarUnitEra | NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay fromDate:[NSDate date]];
if([today day] == [otherDay day] &&
   [today month] == [otherDay month] &&
   [today year] == [otherDay year] &&
   [today era] == [otherDay era]) {
    // 是今天
}
```

#### 12、设置tableView分割线颜色
```objectivec
[self.tableView setSeparatorColor:[UIColor myColor]];
```

#### 13、设置屏幕方向
```objectivec
 NSNumber *orientationTarget = [NSNumber numberWithInt:UIInterfaceOrientationLandscapeLeft];
[[UIDevice currentDevice] setValue:orientationTarget forKey:@"orientation"];
[UIViewController attemptRotationToDeviceOrientation];
```

#### 14、比较两个颜色是否相等
```objectivec
- (BOOL)isEqualToColor:(UIColor *)otherColor {
    CGColorSpaceRef colorSpaceRGB = CGColorSpaceCreateDeviceRGB();

    UIColor *(^convertColorToRGBSpace)(UIColor*) = ^(UIColor *color) {
        if (CGColorSpaceGetModel(CGColorGetColorSpace(color.CGColor)) == kCGColorSpaceModelMonochrome) {
            const CGFloat *oldComponents = CGColorGetComponents(color.CGColor);
            CGFloat components[4] = {oldComponents[0], oldComponents[0], oldComponents[0], oldComponents[1]};
            CGColorRef colorRef = CGColorCreate( colorSpaceRGB, components );

            UIColor *color = [UIColor colorWithCGColor:colorRef];
            CGColorRelease(colorRef);
            return color;            
        } else
            return color;
    };

    UIColor *selfColor = convertColorToRGBSpace(self);
    otherColor = convertColorToRGBSpace(otherColor);
    CGColorSpaceRelease(colorSpaceRGB);

    return [selfColor isEqual:otherColor];
}
```

#### 15、tableViewCell分割线顶到头
```objectivec
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
    [cell setSeparatorInset:UIEdgeInsetsZero];
    [cell setLayoutMargins:UIEdgeInsetsZero];
    cell.preservesSuperviewLayoutMargins = NO;
}

- (void)viewDidLayoutSubviews {
    [self.tableView setSeparatorInset:UIEdgeInsetsZero];
    [self.tableView setLayoutMargins:UIEdgeInsetsZero];
}
```

#### 16、不让控制器的view随着控制器的xib拉伸或压缩
```objectivec
self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
```

#### 17、`cocoaPods报错 : [!] Unable to add a source with url https://github.com/CocoaPods/Specs.git named master-1.You can try adding it manually in ~/.cocoapods/repos or via pod repo add.`

>解决方法：这是因为电脑里安装了另外一个Xcode导致cocoapods找不到路径了
在终端执行 sudo xcode-select -switch /Applications/Xcode.app 即可

#### 18、安装cocoapods的时候出现 ERROR: While executing gem ... (Errno::EPERM)
```
Operation not permitted - /usr/bin/pod
```

#### 19、在状态栏增加网络请求的菊花，类似safari加载网页的时候状态栏菊花
```objectivec
[UIApplication sharedApplication].networkActivityIndicatorVisible = YES;
```

#### 20、检查一个rect是否包含一个point
```objectivec
// point是否在rect内
BOOL isContains = CGRectContainsPoint(rect, point);
```

#### 21、在指定的宽度下，让UILabel自动设置最佳font
```objectivec
label.adjustsFontSizeToFitWidth = YES;
```

#### 22、将一个image保存在相册中
```objectivec
UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil);

或者
#import <Photos/Photos.h>
[[PHPhotoLibrary sharedPhotoLibrary] performChanges:^{
        PHAssetChangeRequest *changeRequest = [PHAssetChangeRequest creationRequestForAssetFromImage:image];
        changeRequest.creationDate          = [NSDate date];
    } completionHandler:^(BOOL success, NSError *error) {
        if (success) {
            NSLog(@"successfully saved");
        }
        else {
            NSLog(@"error saving to photos: %@", error);
        }
    }];

```

#### 23、修改cell.imageView的大小
```objectivec
UIImage *icon = [UIImage imageNamed:@""];
CGSize itemSize = CGSizeMake(30, 30);
UIGraphicsBeginImageContextWithOptions(itemSize, NO ,0.0);
CGRect imageRect = CGRectMake(0.0, 0.0, itemSize.width, itemSize.height);
[icon drawInRect:imageRect];
cell.imageView.image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
```

#### 24、为一个view添加虚线边框
```objectivec
 CAShapeLayer *border = [CAShapeLayer layer];
    border.strokeColor = [UIColor colorWithRed:67/255.0f green:37/255.0f blue:83/255.0f alpha:1].CGColor;
    border.fillColor = nil;
    border.lineDashPattern = @[@4, @2];
    border.path = [UIBezierPath bezierPathWithRect:view.bounds].CGPath;
    border.frame = view.bounds;
    [view.layer addSublayer:border];
```

#### 25、UITextView中打开或禁用复制，剪切，选择，全选等功能
```objectivec
// 继承UITextView重写这个方法
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender
{
// 返回NO为禁用，YES为开启
    // 粘贴
    if (action == @selector(paste:)) return NO;
    // 剪切
    if (action == @selector(cut:)) return NO;
    // 复制
    if (action == @selector(copy:)) return NO;
    // 选择
    if (action == @selector(select:)) return NO;
    // 选中全部
    if (action == @selector(selectAll:)) return NO;
    // 删除
    if (action == @selector(delete:)) return NO;
    // 分享
    if (action == @selector(share)) return NO;
    return [super canPerformAction:action withSender:sender];
}

```

#### 26、设置UILabel行间距
```objectivec
NSMutableAttributedString* attrString = [[NSMutableAttributedString  alloc] initWithString:label.text];
    NSMutableParagraphStyle *style = [[NSMutableParagraphStyle alloc] init];
    [style setLineSpacing:20];
    [attrString addAttribute:NSParagraphStyleAttributeName value:style range:NSMakeRange(0, label.text.length)];
    label.attributedText = attrString;
```
![](http://upload-images.jianshu.io/upload_images/1432270-736756a6216a6753.gif?imageMogr2/auto-orient/strip)

#### 27、当使用-performSelector:withObject:withObject:afterDelay:方法时，需要传入多参数问题
```objectivec
// 方法一、
// 把参数放进一个数组／字典，直接把数组／字典当成一个参数传过去，具体方法实现的地方再解析这个数组／字典
NSArray * array = 
    [NSArray arrayWithObjects: @"first", @"second", nil];
[self performSelector:@selector(fooFirstInput:) withObject: array afterDelay:15.0];

// 方法二、
// 使用NSInvocation
SEL aSelector = NSSelectorFromString(@"doSoming:argument2:");
    NSInteger argument1 = 10;
    NSString *argument2 = @"argument2";
    if([self respondsToSelector:aSelector]) {
        NSInvocation *inv = [NSInvocation invocationWithMethodSignature:[self methodSignatureForSelector:aSelector]];
        [inv setSelector:aSelector];
        [inv setTarget:self];
        [inv setArgument:&(argument1) atIndex:2];
        [inv setArgument:&(argument2) atIndex:3];
        [inv performSelector:@selector(invoke) withObject:nil afterDelay:15.0];
    }
```

#### 28、UILabel显示不同颜色字体
```objectivec
NSMutableAttributedString * string = [[NSMutableAttributedString alloc] initWithString:label.text];
[string addAttribute:NSForegroundColorAttributeName value:[UIColor redColor] range:NSMakeRange(0,5)];
[string addAttribute:NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(5,6)];
[string addAttribute:NSForegroundColorAttributeName value:[UIColor blueColor] range:NSMakeRange(11,5)];
label.attributedText = string;
```

#### 29、比较两个CGRect/CGSize/CGPoint是否相等
```objectivec
if (CGRectEqualToRect(rect1, rect2)) { // 两个区域相等
        // do some
    }
    if (CGPointEqualToPoint(point1, point2)) { // 两个点相等
        // do some
    }
    if (CGSizeEqualToSize(size1, size2)) { // 两个size相等
        // do some
    }
```

#### 30、比较两个NSDate相差多少小时
```objectivec
 NSDate* date1 = someDate;
 NSDate* date2 = someOtherDate;
 NSTimeInterval distanceBetweenDates = [date1 timeIntervalSinceDate:date2];
 double secondsInAnHour = 3600;
// 除以3600是把秒化成小时，除以60得到结果为相差的分钟数
 NSInteger hoursBetweenDates = distanceBetweenDates / secondsInAnHour;
```

