---
title: iOS小经验（三）
date: 2016-02-18 09:53:06
categories: 
	- 小经验
---

#### 1、拿到当前正在显示的控制器，不管是push进去的，还是present进去的都能拿到
```objectivec
- (UIViewController *)getVisibleViewControllerFrom:(UIViewController*)vc {
    if ([vc isKindOfClass:[UINavigationController class]]) {
        return [self getVisibleViewControllerFrom:[((UINavigationController*) vc) visibleViewController]];
    }else if ([vc isKindOfClass:[UITabBarController class]]){
        return [self getVisibleViewControllerFrom:[((UITabBarController*) vc) selectedViewController]];
    } else {
        if (vc.presentedViewController) {
            return [self getVisibleViewControllerFrom:vc.presentedViewController];
        } else {
            return vc;
        }
    }
}
```

#### 2、runtime为一个类动态添加属性
```objectivec
// 动态添加属性的本质是: 让对象的某个属性与值产生关联
        objc_setAssociatedObject(self, WZBPlaceholderViewKey, placeholderView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
```

#### 3、获取runtime为一个类动态添加的属性
```objectivec
objc_getAssociatedObject(self, WZBPlaceholderViewKey);
```

#### 4、KVO监听某个对象的属性
```objectivec
// 添加监听者
[self addObserver:self forKeyPath:property options:NSKeyValueObservingOptionNew context:nil];

// 当监听的属性值变化的时候会来到这个方法
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if ([keyPath isEqualToString:@"property"]) {
       [self textViewTextChange];
       } else {
     }
}
```

#### 5、Reachability判断网络状态
```objectivec
NetworkStatus status = [[Reachability reachabilityForInternetConnection] currentReachabilityStatus];
    if (status == NotReachable) {
        NSLog(@"当前设备无网络");
    }
    if (status == ReachableViaWiFi) {
        NSLog(@"当前wifi网络");
    }
    if (status == ReachableViaWWAN) {
        NSLog(@"当前蜂窝移动网络");
    }
```

#### 6、AFNetworking监听网络状态
```objectivec
// 监听网络状况
    AFNetworkReachabilityManager *mgr = [AFNetworkReachabilityManager sharedManager];
    [mgr setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        switch (status) {
            case AFNetworkReachabilityStatusUnknown:
                break;
            case AFNetworkReachabilityStatusNotReachable: {
                [SVProgressHUD showInfoWithStatus:@"当前设备无网络"];
            }
                break;
            case AFNetworkReachabilityStatusReachableViaWiFi:
                [SVProgressHUD showInfoWithStatus:@"当前Wi-Fi网络"];
                break;
            case AFNetworkReachabilityStatusReachableViaWWAN:
                [SVProgressHUD showInfoWithStatus:@"当前蜂窝移动网络"];
                break;
            default:
                break;
        }
    }];
    [mgr startMonitoring];
```

#### 7、透明颜色不影响子视图透明度
```objectivec
[UIColor colorWithRed:<#(CGFloat)#> green:<#(CGFloat)#> blue:<#(CGFloat)#> alpha:<#(CGFloat)#>];
```

#### 8、取图片某一点的颜色
```objectivec
    if (point.x < 0 || point.y < 0) return nil;

    CGImageRef imageRef = self.CGImage;
    NSUInteger width = CGImageGetWidth(imageRef);
    NSUInteger height = CGImageGetHeight(imageRef);
    if (point.x >= width || point.y >= height) return nil;

    unsigned char *rawData = malloc(height * width * 4);
    if (!rawData) return nil;

    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    NSUInteger bytesPerPixel = 4;
    NSUInteger bytesPerRow = bytesPerPixel * width;
    NSUInteger bitsPerComponent = 8;
    CGContextRef context = CGBitmapContextCreate(rawData,
                                                 width,
                                                 height,
                                                 bitsPerComponent,
                                                 bytesPerRow,
                                                 colorSpace,
                                                 kCGImageAlphaPremultipliedLast
                                                 | kCGBitmapByteOrder32Big);
    if (!context) {
        free(rawData);
        return nil;
    }
    CGColorSpaceRelease(colorSpace);
    CGContextDrawImage(context, CGRectMake(0, 0, width, height), imageRef);
    CGContextRelease(context);

    int byteIndex = (bytesPerRow * point.y) + point.x * bytesPerPixel;
    CGFloat red   = (rawData[byteIndex]     * 1.0) / 255.0;
    CGFloat green = (rawData[byteIndex + 1] * 1.0) / 255.0;
    CGFloat blue  = (rawData[byteIndex + 2] * 1.0) / 255.0;
    CGFloat alpha = (rawData[byteIndex + 3] * 1.0) / 255.0;

    UIColor *result = nil;
    result = [UIColor colorWithRed:red green:green blue:blue alpha:alpha];
    free(rawData);
    return result;
```

#### 9、判断该图片是否有透明度通道
```objectivec
  - (BOOL)hasAlphaChannel
{
    CGImageAlphaInfo alpha = CGImageGetAlphaInfo(self.CGImage);
    return (alpha == kCGImageAlphaFirst ||
            alpha == kCGImageAlphaLast ||
            alpha == kCGImageAlphaPremultipliedFirst ||
            alpha == kCGImageAlphaPremultipliedLast);
}
```

#### 10、获得灰度图
```objectivec
+ (UIImage*)covertToGrayImageFromImage:(UIImage*)sourceImage
{
    int width = sourceImage.size.width;
    int height = sourceImage.size.height;

    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceGray();
    CGContextRef context = CGBitmapContextCreate (nil,width,height,8,0,colorSpace,kCGImageAlphaNone);
    CGColorSpaceRelease(colorSpace);

    if (context == NULL) {
        return nil;
    }

    CGContextDrawImage(context,CGRectMake(0, 0, width, height), sourceImage.CGImage);
    CGImageRef contextRef = CGBitmapContextCreateImage(context);
    UIImage *grayImage = [UIImage imageWithCGImage:contextRef];
    CGContextRelease(context);
    CGImageRelease(contextRef);

    return grayImage;
}
```

#### 11、根据bundle中的文件名读取图片
```objectivec
   + (UIImage *)imageWithFileName:(NSString *)name {
    NSString *extension = @"png";

    NSArray *components = [name componentsSeparatedByString:@"."];
    if ([components count] >= 2) {
        NSUInteger lastIndex = components.count - 1;
        extension = [components objectAtIndex:lastIndex];

        name = [name substringToIndex:(name.length-(extension.length+1))];
    }

    // 如果为Retina屏幕且存在对应图片，则返回Retina图片，否则查找普通图片
    if ([UIScreen mainScreen].scale == 2.0) {
        name = [name stringByAppendingString:@"@2x"];

        NSString *path = [[NSBundle mainBundle] pathForResource:name ofType:extension];
        if (path != nil) {
            return [UIImage imageWithContentsOfFile:path];
        }
    }

    if ([UIScreen mainScreen].scale == 3.0) {
        name = [name stringByAppendingString:@"@3x"];

        NSString *path = [[NSBundle mainBundle] pathForResource:name ofType:extension];
        if (path != nil) {
            return [UIImage imageWithContentsOfFile:path];
        }
    }

    NSString *path = [[NSBundle mainBundle] pathForResource:name ofType:extension];
    if (path) {
        return [UIImage imageWithContentsOfFile:path];
    }

    return nil;
}
```

#### 12、合并两个图片
```objectivec
+ (UIImage*)mergeImage:(UIImage*)firstImage withImage:(UIImage*)secondImage {
    CGImageRef firstImageRef = firstImage.CGImage;
    CGFloat firstWidth = CGImageGetWidth(firstImageRef);
    CGFloat firstHeight = CGImageGetHeight(firstImageRef);
    CGImageRef secondImageRef = secondImage.CGImage;
    CGFloat secondWidth = CGImageGetWidth(secondImageRef);
    CGFloat secondHeight = CGImageGetHeight(secondImageRef);
    CGSize mergedSize = CGSizeMake(MAX(firstWidth, secondWidth), MAX(firstHeight, secondHeight));
    UIGraphicsBeginImageContext(mergedSize);
    [firstImage drawInRect:CGRectMake(0, 0, firstWidth, firstHeight)];
    [secondImage drawInRect:CGRectMake(0, 0, secondWidth, secondHeight)];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```

#### 13、根据bundle中的图片名创建imageview
```objectivec
+ (id)imageViewWithImageNamed:(NSString*)imageName
{
    return [[UIImageView alloc] initWithImage:[UIImage imageNamed:imageName]];
}
```

#### 14、为imageView添加倒影
```objectivec
    CGRect frame = self.frame;
    frame.origin.y += (frame.size.height + 1);

    UIImageView *reflectionImageView = [[UIImageView alloc] initWithFrame:frame];
    self.clipsToBounds = TRUE;
    reflectionImageView.contentMode = self.contentMode;
    [reflectionImageView setImage:self.image];
    reflectionImageView.transform = CGAffineTransformMakeScale(1.0, -1.0);

    CALayer *reflectionLayer = [reflectionImageView layer];

    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.bounds = reflectionLayer.bounds;
    gradientLayer.position = CGPointMake(reflectionLayer.bounds.size.width / 2, reflectionLayer.bounds.size.height * 0.5);
    gradientLayer.colors = [NSArray arrayWithObjects:
                            (id)[[UIColor clearColor] CGColor],
                            (id)[[UIColor colorWithRed:1.0 green:1.0 blue:1.0 alpha:0.3] CGColor], nil];

    gradientLayer.startPoint = CGPointMake(0.5,0.5);
    gradientLayer.endPoint = CGPointMake(0.5,1.0);
    reflectionLayer.mask = gradientLayer;

    [self.superview addSubview:reflectionImageView];
```

#### 15、画水印
```objectivec
// 画水印
- (void) setImage:(UIImage *)image withWaterMark:(UIImage *)mark inRect:(CGRect)rect
{
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 4.0)
    {
        UIGraphicsBeginImageContextWithOptions(self.frame.size, NO, 0.0);
    }
    //原图
    [image drawInRect:self.bounds];
    //水印图
    [mark drawInRect:rect];
    UIImage *newPic = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    self.image = newPic;
}
```

#### 16、让label的文字内容显示在左上／右上／左下／右下／中心顶／中心底部
```objectivec
自定义UILabel
// 重写label的textRectForBounds方法
- (CGRect)textRectForBounds:(CGRect)bounds limitedToNumberOfLines:(NSInteger)numberOfLines {
    CGRect rect = [super textRectForBounds:bounds limitedToNumberOfLines:numberOfLines];
    switch (self.textAlignmentType) {
        case WZBTextAlignmentTypeLeftTop: {
            rect.origin = bounds.origin;
        }
            break;
        case WZBTextAlignmentTypeRightTop: {
            rect.origin = CGPointMake(CGRectGetMaxX(bounds) - rect.size.width, bounds.origin.y);
        }
            break;
        case WZBTextAlignmentTypeLeftBottom: {
            rect.origin = CGPointMake(bounds.origin.x, CGRectGetMaxY(bounds) - rect.size.height);
        }
            break;
        case WZBTextAlignmentTypeRightBottom: {
            rect.origin = CGPointMake(CGRectGetMaxX(bounds) - rect.size.width, CGRectGetMaxY(bounds) - rect.size.height);
        }
            break;
        case WZBTextAlignmentTypeTopCenter: {
            rect.origin = CGPointMake((CGRectGetWidth(bounds) - CGRectGetWidth(rect)) / 2, CGRectGetMaxY(bounds) - rect.origin.y);
        }
            break;
        case WZBTextAlignmentTypeBottomCenter: {
            rect.origin = CGPointMake((CGRectGetWidth(bounds) - CGRectGetWidth(rect)) / 2, CGRectGetMaxY(bounds) - CGRectGetMaxY(bounds) - rect.size.height);
        }
            break;
        case WZBTextAlignmentTypeLeft: {
            rect.origin = CGPointMake(0, rect.origin.y);
        }
            break;
        case WZBTextAlignmentTypeRight: {
            rect.origin = CGPointMake(rect.origin.x, 0);
        }
            break;
        case WZBTextAlignmentTypeCenter: {
            rect.origin = CGPointMake((CGRectGetWidth(bounds) - CGRectGetWidth(rect)) / 2, (CGRectGetHeight(bounds) - CGRectGetHeight(rect)) / 2);
        }
            break;

        default:
            break;
    }
    return rect;
}
- (void)drawTextInRect:(CGRect)rect {
    CGRect textRect = [self textRectForBounds:rect limitedToNumberOfLines:self.numberOfLines];
    [super drawTextInRect:textRect];
}
```

#### 17、scrollView上的输入框，键盘挡住的问题
```objectivec
推荐用IQKeyboardManager这个框架！
手动解决如下
1、监听键盘弹出／消失的通知
2、在通知中加入代码：
NSDictionary* info = [aNotification userInfo];
CGRect keyPadFrame=[[UIApplication sharedApplication].keyWindow convertRect:[[info objectForKey:UIKeyboardFrameBeginUserInfoKey] CGRectValue] fromView:self.view];
CGSize kbSize =keyPadFrame.size;
CGRect activeRect=[self.view convertRect:activeField.frame fromView:activeField.superview];
CGRect aRect = self.view.bounds;
aRect.size.height -= (kbSize.height);

CGPoint origin =  activeRect.origin;
origin.y -= backScrollView.contentOffset.y;
if (!CGRectContainsPoint(aRect, origin)) {
    CGPoint scrollPoint = CGPointMake(0.0,CGRectGetMaxY(activeRect)-(aRect.size.height));
    [backScrollView setContentOffset:scrollPoint animated:YES];
}
```

#### 18、frame布局的cell动态高度
```objectivec
这种通常在你的模型中添加一个辅助属性cellHeight，在模型中重写这个属性的get方法，根据你的布局和模型中的其他属性值计算出总高度。最后在tableView：heightForRow方法中，根据indexPath找出对应的模型，返回这个高度即可。
```

#### 19、AutoLayout布局的cell动态高度
```objectivec
// 1、设置tableView的属性
self.tableView.rowHeight = UITableViewAutomaticDimension;
self.tableView.estimatedRowHeight = 44.0; // 这个属性非0，估计cell高度
// 2、至上而下设置cell的约束，注意，上下左右最好都要顶到cell的四周
```

#### 20、使用performSelector:调用函数，内存泄漏问题

>当我们在开发中使用[obj performSelector:NSSelectorFromString(@"aMethod")];这类方法时可能会收到一个警告"performSelector may cause a leak because its selector is unknown".
是因为编译器不清楚这个对象能不能相应这个方法，如果不能，则是不安全的，而且编译器也不清楚该怎么处理这个方法的返回值！

```objectivec
使用以下代码调用即可：
if (! obj) { return; }
SEL selector = NSSelectorFromString(@"aMethod");
IMP imp = [obj methodForSelector:selector];
void (*func)(id, SEL) = (void *)imp;
func(obj, selector);

或者：
SEL selector = NSSelectorFromString(@"aMethod");
((void (*)(id, SEL))[obj methodForSelector:selector])(obj, selector);
```

#### 21、一个字符串是否包含另一个字符串
```objectivec
// 方法1
if ([str1 containsString:str2]) {
        NSLog(@"str1包含str2");
    } else {
        NSLog(@"str1不包含str2");
    }

// 方法2
if ([str1 rangeOfString: str2].location == NSNotFound) {
        NSLog(@"str1不包含str2");
    } else {
        NSLog(@"str1包含str2");
    }
```

#### 22、cell去除选中效果
```objective
cell.selectionStyle = UITableViewCellSelectionStyleNone;
```

#### 23、cell点按效果
```objective
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
}
```


#### 24、当删除一个从xib拖出来的属性时，一定记得把xib中对应的线也删掉，不然会报类似[<ViewController 0x7fea6ed05980> setValue:forUndefinedKey:]: this class is not key value coding-compliant for the key的crash


#### 25、真机测试的时候报错：Could not launch "你的 App"，process launch failed: Security
```objective
因为你的app没有上线，iOS9开始，需要手动信任Xcode生成的描述文件，打开手机设置->通用->描述文件->点击你的app的描述文件->点击信任
```


#### 26、真机测试的时候报错：Could not find Developer Disk Image

>这是因为你的设备系统版本大于Xcode能兼容的系统版本，比如你的设备是iOS10.3，而Xcode版本是8.2（Xcode8.2最大兼容iOS10.2），就会报这个错误。解决办法就是升级Xcode！

#### 27、UITextView没有placeholder的问题？
网上有很多此类自定义控件，也可以参考下我写的一个UITextView分类 [UITextView-WZB](https://github.com/WZBbiao/UITextView-WZB)


#### 28、移除字符串中的空格和换行
```objective
+ (NSString *)removeSpaceAndNewline:(NSString *)str {
    NSString *temp = [str stringByReplacingOccurrencesOfString:@" " withString:@""];
    temp = [temp stringByReplacingOccurrencesOfString:@"\r" withString:@""];
    temp = [temp stringByReplacingOccurrencesOfString:@"\n" withString:@""];
    return temp;
}
```

#### 29、判断字符串中是否有空格
```objective
+ (BOOL)isBlank:(NSString *)str {
    NSRange _range = [str rangeOfString:@" "];
    if (_range.location != NSNotFound) {
        //有空格
        return YES;
    } else {
        //没有空格
        return NO;
    }
}
```

#### 30、获取一个视频的第一帧图片

```objectivec
    NSURL *url = [NSURL URLWithString:filepath];
    AVURLAsset *asset1 = [[AVURLAsset alloc] initWithURL:url options:nil];
    AVAssetImageGenerator *generate1 = [[AVAssetImageGenerator alloc] initWithAsset:asset1];
    generate1.appliesPreferredTrackTransform = YES;
    NSError *err = NULL;
    CMTime time = CMTimeMake(1, 2);
    CGImageRef oneRef = [generate1 copyCGImageAtTime:time actualTime:NULL error:&err];
    UIImage *one = [[UIImage alloc] initWithCGImage:oneRef];

    return one;
```


