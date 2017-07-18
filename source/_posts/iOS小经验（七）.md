---
title: iOS小经验（七）
date: 2016-03-20 09:53:06
categories: 
	- iOS小经验
---
#### 1、地图上两个点之间的实际距离
```objectivec
// 需要导入#import <CoreLocation/CoreLocation.h>
CLLocation *locA = [[CLLocation alloc] initWithLatitude:34 longitude:113];
    CLLocation *locB = [[CLLocation alloc] initWithLatitude:31.05 longitude:121.76];
// CLLocationDistance求出的单位为米
    CLLocationDistance distance = [locA distanceFromLocation:locB];
```

#### 2、在应用中打开设置的某个界面
```objectivec
// 打开设置->通用
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=General"]];

// 以下是设置其他界面
prefs:root=General&path=About
prefs:root=General&path=ACCESSIBILITY
prefs:root=AIRPLANE_MODE
prefs:root=General&path=AUTOLOCK
prefs:root=General&path=USAGE/CELLULAR_USAGE
prefs:root=Brightness
prefs:root=Bluetooth
prefs:root=General&path=DATE_AND_TIME
prefs:root=FACETIME
prefs:root=General
prefs:root=General&path=Keyboard
prefs:root=CASTLE
prefs:root=CASTLE&path=STORAGE_AND_BACKUP
prefs:root=General&path=INTERNATIONAL
prefs:root=LOCATION_SERVICES
prefs:root=ACCOUNT_SETTINGS
prefs:root=MUSIC
prefs:root=MUSIC&path=EQ
prefs:root=MUSIC&path=VolumeLimit
prefs:root=General&path=Network
prefs:root=NIKE_PLUS_IPOD
prefs:root=NOTES
prefs:root=NOTIFICATIONS_ID
prefs:root=Phone
prefs:root=Photos
prefs:root=General&path=ManagedConfigurationList
prefs:root=General&path=Reset
prefs:root=Sounds&path=Ringtone
prefs:root=Safari
prefs:root=General&path=Assistant
prefs:root=Sounds
prefs:root=General&path=SOFTWARE_UPDATE_LINK
prefs:root=STORE
prefs:root=TWITTER
prefs:root=FACEBOOK
prefs:root=General&path=USAGE prefs:root=VIDEO
prefs:root=General&path=Network/VPN
prefs:root=Wallpaper
prefs:root=WIFI
prefs:root=INTERNET_TETHERING
prefs:root=Phone&path=Blocked
prefs:root=DO_NOT_DISTURB
```

#### 3、在UITextView中显示html文本
```objectivec
    UITextView *textView = [[UITextView alloc] initWithFrame:CGRectMake(20, 30, 100, 199)];
    textView.backgroundColor = [UIColor redColor];
    [self.view addSubview:textView];
    NSString *htmlString = @"<h1>Header</h1><h2>Subheader</h2><p>Some <em>text</em></p>![](http://blogs.babble.com/famecrawler/files/2010/11/mickey_mouse-1097.jpg)";
    NSAttributedString *attributedString = [[NSAttributedString alloc] initWithData: [htmlString dataUsingEncoding:NSUnicodeStringEncoding] options: @{ NSDocumentTypeDocumentAttribute: NSHTMLTextDocumentType } documentAttributes: nil error: nil];
    textView.attributedText = attributedString;
```

#### 4、监听scrollView是否滚动到了顶部／底部
```objectivec
-(void)scrollViewDidScroll: (UIScrollView*)scrollView
{
    float scrollViewHeight = scrollView.frame.size.height;
    float scrollContentSizeHeight = scrollView.contentSize.height;
    float scrollOffset = scrollView.contentOffset.y;

    if (scrollOffset == 0)
    {
        // 滚动到了顶部
    }
    else if (scrollOffset + scrollViewHeight == scrollContentSizeHeight)
    {
        // 滚动到了底部
    }
}
```

#### 5、UISlider增量／减量为固定值(假如为5)
```objectivec
- (void)setupSlider
{
    UISlider *slider = [[UISlider alloc] init];
    [self.view addSubview:slider];
    [slider addTarget:self action:@selector(sliderAction:) forControlEvents:UIControlEventValueChanged];
    slider.maximumValue = 100;
    slider.minimumValue = 0;
    slider.frame = CGRectMake(200, 20, 100, 30);
}

- (void)sliderAction:(UISlider *)slider
{
    [slider setValue:((int)((slider.value + 2.5) / 5) * 5) animated:NO];
}
```

#### 6、选中textField或者textView所有文本(我这里以textView为例)
```objectivec
[self.textView setSelectedTextRange:[self.textView textRangeFromPosition:self.textView.beginningOfDocument toPosition:self.textView.endOfDocument]]
```


#### 7、从导航控制器中删除某个控制器
```objectivec
// 方法一、知道这个控制器所处的导航控制器下标
NSMutableArray *navigationArray = [[NSMutableArray alloc] initWithArray: self.navigationController.viewControllers];
[navigationArray removeObjectAtIndex: 2]; 
self.navigationController.viewControllers = navigationArray;

// 方法二、知道具体是哪个控制器
NSArray* tempVCA = [self.navigationController viewControllers];

for(UIViewController *tempVC in tempVCA)
{
    if([tempVC isKindOfClass:[urViewControllerClass class]])
    {
        [tempVC removeFromParentViewController];
    }
}
```

#### 8、隐藏UITextView/UITextField光标
```objectivec
textField.tintColor = [UIColor clearColor];
```

#### 9、当UITextView/UITextField中没有文字时，禁用回车键
```objectivec
textField.enablesReturnKeyAutomatically = YES;
```

#### 10、字符串encode编码(编码url字符串不成功的问题)
```objectivec
// 我们一般用这个方法处理stringByAddingPercentEscapesUsingEncoding但是这个方法好想不会处理／和&这种特殊符号，这种情况就需要用下边这个方法处理
@implementation NSString (NSString_Extended)
- (NSString *)urlencode {
    NSMutableString *output = [NSMutableString string];
    const unsigned char *source = (const unsigned char *)[self UTF8String];
    int sourceLen = strlen((const char *)source);
    for (int i = 0; i < sourceLen; ++i) {
        const unsigned char thisChar = source[i];
        if (thisChar == ' '){
            [output appendString:@"+"];
        } else if (thisChar == '.' || thisChar == '-' || thisChar == '_' || thisChar == '~' || 
                   (thisChar >= 'a' && thisChar <= 'z') ||
                   (thisChar >= 'A' && thisChar <= 'Z') ||
                   (thisChar >= '0' && thisChar <= '9')) {
            [output appendFormat:@"%c", thisChar];
        } else {
            [output appendFormat:@"%%%02X", thisChar];
        }
    }
    return output;
}
```

#### 11、计算UILabel上某段文字的frame
```objectivec
@implementation UILabel (TextRect)

- (CGRect)boundingRectForCharacterRange:(NSRange)range
{
    NSTextStorage *textStorage = [[NSTextStorage alloc] initWithAttributedString:[self attributedText]];
    NSLayoutManager *layoutManager = [[NSLayoutManager alloc] init];
    [textStorage addLayoutManager:layoutManager];
    NSTextContainer *textContainer = [[NSTextContainer alloc] initWithSize:[self bounds].size];
    textContainer.lineFragmentPadding = 0;
    [layoutManager addTextContainer:textContainer];
    NSRange glyphRange;
    [layoutManager characterRangeForGlyphRange:range actualGlyphRange:&glyphRange];
    return [layoutManager boundingRectForGlyphRange:glyphRange inTextContainer:textContainer];
}
```

#### 12、获取随机UUID
```objectivec
NSString *result;
    if([[[UIDevice currentDevice] systemVersion] floatValue] > 6.0)
    {
       result = [[NSUUID UUID] UUIDString];
    }
    else
    {
        CFUUIDRef uuidRef = CFUUIDCreate(NULL);
        CFStringRef uuid = CFUUIDCreateString(NULL, uuidRef);
        CFRelease(uuidRef);
        result = (__bridge_transfer NSString *)uuid;
    }
```

#### 13、仿苹果抖动动画
```objectivec
#define RADIANS(degrees) (((degrees) * M_PI) / 180.0)

- (void)startAnimate {
    view.transform = CGAffineTransformRotate(CGAffineTransformIdentity, RADIANS(-5));

    [UIView animateWithDuration:0.25 delay:0.0 options:(UIViewAnimationOptionAllowUserInteraction | UIViewAnimationOptionRepeat | UIViewAnimationOptionAutoreverse) animations:^ {
                         view.transform = CGAffineTransformRotate(CGAffineTransformIdentity, RADIANS(5));
                     } completion:nil];
}

- (void)stopAnimate {
    [UIView animateWithDuration:0.25 delay:0.0 options:(UIViewAnimationOptionAllowUserInteraction | UIViewAnimationOptionBeginFromCurrentState | UIViewAnimationOptionCurveLinear) animations:^ {
                         view.transform = CGAffineTransformIdentity;
                     } completion:nil];
}
```

#### 14、修改UISearBar内部背景颜色
```objectivec
UITextField *textField = [_searchBar valueForKey:@"_searchField"];
textField.backgroundColor = [UIColor redColor];
```

#### 15、UITextView滚动到顶部
```objectivec
    // 方法一
    [self.textView scrollRangeToVisible:NSMakeRange(0, 0)];
    // 方法二
    [self.textView setContentOffset:CGPointZero animated:YES];
```

#### 16、通知监听APP生命周期
```objectivec
UIApplicationDidEnterBackgroundNotification 应用程序进入后台
UIApplicationWillEnterForegroundNotification 应用程序将要进入前台
UIApplicationDidFinishLaunchingNotification 应用程序完成启动
UIApplicationDidFinishLaunchingNotification 应用程序由挂起变的活跃
UIApplicationWillResignActiveNotification 应用程序挂起(有电话进来或者锁屏)
UIApplicationDidReceiveMemoryWarningNotification 应用程序收到内存警告
UIApplicationDidReceiveMemoryWarningNotification 应用程序终止(后台杀死、手机关机等)
UIApplicationSignificantTimeChangeNotification 当有重大时间改变(凌晨0点，设备时间被修改，时区改变等)
UIApplicationWillChangeStatusBarOrientationNotification 设备方向将要改变
UIApplicationDidChangeStatusBarOrientationNotification 设备方向改变
UIApplicationWillChangeStatusBarFrameNotification 设备状态栏frame将要改变
UIApplicationDidChangeStatusBarFrameNotification 设备状态栏frame改变
UIApplicationBackgroundRefreshStatusDidChangeNotification 应用程序在后台下载内容的状态发生变化
UIApplicationProtectedDataWillBecomeUnavailable 本地受保护的文件被锁定,无法访问
UIApplicationProtectedDataWillBecomeUnavailable 本地受保护的文件可用了
```

#### 17、触摸事件类型
```objectivec
UIControlEventTouchCancel 取消控件当前触发的事件
UIControlEventTouchDown 点按下去的事件
UIControlEventTouchDownRepeat 重复的触动事件
UIControlEventTouchDragEnter 手指被拖动到控件的边界的事件
UIControlEventTouchDragExit 一个手指从控件内拖到外界的事件
UIControlEventTouchDragInside 手指在控件的边界内拖动的事件
UIControlEventTouchDragOutside 手指在控件边界之外被拖动的事件
UIControlEventTouchUpInside 手指处于控制范围内的触摸事件
UIControlEventTouchUpOutside 手指超出控制范围的控制中的触摸事件
```

#### 18、UITextField文字周围增加边距
```objectivec
// 子类化UITextField，增加insert属性
@interface WZBTextField : UITextField
@property (nonatomic, assign) UIEdgeInsets insets;
@end

// 在.m文件重写下列方法
- (CGRect)textRectForBounds:(CGRect)bounds {
    CGRect paddedRect = UIEdgeInsetsInsetRect(bounds, self.insets);
    if (self.rightViewMode == UITextFieldViewModeAlways || self.rightViewMode == UITextFieldViewModeUnlessEditing) {
        return [self adjustRectWithWidthRightView:paddedRect];
    }
    return paddedRect;
}

- (CGRect)placeholderRectForBounds:(CGRect)bounds {
    CGRect paddedRect = UIEdgeInsetsInsetRect(bounds, self.insets);

    if (self.rightViewMode == UITextFieldViewModeAlways || self.rightViewMode == UITextFieldViewModeUnlessEditing) {
        return [self adjustRectWithWidthRightView:paddedRect];
    }
    return paddedRect;
}

- (CGRect)editingRectForBounds:(CGRect)bounds {
    CGRect paddedRect = UIEdgeInsetsInsetRect(bounds, self.insets);
    if (self.rightViewMode == UITextFieldViewModeAlways || self.rightViewMode == UITextFieldViewModeWhileEditing) {
        return [self adjustRectWithWidthRightView:paddedRect];
    }
    return paddedRect;
}

- (CGRect)adjustRectWithWidthRightView:(CGRect)bounds {
    CGRect paddedRect = bounds;
    paddedRect.size.width -= CGRectGetWidth(self.rightView.frame);

    return paddedRect;
}
```

#### 19、监听UISlider拖动状态
```objectivec
// 添加事件
[slider addTarget:self action:@selector(sliderValurChanged:forEvent:) forControlEvents:UIControlEventValueChanged];

// 实现方法
- (void)sliderValurChanged:(UISlider*)slider forEvent:(UIEvent*)event {
    UITouch *touchEvent = [[event allTouches] anyObject];
    switch (touchEvent.phase) {
        case UITouchPhaseBegan:
            NSLog(@"开始拖动");
            break;
        case UITouchPhaseMoved:
            NSLog(@"正在拖动");
            break;
        case UITouchPhaseEnded:
            NSLog(@"结束拖动");
            break;
        default:
            break;
    }
}
```

#### 20、设置UITextField光标位置
```objectivec
// textField需要设置的textField，index要设置的光标位置
- (void)cursorLocation:(UITextField *)textField index:(NSInteger)index
{
    NSRange range = NSMakeRange(index, 0);
    UITextPosition *start = [textField positionFromPosition:[textField beginningOfDocument] offset:range.location];
    UITextPosition *end = [textField positionFromPosition:start offset:range.length];
    [textField setSelectedTextRange:[textField textRangeFromPosition:start toPosition:end]];
}
```

#### 21、去除webView底部黑色
```objectivec
[webView setBackgroundColor:[UIColor clearColor]];
    [webView setOpaque:NO];

    for (UIView *v1 in [webView subviews])
    {
        if ([v1 isKindOfClass:[UIScrollView class]])
        {
            for (UIView *v2 in v1.subviews)
            {
                if ([v2 isKindOfClass:[UIImageView class]])
                {
                    v2.hidden = YES;
                }
            }
        }
    }
```
#### 22、获取collectionViewCell在屏幕中的frame
```objectivec
UICollectionViewLayoutAttributes *attributes = [collectionView layoutAttributesForItemAtIndexPath:indexPath];
CGRect cellRect = attributes.frame;
CGRect cellFrameInSuperview = [collectionView convertRect:cellRect toView:[cv superview]];
```

#### 23、比较两个UIImage是否相等
```objectivec
- (BOOL)image:(UIImage *)image1 isEqualTo:(UIImage *)image2
{
    NSData *data1 = UIImagePNGRepresentation(image1);
    NSData *data2 = UIImagePNGRepresentation(image2);

    return [data1 isEqual:data2];
}
```

#### 24、解决当UIScrollView上有UIButton的时候，触摸到button滑动不了的问题
```objectivec
// 子类化UIScrollView，并重写以下方法
- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        self.delaysContentTouches = NO;
    }

    return self;
}

- (BOOL)touchesShouldCancelInContentView:(UIView *)view {
    if ([view isKindOfClass:UIButton.class]) {
        return YES;
    }

    return [super touchesShouldCancelInContentView:view];
}
```

#### 25、UITextView中的文字添加阴影效果
```objectivec
- (void)setTextLayer:(UITextView *)textView color:(UIColor *)color
{
    CALayer *textLayer = ((CALayer *)[textView.layer.sublayers objectAtIndex:0]);
    textLayer.shadowColor = color.CGColor;
    textLayer.shadowOffset = CGSizeMake(0.0f, 1.0f);
    textLayer.shadowOpacity = 1.0f;
    textLayer.shadowRadius = 1.0f;
}
```

#### 26、MD5加密
```objectivec
+ (NSString *)md5:(NSString *)str
{
    const char *concat_str = [str UTF8String];
    unsigned char result[CC_MD5_DIGEST_LENGTH];
    CC_MD5(concat_str, (unsigned int)strlen(concat_str), result);
    NSMutableString *hash = [NSMutableString string];
    for (int i =0; i <16; i++){
        [hash appendFormat:@"%02X", result[i]];
    }
    return [hash uppercaseString];
}
```


#### 27、base64加密
```objectivec
@interface NSData (Base64)
/**
 *  @brief  字符串base64后转data
 */
+ (NSData *)dataWithBase64EncodedString:(NSString *)string
{
    if (![string length]) return nil;
    NSData *decoded = nil;
#if __MAC_OS_X_VERSION_MIN_REQUIRED < __MAC_10_9 || __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_7_0
    if (![NSData instancesRespondToSelector:@selector(initWithBase64EncodedString:options:)])
    {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        decoded = [[self alloc] initWithBase64Encoding:[string stringByReplacingOccurrencesOfString:@"[^A-Za-z0-9+/=]" withString:@"" options:NSRegularExpressionSearch range:NSMakeRange(0, [string length])]];
#pragma clang diagnostic pop
    }
    else
#endif
    {
        decoded = [[self alloc] initWithBase64EncodedString:string options:NSDataBase64DecodingIgnoreUnknownCharacters];
    }
    return [decoded length]? decoded: nil;
}
/**
 *  @brief  NSData转string
 *  @param wrapWidth 换行长度  76  64
 */
- (NSString *)base64EncodedStringWithWrapWidth:(NSUInteger)wrapWidth
{
    if (![self length]) return nil;
    NSString *encoded = nil;
#if __MAC_OS_X_VERSION_MIN_REQUIRED < __MAC_10_9 || __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_7_0
    if (![NSData instancesRespondToSelector:@selector(base64EncodedStringWithOptions:)])
    {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        encoded = [self base64Encoding];
#pragma clang diagnostic pop

    }
    else
#endif
    {
        switch (wrapWidth)
        {
            case 64:
            {
                return [self base64EncodedStringWithOptions:NSDataBase64Encoding64CharacterLineLength];
            }
            case 76:
            {
                return [self base64EncodedStringWithOptions:NSDataBase64Encoding76CharacterLineLength];
            }
            default:
            {
                encoded = [self base64EncodedStringWithOptions:(NSDataBase64EncodingOptions)0];
            }
        }
    }
    if (!wrapWidth || wrapWidth >= [encoded length])
    {
        return encoded;
    }
    wrapWidth = (wrapWidth / 4) * 4;
    NSMutableString *result = [NSMutableString string];
    for (NSUInteger i = 0; i < [encoded length]; i+= wrapWidth)
    {
        if (i + wrapWidth >= [encoded length])
        {
            [result appendString:[encoded substringFromIndex:i]];
            break;
        }
        [result appendString:[encoded substringWithRange:NSMakeRange(i, wrapWidth)]];
        [result appendString:@"\r\n"];
    }
    return result;
}
/**
 *  @brief  NSData转string 换行长度默认64
 */
- (NSString *)base64EncodedString
{
    return [self base64EncodedStringWithWrapWidth:0];
}
```

#### 28、AES加密
```objectivec
#import <CommonCrypto/CommonCryptor.h>
@interface NSData (AES)
/**
 *  利用AES加密数据
 */
- (NSData*)encryptedWithAESUsingKey:(NSString*)key andIV:(NSData*)iv {

    NSData *keyData = [key dataUsingEncoding:NSUTF8StringEncoding];

    size_t dataMoved;
    NSMutableData *encryptedData = [NSMutableData dataWithLength:self.length + kCCBlockSizeAES128];

    CCCryptorStatus status = CCCrypt(kCCEncrypt,kCCAlgorithmAES128,kCCOptionPKCS7Padding,keyData.bytes,keyData.length,iv.bytes,self.bytes,self.length,encryptedData.mutableBytes, encryptedData.length,&dataMoved);

    if (status == kCCSuccess) {
        encryptedData.length = dataMoved;
        return encryptedData;
    }

    return nil;

}

/**
 *  @brief  利用AES解密据
 */
- (NSData*)decryptedWithAESUsingKey:(NSString*)key andIV:(NSData*)iv {

    NSData *keyData = [key dataUsingEncoding:NSUTF8StringEncoding];

    size_t dataMoved;
    NSMutableData *decryptedData = [NSMutableData dataWithLength:self.length + kCCBlockSizeAES128];

    CCCryptorStatus result = CCCrypt(kCCDecrypt,kCCAlgorithmAES128,kCCOptionPKCS7Padding,keyData.bytes,keyData.length,iv.bytes,self.bytes,self.length,decryptedData.mutableBytes, decryptedData.length,&dataMoved);

    if (result == kCCSuccess) {
        decryptedData.length = dataMoved;
        return decryptedData;
    }

    return nil;

}
```

#### 29、3DES加密
```objectivec
#import <CommonCrypto/CommonCryptor.h>
@interface NSData (3DES)
/**
 *  利用3DES加密数据
 */
- (NSData*)encryptedWith3DESUsingKey:(NSString*)key andIV:(NSData*)iv {

    NSData *keyData = [key dataUsingEncoding:NSUTF8StringEncoding];

    size_t dataMoved;
    NSMutableData *encryptedData = [NSMutableData dataWithLength:self.length + kCCBlockSize3DES];

    CCCryptorStatus result = CCCrypt(kCCEncrypt,kCCAlgorithm3DES,kCCOptionPKCS7Padding,keyData.bytes,keyData.length,iv.bytes,self.bytes,self.length,encryptedData.mutableBytes,encryptedData.length,&dataMoved);

    if (result == kCCSuccess) {
        encryptedData.length = dataMoved;
        return encryptedData;
    }

    return nil;

}
/**
 *  @brief   利用3DES解密数据
 */
- (NSData*)decryptedWith3DESUsingKey:(NSString*)key andIV:(NSData*)iv {

    NSData *keyData = [key dataUsingEncoding:NSUTF8StringEncoding];

    size_t dataMoved;
    NSMutableData *decryptedData = [NSMutableData dataWithLength:self.length + kCCBlockSize3DES];

    CCCryptorStatus result = CCCrypt(kCCDecrypt,kCCAlgorithm3DES,kCCOptionPKCS7Padding,keyData.bytes,keyData.length,iv.bytes,self.bytes,self.length,decryptedData.mutableBytes,decryptedData.length,&dataMoved);

    if (result == kCCSuccess) {
        decryptedData.length = dataMoved;
        return decryptedData;
    }

    return nil;

}
```

#### 30、单个页面多个网络请求的情况，需要监听所有网络请求结束后刷新UI
```objectivec
dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t serialQueue = dispatch_queue_create("com.wzb.test.www", DISPATCH_QUEUE_SERIAL);
    dispatch_group_enter(group);
    dispatch_group_async(group, serialQueue, ^{
        // 网络请求一
        [WebClick getDataSuccess:^(ResponseModel *model) {
            dispatch_group_leave(group);
        } failure:^(NSString *err) {
            dispatch_group_leave(group);
        }];
    });
    dispatch_group_enter(group);
    dispatch_group_async(group, serialQueue, ^{
        // 网络请求二
        [WebClick getDataSuccess:getBigTypeRM onSuccess:^(ResponseModel *model) {
            dispatch_group_leave(group);
        }                                  failure:^(NSString *errorString) {
            dispatch_group_leave(group);
        }];
    });
    dispatch_group_enter(group);
    dispatch_group_async(group, serialQueue, ^{
        // 网络请求三
        [WebClick getDataSuccess:^{
            dispatch_group_leave(group);
        } failure:^(NSString *errorString) {
            dispatch_group_leave(group);
        }];
    });

    // 所有网络请求结束后会来到这个方法
    dispatch_group_notify(group, serialQueue, ^{
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            dispatch_async(dispatch_get_main_queue(), ^{
                // 刷新UI
            });
        });
    });
```


