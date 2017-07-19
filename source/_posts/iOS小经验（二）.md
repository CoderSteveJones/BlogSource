---
title: iOS小经验（二）
date: 2016-02-14 09:53:06
categories: 
	- 小经验
---
#### 1、自定义NSLog
```objectivec
#ifdef DEBUG
#define NSLog(fmt, ...) NSLog((@"%s [Line %d] " fmt), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__)
#else
#define NSLog(...)
#endif
```

#### 2、Font
```objectivec
#define FontL(s)             [UIFont systemFontOfSize:s weight:UIFontWeightLight]
#define FontR(s)             [UIFont systemFontOfSize:s weight:UIFontWeightRegular]
#define FontB(s)             [UIFont systemFontOfSize:s weight:UIFontWeightBold]
#define FontT(s)             [UIFont systemFontOfSize:s weight:UIFontWeightThin]
#define Font(s)              FontL(s)
```

#### 3、FORMAT
```objectivec
#define FORMAT(f, ...)      [NSString stringWithFormat:f, ## __VA_ARGS__]
```

#### 4、在主线程上运行
```objectivec
#define kDISPATCH_MAIN_THREAD(mainQueueBlock) dispatch_async(dispatch_get_main_queue(), mainQueueBlock);
```

#### 5、开启异步线程
```objectivec
#define kDISPATCH_GLOBAL_QUEUE_DEFAULT(globalQueueBlock) dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), globalQueueBlocl);
```

#### 6、通知
```objectivec
#define NOTIF_ADD(n, f)     [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(f) name:n object:nil]
#define NOTIF_POST(n, o)    [[NSNotificationCenter defaultCenter] postNotificationName:n object:o]
#define NOTIF_REMV()        [[NSNotificationCenter defaultCenter] removeObserver:self]
```

#### 7、随机颜色
```objectivec
+ (UIColor *)RandomColor {
    NSInteger aRedValue = arc4random() % 255;
    NSInteger aGreenValue = arc4random() % 255;
    NSInteger aBlueValue = arc4random() % 255;
    UIColor *randColor = [UIColor colorWithRed:aRedValue / 255.0f green:aGreenValue / 255.0f blue:aBlueValue / 255.0f alpha:1.0f];
    return randColor;
}
```


#### 8、获取window
```objectivec
+(UIWindow*)getWindow {
    UIWindow* win = nil; //[UIApplication sharedApplication].keyWindow;
    for (id item in [UIApplication sharedApplication].windows) {
        if ([item class] == [UIWindow class]) {
            if (!((UIWindow*)item).hidden) {
                win = item;
                break;
            }
        }
    }
    return win;
}
```

#### 9、修改textField的placeholder的字体颜色、大小
```objectivec
[textField setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
[textField setValue:[UIFont boldSystemFontOfSize:16] forKeyPath:@"_placeholderLabel.font"];
```


#### 10、统一收起键盘
```objectivec
[[[UIApplication sharedApplication] keyWindow] endEditing:YES];
```

#### 11、控制屏幕旋转，在控制器中写
```objectivec
/** 是否支持自动转屏 */
- (BOOL)shouldAutorotate {
    return YES;
}

/** 支持哪些屏幕方向 */
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscapeLeft | UIInterfaceOrientationMaskLandscapeRight;
}

/** 默认的屏幕方向（当前ViewController必须是通过模态出来的UIViewController（模态带导航的无效）方式展现出来的，才会调用这个方法） */
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return UIInterfaceOrientationLandscapeLeft | UIInterfaceOrientationLandscapeRight;
}
```

#### 12、获取app缓存大小
```objectivec
- (CGFloat)getCachSize {

    NSUInteger imageCacheSize = [[SDImageCache sharedImageCache] getSize];
    //获取自定义缓存大小
    //用枚举器遍历 一个文件夹的内容
    //1.获取 文件夹枚举器
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    NSDirectoryEnumerator *enumerator = [[NSFileManager defaultManager] enumeratorAtPath:myCachePath];
    __block NSUInteger count = 0;
    //2.遍历
    for (NSString *fileName in enumerator) {
        NSString *path = [myCachePath stringByAppendingPathComponent:fileName];
        NSDictionary *fileDict = [[NSFileManager defaultManager] attributesOfItemAtPath:path error:nil];
        count += fileDict.fileSize;//自定义所有缓存大小
    }
    // 得到是字节  转化为M
    CGFloat totalSize = ((CGFloat)imageCacheSize+count)/1024/1024;
    return totalSize;
}
```

#### 13、清理app缓存
```objectivec
- (void)handleClearView {
    //删除两部分
    //1.删除 sd 图片缓存
    //先清除内存中的图片缓存
    [[SDImageCache sharedImageCache] clearMemory];
    //清除磁盘的缓存
    [[SDImageCache sharedImageCache] clearDisk];
    //2.删除自己缓存
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    [[NSFileManager defaultManager] removeItemAtPath:myCachePath error:nil];
}
```

#### 14、模型转字典
```objectivec
static NSSet *classes;

- (NSMutableDictionary *)getParameterDictionary {

    NSMutableDictionary *dict = [NSMutableDictionary dictionary];

    Class c = self.class;

    while (c) {
        unsigned count;
        objc_property_t *properties = class_copyPropertyList([c class], &count);

        for (int i = 0; i < count; i++) {
            NSString *key = [NSString stringWithUTF8String:property_getName(properties[i])];
            dict[key] = [self valueForKey:key];
        }
        free(properties);

        // 获得父类
        c = class_getSuperclass(c);

        if ([self isClassFromFoundation:c]) break;
    }
    return dict;
}

- (BOOL)isClassFromFoundation:(Class)c
{
    if (c == [NSObject class] || c == [NSManagedObject class]) return YES;

    __block BOOL result = NO;
    [[self foundationClasses] enumerateObjectsUsingBlock:^(Class foundationClass, BOOL *stop) {
        if ([c isSubclassOfClass:foundationClass]) {
            result = YES;
            *stop = YES;
        }
    }];
    return result;
}

- (NSSet *)foundationClasses
{
    if (classes == nil) {
        // 集合中没有NSObject，因为几乎所有的类都是继承自NSObject，具体是不是NSObject需要特殊判断
        classes = [NSSet setWithObjects:
                              [NSURL class],
                              [NSDate class],
                              [NSValue class],
                              [NSData class],
                              [NSError class],
                              [NSArray class],
                              [NSDictionary class],
                              [NSString class],
                              [NSAttributedString class], nil];
    }
    return classes;
}
```

#### 15、交换两个方法实现
```objectivec
Class aClass = [self class]; 

        SEL originalSelector = @selector(viewWillAppear:); 
        SEL swizzledSelector = @selector(xxx_viewWillAppear:); 

        Method originalMethod = class_getInstanceMethod(aClass, originalSelector); 
        Method swizzledMethod = class_getInstanceMethod(aClass, swizzledSelector); 

        BOOL didAddMethod = 
            class_addMethod(aClass, 
                originalSelector, 
                method_getImplementation(swizzledMethod), 
                method_getTypeEncoding(swizzledMethod)); 

        if (didAddMethod) { 
            class_replaceMethod(aClass, 
                swizzledSelector, 
                method_getImplementation(originalMethod), 
                method_getTypeEncoding(originalMethod)); 
        } else { 
            method_exchangeImplementations(originalMethod, swizzledMethod); 
        }
```

#### 6、打印百分号和引号
```objectivec
    NSLog(@"%%");
    NSLog(@"\"");
```

#### 17、几个常用权限判断
```objectivec
    if ([CLLocationManager authorizationStatus] ==kCLAuthorizationStatusDenied) {
        NSLog(@"没有定位权限");
    }
    AVAuthorizationStatus statusVideo = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (statusVideo == AVAuthorizationStatusDenied) {
        NSLog(@"没有摄像头权限");
    }
    //是否有麦克风权限
    AVAuthorizationStatus statusAudio = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeAudio];
    if (statusAudio == AVAuthorizationStatusDenied) {
        NSLog(@"没有录音权限");
    }
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
        if (status == PHAuthorizationStatusDenied) {
            NSLog(@"没有相册权限");
        }
    }];
```

#### 18、获取手机型号
```objectivec
    + (NSString *)getDeviceInfo {
    struct utsname systemInfo;
    uname(&systemInfo);
    NSString *platform = [NSString stringWithCString:systemInfo.machine encoding:NSASCIIStringEncoding];
    if ([platform isEqualToString:@"iPhone1,1"]) return @"iPhone 2G";
    if ([platform isEqualToString:@"iPhone1,2"]) return @"iPhone 3G";
    if ([platform isEqualToString:@"iPhone2,1"]) return @"iPhone 3GS";
    if ([platform isEqualToString:@"iPhone3,1"]) return @"iPhone 4";
    if ([platform isEqualToString:@"iPhone3,2"]) return @"iPhone 4";
    if ([platform isEqualToString:@"iPhone3,3"]) return @"iPhone 4";
    if ([platform isEqualToString:@"iPhone4,1"]) return @"iPhone 4S";
    if ([platform isEqualToString:@"iPhone5,1"]) return @"iPhone 5";
    if ([platform isEqualToString:@"iPhone5,2"]) return @"iPhone 5";
    if ([platform isEqualToString:@"iPhone5,3"]) return @"iPhone 5c";
    if ([platform isEqualToString:@"iPhone5,4"]) return @"iPhone 5c";
    if ([platform isEqualToString:@"iPhone6,1"]) return @"iPhone 5s";
    if ([platform isEqualToString:@"iPhone6,2"]) return @"iPhone 5s";
    if ([platform isEqualToString:@"iPhone7,1"]) return @"iPhone 6 Plus";
    if ([platform isEqualToString:@"iPhone7,2"]) return @"iPhone 6";
    if ([platform isEqualToString:@"iPhone8,1"]) return @"iPhone 6s";
    if ([platform isEqualToString:@"iPhone8,2"]) return @"iPhone 6s Plus";
    // 日行两款手机型号均为日本独占，可能使用索尼FeliCa支付方案而不是苹果支付
    if ([platform isEqualToString:@"iPhone9,1"])    return @"国行、日版、港行iPhone 7";
    if ([platform isEqualToString:@"iPhone9,2"])    return @"港行、国行iPhone 7 Plus";
    if ([platform isEqualToString:@"iPhone9,3"])    return @"美版、台版iPhone 7";
    if ([platform isEqualToString:@"iPhone9,4"])    return @"美版、台版iPhone 7 Plus";
    if ([platform isEqualToString:@"iPhone8,4"]) return @"iPhone SE";
    if ([platform isEqualToString:@"iPod1,1"]) return @"iPod Touch 1G";
    if ([platform isEqualToString:@"iPod2,1"]) return @"iPod Touch 2G";
    if ([platform isEqualToString:@"iPod3,1"]) return @"iPod Touch 3G";
    if ([platform isEqualToString:@"iPod4,1"]) return @"iPod Touch 4G";
    if ([platform isEqualToString:@"iPod5,1"]) return @"iPod Touch 5G";
    if ([platform isEqualToString:@"iPad1,1"]) return @"iPad 1G";
    if ([platform isEqualToString:@"iPad2,1"]) return @"iPad 2";
    if ([platform isEqualToString:@"iPad2,2"]) return @"iPad 2";
    if ([platform isEqualToString:@"iPad2,3"]) return @"iPad 2";
    if ([platform isEqualToString:@"iPad2,4"]) return @"iPad 2";
    if ([platform isEqualToString:@"iPad2,5"]) return @"iPad Mini 1G";
    if ([platform isEqualToString:@"iPad2,6"]) return @"iPad Mini 1G";
    if ([platform isEqualToString:@"iPad2,7"]) return @"iPad Mini 1G";
    if ([platform isEqualToString:@"iPad3,1"]) return @"iPad 3";
    if ([platform isEqualToString:@"iPad3,2"]) return @"iPad 3";
    if ([platform isEqualToString:@"iPad3,3"]) return @"iPad 3";
    if ([platform isEqualToString:@"iPad3,4"]) return @"iPad 4";
    if ([platform isEqualToString:@"iPad3,5"]) return @"iPad 4";
    if ([platform isEqualToString:@"iPad3,6"]) return @"iPad 4";
    if ([platform isEqualToString:@"iPad4,1"]) return @"iPad Air";
    if ([platform isEqualToString:@"iPad4,2"]) return @"iPad Air";
    if ([platform isEqualToString:@"iPad4,3"]) return @"iPad Air";
    if ([platform isEqualToString:@"iPad4,4"]) return @"iPad Mini 2G";
    if ([platform isEqualToString:@"iPad4,5"]) return @"iPad Mini 2G";
    if ([platform isEqualToString:@"iPad4,6"]) return @"iPad Mini 2G";
    if ([platform isEqualToString:@"i386"]) return @"iPhone Simulator";
    if ([platform isEqualToString:@"x86_64"]) return @"iPhone Simulator";
    return platform;
}
```

#### 19、长按复制功能
```objectivec
- (void)viewDidLoad
{
    [self.view addGestureRecognizer:[[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(pasteBoard:)]];
}
- (void)pasteBoard:(UILongPressGestureRecognizer *)longPress {
    if (longPress.state == UIGestureRecognizerStateBegan) {
        UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
        pasteboard.string = @"需要复制的文本";
    }
}
```

#### 20、cocoapods升级
```objectivec
在终端执行 sudo gem install -n / usr / local / bin cocoapods --pre
```

#### 21、设置启动页后，依然显示之前的
```objectivec
删除app，手机重启，重新安装
```

#### 22、判断图片类型
```objectivec
//通过图片Data数据第一个字节 来获取图片扩展名
- (NSString *)contentTypeForImageData:(NSData *)data
{
    uint8_t c;
    [data getBytes:&c length:1];
    switch (c)
    {
        case 0xFF:
            return @"jpeg";

        case 0x89:
            return @"png";

        case 0x47:
            return @"gif";

        case 0x49:
        case 0x4D:
            return @"tiff";

        case 0x52:
        if ([data length] < 12) {
            return nil;
        }

        NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(0, 12)] encoding:NSASCIIStringEncoding];
        if ([testString hasPrefix:@"RIFF"]
            && [testString hasSuffix:@"WEBP"])
        {
            return @"webp";
        }

        return nil;
    }

    return nil;
}
```

#### 23、获取手机和app信息
```objectivec
NSDictionary *infoDictionary = [[NSBundle mainBundle] infoDictionary];  
 CFShow(infoDictionary);  
// app名称  
 NSString *app_Name = [infoDictionary objectForKey:@"CFBundleDisplayName"];  
 // app版本  
 NSString *app_Version = [infoDictionary objectForKey:@"CFBundleShortVersionString"];  
 // app build版本  
 NSString *app_build = [infoDictionary objectForKey:@"CFBundleVersion"];  



    //手机序列号  
    NSString* identifierNumber = [[UIDevice currentDevice] uniqueIdentifier];  
    NSLog(@"手机序列号: %@",identifierNumber);  
    //手机别名： 用户定义的名称  
    NSString* userPhoneName = [[UIDevice currentDevice] name];  
    NSLog(@"手机别名: %@", userPhoneName);  
    //设备名称  
    NSString* deviceName = [[UIDevice currentDevice] systemName];  
    NSLog(@"设备名称: %@",deviceName );  
    //手机系统版本  
    NSString* phoneVersion = [[UIDevice currentDevice] systemVersion];  
    NSLog(@"手机系统版本: %@", phoneVersion);  
    //手机型号  
    NSString* phoneModel = [[UIDevice currentDevice] model];  
    NSLog(@"手机型号: %@",phoneModel );  
    //地方型号  （国际化区域名称）  
    NSString* localPhoneModel = [[UIDevice currentDevice] localizedModel];  
    NSLog(@"国际化区域名称: %@",localPhoneModel );  

    NSDictionary *infoDictionary = [[NSBundle mainBundle] infoDictionary];  
    // 当前应用名称  
    NSString *appCurName = [infoDictionary objectForKey:@"CFBundleDisplayName"];  
    NSLog(@"当前应用名称：%@",appCurName);  
    // 当前应用软件版本  比如：1.0.1  
    NSString *appCurVersion = [infoDictionary objectForKey:@"CFBundleShortVersionString"];  
    NSLog(@"当前应用软件版本:%@",appCurVersion);  
    // 当前应用版本号码   int类型  
    NSString *appCurVersionNum = [infoDictionary objectForKey:@"CFBundleVersion"];  
    NSLog(@"当前应用版本号码：%@",appCurVersionNum);
```

#### 24、获取一个类的所有属性
```objectivec
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
}
```

#### 25、image圆角
```objectivec
- (UIImage *)circleImage
{
    // NO代表透明
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 1);
    // 获得上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 添加一个圆
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    // 方形变圆形
    CGContextAddEllipseInRect(ctx, rect);
    // 裁剪
    CGContextClip(ctx);
    // 将图片画上去
    [self drawInRect:rect];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```

#### 26、image拉伸
```objectivec
+ (UIImage *)resizableImage:(NSString *)imageName
{
    UIImage *image = [UIImage imageNamed:imageName];
    CGFloat imageW = image.size.width;
    CGFloat imageH = image.size.height;
    return [image resizableImageWithCapInsets:UIEdgeInsetsMake(imageH * 0.5, imageW * 0.5, imageH * 0.5, imageW * 0.5) resizingMode:UIImageResizingModeStretch];
}
```

#### 27、JSON字符串转字典
```objectivec
+ (NSDictionary *)parseJSONStringToNSDictionary:(NSString *)JSONString {
    NSData *JSONData = [JSONString dataUsingEncoding:NSUTF8StringEncoding];
    NSDictionary *responseJSON = [NSJSONSerialization JSONObjectWithData:JSONData options:NSJSONReadingMutableLeaves error:nil];
    return responseJSON;
}
```

#### 28、身份证号验证
```objectivec
- (BOOL)validateIdentityCard {
    BOOL flag;
    if (self.length <= 0) {
        flag = NO;
        return flag;
    }
    NSString *regex2 = @"^(\\d{14}|\\d{17})(\\d|[xX])$";
    NSPredicate *identityCardPredicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@",regex2];
    return [identityCardPredicate evaluateWithObject:self];
}
```

#### 29、获取设备mac地址

```objectivec
+ (NSString *)macAddress {
    int                 mib[6];
    size_t              len;
    char                *buf;
    unsigned char       *ptr;
    struct if_msghdr    *ifm;
    struct sockaddr_dl  *sdl;

    mib[0] = CTL_NET;
    mib[1] = AF_ROUTE;
    mib[2] = 0;
    mib[3] = AF_LINK;
    mib[4] = NET_RT_IFLIST;

    if((mib[5] = if_nametoindex("en0")) == 0) {
        printf("Error: if_nametoindex error\n");
        return NULL;
    }

    if(sysctl(mib, 6, NULL, &len, NULL, 0) < 0) {
        printf("Error: sysctl, take 1\n");
        return NULL;
    }

    if((buf = malloc(len)) == NULL) {
        printf("Could not allocate memory. Rrror!\n");
        return NULL;
    }

    if(sysctl(mib, 6, buf, &len, NULL, 0) < 0) {
        printf("Error: sysctl, take 2");
        return NULL;
    }

    ifm = (struct if_msghdr *)buf;
    sdl = (struct sockaddr_dl *)(ifm + 1);
    ptr = (unsigned char *)LLADDR(sdl);
    NSString *outstring = [NSString stringWithFormat:@"%02X:%02X:%02X:%02X:%02X:%02X",
                           *ptr, *(ptr+1), *(ptr+2), *(ptr+3), *(ptr+4), *(ptr+5)];
    free(buf);

    return outstring;
}
```

#### 30、导入自定义字体库
```objectivec
1、找到你想用的字体的 ttf 格式，拖入工程
2、在工程的plist中增加一行数组，“Fonts provided by application”
3、为这个key添加一个item，value为你刚才导入的ttf文件名
4、直接使用即可：label.font = [UIFont fontWithName:@"你刚才导入的ttf文件名" size:20.0]；
```

