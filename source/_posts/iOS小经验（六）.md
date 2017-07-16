---
title: iOS小经验（六）
date: 2016-03-17 09:53:06
categories: 
	- iOS小经验
---


#### 1、每个cell之间增加间距
```objectivec
// 方法一，每个分区只显示一行cell，分区头当作你想要的间距(注意，从数据源数组中取值的时候需要用indexPath.section而不是indexPath.row)
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return yourArry.count;
}
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return 1;
}
-(CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
    return cellSpacingHeight;
}

// 方法二，在cell的contentView上加个稍微低一点的view，cell上原本的内容放在你的view上，而不是contentView上，这样能伪造出一个间距来。

// 方法三，自定义cell，重写setFrame：方法
- (void)setFrame:(CGRect)frame
{
    frame.size.height -= 20;
    [super setFrame:frame];
}
```

#### 7、播放一张张连续的图片
```objectivec
// 加入现在有三张图片分别为animate_1、animate_2、animate_3
// 方法一
    imageView.animationImages = @[[UIImage imageNamed:@"animate_1"], [UIImage imageNamed:@"animate_2"], [UIImage imageNamed:@"animate_3"]];
imageView.animationDuration = 1.0;
// 方法二
    imageView.image = [UIImage animatedImageNamed:@"animate_" duration:1.0];
// 方法二解释下，这个方法会加载animate_为前缀的，后边0-1024，也就是animate_0、animate_1一直到animate_1024
```

#### 3、加载gif图片

>推荐使用这个框架 [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage)

#### 4、防止离屏渲染为image添加圆角
```objectivec
// image分类
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

#### 5、查看系统所有字体
```objectivec
// 打印字体
for (id familyName in [UIFont familyNames]) {
    NSLog(@"%@", familyName);
    for (id fontName in [UIFont fontNamesForFamilyName:familyName]) NSLog(@"  %@", fontName);
}
// 也可以进入这个网址查看 http://iosfonts.com/

```

#### 6、获取随机数
```objectivec
NSInteger i = arc4random();
```

#### 7、获取随机数小数(0-1之间)
```objectivec
#define ARC4RANDOM_MAX      0x100000000
double val = ((double)arc4random() / ARC4RANDOM_MAX);
```

#### 8、AVPlayer视频播放完成的通知监听
```objectivec
[[NSNotificationCenter defaultCenter] 
      addObserver:self
      selector:@selector(videoPlayEnd)
      name:AVPlayerItemDidPlayToEndTimeNotification 
      object:nil];

```

#### 9、判断两个rect是否有交叉
```objectivec
 if (CGRectIntersectsRect(rect1, rect2)) {
}
```

#### 10、判断一个字符串是否为数字
```objectivec
NSCharacterSet *notDigits = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
    if ([str rangeOfCharacterFromSet:notDigits].location == NSNotFound)
    {
      // 是数字
    } else
    {
      // 不是数字
    }

```
#### 11、将一个view保存为pdf格式
```objectivec
- (void)createPDFfromUIView:(UIView*)aView saveToDocumentsWithFileName:(NSString*)aFilename
{
    NSMutableData *pdfData = [NSMutableData data];
    UIGraphicsBeginPDFContextToData(pdfData, aView.bounds, nil);
    UIGraphicsBeginPDFPage();
    CGContextRef pdfContext = UIGraphicsGetCurrentContext();
    [aView.layer renderInContext:pdfContext];
    UIGraphicsEndPDFContext();

    NSArray* documentDirectories = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask,YES);
    NSString* documentDirectory = [documentDirectories objectAtIndex:0];
    NSString* documentDirectoryFilename = [documentDirectory stringByAppendingPathComponent:aFilename];
    [pdfData writeToFile:documentDirectoryFilename atomically:YES];
    NSLog(@"documentDirectoryFileName: %@",documentDirectoryFilename);
}

```

#### 12、让一个view在父视图中心
```objectivec
child.center = [parent convertPoint:parent.center fromView:parent.superview];
```

#### 13、获取当前导航控制器下前一个控制器
```objectivec
- (UIViewController *)backViewController
{
    NSInteger myIndex = [self.navigationController.viewControllers indexOfObject:self];

    if ( myIndex != 0 && myIndex != NSNotFound ) {
        return [self.navigationController.viewControllers objectAtIndex:myIndex-1];
    } else {
        return nil;
    }
}
```

#### 14、保存UIImage到本地
```objectivec
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *filePath = [[paths objectAtIndex:0] stringByAppendingPathComponent:@"Image.png"];
[UIImagePNGRepresentation(image) writeToFile:filePath atomically:YES];
```

#### 15、键盘上方增加工具栏
```objectivec
UIToolbar *keyboardDoneButtonView = [[UIToolbar alloc] init];
[keyboardDoneButtonView sizeToFit];
UIBarButtonItem *doneButton = [[UIBarButtonItem alloc] initWithTitle:@"Done"
                                                               style:UIBarButtonItemStyleBordered target:self
                                                              action:@selector(doneClicked:)];
[keyboardDoneButtonView setItems:[NSArray arrayWithObjects:doneButton, nil]];
txtField.inputAccessoryView = keyboardDoneButtonView;

```

#### 16、copy一个view
>因为UIView没有实现copy协议，因此找不到copyWithZone方法，使用copy的时候导致崩溃
但是我们可以通过归档再解档实现copy，这相当于对视图进行了一次深拷贝，代码如下

```objectivec
id copyOfView = 
[NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:originalView]];
```

#### 17、在image上绘制文字并生成新的image
```objectivec
UIFont *font = [UIFont boldSystemFontOfSize:12];
    UIGraphicsBeginImageContext(image.size);
    [image drawInRect:CGRectMake(0,0,image.size.width,image.size.height)];
    CGRect rect = CGRectMake(point.x, point.y, image.size.width, image.size.height);
    [[UIColor whiteColor] set];
    [text drawInRect:CGRectIntegral(rect) withFont:font]; 
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
```

#### 18、判断一个view是否为另一个view的子视图
```objectivec
// 如果myView是self.view本身，也会返回yes
BOOL isSubView = [myView isDescendantOfView:self.view];
```

#### 19、判断一个字符串是否包含另一个字符串
```objectivec
// 方法一、这种方法只适用于iOS8之后，如果是配iOS8之前用方法二
if ([str containsString:otherStr]) NSLog(@"包含");

// 方法二
NSRange range = [str rangeOfString:otherStr];
if (range.location != NSNotFound) NSLog(@"包含");
```

#### 20、UICollectionView自动滚动到某行
```objectivec
// 重写viewDidLayoutSubviews方法
-(void)viewDidLayoutSubviews {
   [super viewDidLayoutSubviews];
   [self.collectionView scrollToItemAtIndexPath:indexPath atScrollPosition:UICollectionViewScrollPositionCenteredVertically animated:NO];
}
```

#### 21、修改系统UIAlertController
```objectivec
// 但是据说这种方法会被App Store拒绝(慎用！)
UIAlertController *alertVC = [UIAlertController alertControllerWithTitle:@"" message:@"" preferredStyle:UIAlertControllerStyleActionSheet];
    NSMutableAttributedString *hogan = [[NSMutableAttributedString alloc] initWithString:@"我是一个大文本"];
    [hogan addAttribute:NSFontAttributeName
                  value:[UIFont systemFontOfSize:30]
                  range:NSMakeRange(4, 1)];
    [hogan addAttribute:NSForegroundColorAttributeName
                  value:[UIColor redColor]
                  range:NSMakeRange(4, 1)];
    [alertVC setValue:hogan forKey:@"attributedTitle"];

    UIAlertAction *button = [UIAlertAction actionWithTitle:@"Label text" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action){ }];
    UIImage *accessoryImage = [UIImage imageNamed:@"1"];
    [button setValue:accessoryImage forKey:@"image"];
    [alertVC addAction:button];
    [self presentViewController:alertVC animated:YES completion:nil];
```

#### 22、判断某一行的cell是否已经显示
```objectivec
CGRect cellRect = [tableView rectForRowAtIndexPath:indexPath];
BOOL completelyVisible = CGRectContainsRect(tableView.bounds, cellRect);
```

#### 23、让导航控制器pop回指定的控制器
```objectivec
NSMutableArray *allViewControllers = [NSMutableArray arrayWithArray:[self.navigationController viewControllers]];
for (UIViewController *aViewController in allViewControllers) {
    if ([aViewController isKindOfClass:[RequiredViewController class]]) {
        [self.navigationController popToViewController:aViewController animated:NO];
    }
}

```

#### 24、动画修改label上的文字
```objectivec
// 方法一
CATransition *animation = [CATransition animation];
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    animation.type = kCATransitionFade;
    animation.duration = 0.75;
    [self.label.layer addAnimation:animation forKey:@"kCATransitionFade"];
    self.label.text = @"New";

// 方法二
[UIView transitionWithView:self.label
                      duration:0.25f
                       options:UIViewAnimationOptionTransitionCrossDissolve
                    animations:^{

                        self.label.text = @"Well done!";

                    } completion:nil];

// 方法三
[UIView animateWithDuration:1.0
                     animations:^{
                         self.label.alpha = 0.0f;
                         self.label.text = @"newText";
                         self.label.alpha = 1.0f;
                     }];

```

#### 25、判断字典中是否包含某个key值
```objectivec
if ([dic objectForKey:@"yourKey"]) {
    NSLog(@"有这个值");
} else {
    NSLog(@"没有这个值");
}
```

#### 26、获取屏幕方向
```objectivec
UIInterfaceOrientation orientation = [UIApplication sharedApplication].statusBarOrientation;

if(orientation == 0) //Default orientation 
    //默认
else if(orientation == UIInterfaceOrientationPortrait)
    //竖屏
else if(orientation == UIInterfaceOrientationLandscapeLeft)
    // 左横屏
else if(orientation == UIInterfaceOrientationLandscapeRight)
    //右横屏
```

#### 27、设置UIImage的透明度
```objectivec
// 方法一、添加UIImage分类
- (UIImage *)imageByApplyingAlpha:(CGFloat) alpha {
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0f);

    CGContextRef ctx = UIGraphicsGetCurrentContext();
    CGRect area = CGRectMake(0, 0, self.size.width, self.size.height);

    CGContextScaleCTM(ctx, 1, -1);
    CGContextTranslateCTM(ctx, 0, -area.size.height);

    CGContextSetBlendMode(ctx, kCGBlendModeMultiply);

    CGContextSetAlpha(ctx, alpha);

    CGContextDrawImage(ctx, area, self.CGImage);

    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();

    UIGraphicsEndImageContext();

    return newImage;
}

// 方法二、如果没有奇葩需求，干脆用UIImageView设置透明度
UIImageView *imageView = [[UIImageView alloc] initWithImage:[UIImage imageWithName:@"yourImage"]];
imageView.alpha = 0.5;
```

#### 28、Attempt to mutate immutable object with insertString:atIndex:

>这个错是因为你拿字符串调用insertString:atIndex:方法的时候，调用对象不是NSMutableString，应该先转成这个类型再调用

#### 29、UIWebView添加单击手势不响应
```objectivec
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(webViewClick)];
        tap.delegate = self;
        [_webView addGestureRecognizer:tap];

// 因为webView本身有一个单击手势，所以再添加会造成手势冲突，从而不响应。需要绑定手势代理，并实现下边的代理方法
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer{
    return YES;
}
```

#### 30、获取手机RAM容量
```objectivec
// 需要导入#import <mach/mach.h>
mach_port_t host_port;
    mach_msg_type_number_t host_size;
    vm_size_t pagesize;

    host_port = mach_host_self();
    host_size = sizeof(vm_statistics_data_t) / sizeof(integer_t);
    host_page_size(host_port, &pagesize);

    vm_statistics_data_t vm_stat;

    if (host_statistics(host_port, HOST_VM_INFO, (host_info_t)&vm_stat, &host_size) != KERN_SUCCESS) {
        NSLog(@"Failed to fetch vm statistics");
    }

    /* Stats in bytes */
    natural_t mem_used = (vm_stat.active_count +
                          vm_stat.inactive_count +
                          vm_stat.wire_count) * pagesize;
    natural_t mem_free = vm_stat.free_count * pagesize;
    natural_t mem_total = mem_used + mem_free;
    NSLog(@"已用: %u 可用: %u 总共: %u", mem_used, mem_free, mem_total);
```

