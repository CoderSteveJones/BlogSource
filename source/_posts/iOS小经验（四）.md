---
title: iOS小经验（四）
date: 2016-02-26 09:53:06
categories: 
	- iOS小经验
---
#### 1、获取视频的时长
```objectivec
+ (NSInteger)getVideoTimeByUrlString:(NSString *)urlString {
    NSURL *videoUrl = [NSURL URLWithString:urlString];
    AVURLAsset *avUrl = [AVURLAsset assetWithURL:videoUrl];
    CMTime time = [avUrl duration];
    int seconds = ceil(time.value/time.timescale);
    return seconds;
}
```

#### 2、字符串是否为空
```objectivec
+ (BOOL)isEqualToNil:(NSString *)str {
    return str.length <= 0 || [str isEqualToString:@""] || !str;
}
```
#### 3、将app上传到App Store的时候通常会遇到这个问题

![](http://upload-images.jianshu.io/upload_images/1432270-8b75d749ac20c31e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>很多人说这事苹果爸爸服务器问题，重复尝试几次，总会成功的！
但是经过尝试发现如果使用Application Loader上传成功率就非常高，所以还是推荐把ipa文件导出直接用Application Loader上传。
如果Application Loader也不行，需要检查下自己的网络，有时候vpn也会提高速度。

#### 4、当tableView占不满一屏时，去除下边多余的单元格
```objectivec
self.tableView.tableHeaderView = [UIView new];
self.tableView.tableFooterView = [UIView new];
```

#### 5、isKindOfClass和isMemberOfClass的区别
```objectivec
isKindOfClass可以判断某个对象是否属于某个类，或者这个类的子类。
isMemberOfClass更加精准，它只能判断这个对象类型是否为这个类(不能判断子类)
```

#### 6、__block
```objectivec
当一个局部变量需要在block里改变时，需要在定义时加上__block修饰，具体请看官方文档 http://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Blocks/Articles/bxVariables.html#//apple_ref/doc/uid/TP40007502-CH6-SW6
```

#### 7、-[ViewController aMethod:]: unrecognized selector sent to instance 0x7fe91e607fb0
> 这是一个经典错误，ViewController不能响应aMethod这个方法，错误原因可能viewController文件中没有实现aMethod这个方法

#### 8、UITableView (<UITableView: 0x7ff19b027000; >) failed to obtain a cell from its dataSource (<ViewController: 0x7ff19a507520>)

>这个错误原因是tableView的代理方法-tableView:cellForRowAtIndexPath:需要返回一个UITableViewCell,而你返回了一个nil。另外这个地方返回值不是UITableViewCell类型也会导致崩溃

#### 9、约束如何做UIView动画？
```objectivec
1、把需要改的约束Constraint拖条线出来，成为属性
2、在需要动画的地方加入代码，改变此属性的constant属性
3、开始做UIView动画，动画里边调用layoutIfNeeded方法

@property (weak, nonatomic) IBOutlet NSLayoutConstraint *buttonTopConstraint;
self.buttonTopConstraint.constant = 100;
    [UIView animateWithDuration:.5 animations:^{
        [self.view layoutIfNeeded];
    }];
```

#### 10、从NSURL中拿到链接字符串
```objectivec
NSString *urlString = myURL.absoluteString;
```

#### 11、将tableView滚动到顶部
```objectivec
[tableView setContentOffset:CGPointZero animated:YES];
或者
[tableView scrollRectToVisible:CGRectMake(0, 0, 1, 1) animated:YES];
```

#### 12、如果用addTarget:action:forControlEvents:方法为一个button添加了很多点击事件，在某个时刻想一次删除怎么办？只需要调用下边这句代码
```objectivec
[youButton removeTarget:nil action:nil forControlEvents:UIControlEventAllEvents];
```
#### 13、某个字体的高度
```objectivec
font.lineHeight;
```

#### 14、删除某个view所有的子视图
```objectivec
[[someView subviews]
 makeObjectsPerformSelector:@selector(removeFromSuperview)];
```

#### 15、删除NSUserDefaults所有记录
```objectivec
//方法一
  NSString *appDomain = [[NSBundle mainBundle] bundleIdentifier];
 [[NSUserDefaults standardUserDefaults] removePersistentDomainForName:appDomain];   
 //方法二  
- (void)resetDefaults {   
  NSUserDefaults * defs = [NSUserDefaults standardUserDefaults];
     NSDictionary * dict = [defs dictionaryRepresentation];
     for (id key in dict) {
          [defs removeObjectForKey:key];
     }
      [defs synchronize];
 }
// 方法三
[[NSUserDefaults standardUserDefaults] setPersistentDomain:[NSDictionary dictionary] forName:[[NSBundle mainBundle] bundleIdentifier]];
```

#### 16、禁用系统滑动返回功能
```objectivec
- (void)viewDidAppear:(BOOL)animated
{
     [super viewDidAppear:animated];
if ([self.navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {self.navigationController.interactivePopGestureRecognizer.delegate = self;
    }
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    if ([self.navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {self.navigationController.interactivePopGestureRecognizer.delegate = nil;
    }
}
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
{
     return NO;
}
```

#### 17、模拟器报错
![](http://upload-images.jianshu.io/upload_images/1432270-76da2a462b9f1eca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>解决办法：
打开模拟器->Simulator->Reset Content and Settings...
如果不行，就重启试试！

#### 18、自定义cell选中背景颜色
```objectivec
UIView *bgColorView = [[UIView alloc] init];
bgColorView.backgroundColor = [UIColor redColor];
[cell setSelectedBackgroundView:bgColorView];
```

#### 19、UILabel设置内边距
```objectivec
子类化UILabel，重写drawTextInRect方法
- (void)drawTextInRect:(CGRect)rect {
    // 边距，上左下右
    UIEdgeInsets insets = {0, 5, 0, 5};
    [super drawTextInRect:UIEdgeInsetsInsetRect(rect, insets)];
}
```

#### 20、UILabel设置文字描边
```objectivec
子类化UILabel，重写drawTextInRect方法
- (void)drawTextInRect:(CGRect)rect
{
    CGContextRef c = UIGraphicsGetCurrentContext();
    // 设置描边宽度
    CGContextSetLineWidth(c, 1);
    CGContextSetLineJoin(c, kCGLineJoinRound);
    CGContextSetTextDrawingMode(c, kCGTextStroke);
    // 描边颜色
    self.textColor = [UIColor redColor];
    [super drawTextInRect:rect];
    // 文本颜色
    self.textColor = [UIColor yellowColor];
    CGContextSetTextDrawingMode(c, kCGTextFill);
    [super drawTextInRect:rect];
}
```

#### 21、使用模拟器截图
```objectivec
快捷键command + s
或者File->Save Screen Shot
```

#### 22、scrollView滚动到最下边
```objectivec
CGPoint bottomOffset = CGPointMake(0, scrollView.contentSize.height - scrollView.bounds.size.height);
[scrollView setContentOffset:bottomOffset animated:YES];
```

#### 23、UIView背景颜色渐变
```objectivec
    UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 100)];
    [self.view addSubview:view];
    CAGradientLayer *gradient = [CAGradientLayer layer];
    gradient.frame = view.bounds;
    gradient.colors = [NSArray arrayWithObjects:(id)[[UIColor blackColor] CGColor], (id)[[UIColor whiteColor] CGColor], nil];
    [view.layer insertSublayer:gradient atIndex:0];
```

#### 24、停止UIView动画
```objectivec
[yourView.layer removeAllAnimations]
```

#### 25、为UIView某个角添加圆角
```objectivec
// 左上角和右下角添加圆角
UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:view.bounds byRoundingCorners:(UIRectCornerTopLeft | UIRectCornerBottomRight) cornerRadii:CGSizeMake(20, 20)];
    CAShapeLayer *maskLayer = [CAShapeLayer layer];
    maskLayer.frame = view.bounds;
    maskLayer.path = maskPath.CGPath;
    view.layer.mask = maskLayer;
```

#### 26、删除Xcode Derived data缓存数据
![](http://upload-images.jianshu.io/upload_images/1432270-6da089f7000ad432.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####  27、将一个view放置在其兄弟视图的最上面
```objectivec
[parentView bringSubviewToFront:yourView]
```

#### 28、将一个view放置在其兄弟视图的最下面
```objectivec
[parentView sendSubviewToBack:yourView]
```

#### 29、让手机震动一下
```objectivec
倒入框架
#import <AudioToolbox/AudioToolbox.h>
AudioServicesPlayAlertSound(kSystemSoundID_Vibrate);
或者
AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
```

#### 30、layoutSubviews方法什么时候调用？

>1、init方法不会调用
2、addSubview方法等时候会调用
3、bounds改变的时候调用
4、scrollView滚动的时候会调用scrollView的layoutSubviews方法(所以不建议在scrollView的layoutSubviews方法中做复杂逻辑)
5、旋转设备的时候调用
6、子视图被移除的时候调用
参考请看：http://blog.logichigh.com/2011/03/16/when-does-layoutsubviews-get-called/



