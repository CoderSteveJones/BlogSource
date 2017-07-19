---
title: iOS小经验（八）
date: 2016-03-25 09:53:06
categories: 
	- 小经验
---


#### 1、解决openUrl延时问题
```objectivec
// 方法一
dispatch_async(dispatch_get_main_queue(), ^{

    UIApplication *application = [UIApplication sharedApplication];
    if ([application respondsToSelector:@selector(openURL:options:completionHandler:)]) {
        [application openURL:URL options:@{}
           completionHandler:nil];
    } else {
        [application openURL:URL];
    }
    });
// 方法二
[self performSelector:@selector(redirectToURL:) withObject:url afterDelay:0.1];

- (void) redirectToURL
{
UIApplication *application = [UIApplication sharedApplication];
    if ([application respondsToSelector:@selector(openURL:options:completionHandler:)]) {
        [application openURL:URL options:@{}
           completionHandler:nil];
    } else {
        [application openURL:URL];
    }
}
```

#### 2、页面跳转实现翻转动画
```objectivec
// modal方式
    TestViewController *vc = [[TestViewController alloc] init];
    vc.view.backgroundColor = [UIColor redColor];
    vc.modalTransitionStyle = UIModalTransitionStyleCoverVertical;
    [self presentViewController:vc animated:YES completion:nil];

// push方式
    TestViewController *vc = [[TestViewController alloc] init];
    vc.view.backgroundColor = [UIColor redColor];
    [UIView beginAnimations:@"View Flip" context:nil];
    [UIView setAnimationDuration:0.80];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [UIView setAnimationTransition:UIViewAnimationTransitionFlipFromRight forView:self.navigationController.view cache:NO];
    [self.navigationController pushViewController:vc animated:YES];
    [UIView commitAnimations];
```

#### 3、tableView实现无限滚动
```objectivec
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    CGFloat actualPosition = scrollView.contentOffset.y;
    CGFloat contentHeight = scrollView.contentSize.height - scrollView.frame.size.height;
    if (actualPosition >= contentHeight) {
        [self.dataArr addObjectsFromArray:self.dataArr];
        [self.tableView reloadData];
    }
}
```

#### 4、代码方式调整屏幕亮度
```objectivec
// brightness属性值在0-1之间，0代表最小亮度，1代表最大亮度
[[UIScreen mainScreen] setBrightness:0.5];
```


#### 5、获取当前应用CUP用量
```objectivec
float cpu_usage()
{
    kern_return_t kr;
    task_info_data_t tinfo;
    mach_msg_type_number_t task_info_count;

    task_info_count = TASK_INFO_MAX;
    kr = task_info(mach_task_self(), TASK_BASIC_INFO, (task_info_t)tinfo, &task_info_count);
    if (kr != KERN_SUCCESS) {
        return -1;
    }

    task_basic_info_t      basic_info;
    thread_array_t         thread_list;
    mach_msg_type_number_t thread_count;

    thread_info_data_t     thinfo;
    mach_msg_type_number_t thread_info_count;

    thread_basic_info_t basic_info_th;
    uint32_t stat_thread = 0; // Mach threads

    basic_info = (task_basic_info_t)tinfo;

    // get threads in the task
    kr = task_threads(mach_task_self(), &thread_list, &thread_count);
    if (kr != KERN_SUCCESS) {
        return -1;
    }
    if (thread_count > 0)
        stat_thread += thread_count;

    long tot_sec = 0;
    long tot_usec = 0;
    float tot_cpu = 0;
    int j;

    for (j = 0; j < (int)thread_count; j++)
    {
        thread_info_count = THREAD_INFO_MAX;
        kr = thread_info(thread_list[j], THREAD_BASIC_INFO,
                         (thread_info_t)thinfo, &thread_info_count);
        if (kr != KERN_SUCCESS) {
            return -1;
        }

        basic_info_th = (thread_basic_info_t)thinfo;

        if (!(basic_info_th->flags & TH_FLAGS_IDLE)) {
            tot_sec = tot_sec + basic_info_th->user_time.seconds + basic_info_th->system_time.seconds;
            tot_usec = tot_usec + basic_info_th->user_time.microseconds + basic_info_th->system_time.microseconds;
            tot_cpu = tot_cpu + basic_info_th->cpu_usage / (float)TH_USAGE_SCALE * 100.0;
        }

    } // for each thread

    kr = vm_deallocate(mach_task_self(), (vm_offset_t)thread_list, thread_count * sizeof(thread_t));
    assert(kr == KERN_SUCCESS);

    return tot_cpu;
}
```

#### 6、float数据取整四舍五入
```objectivec
    CGFloat f = 4.65;
    NSLog(@"%d", (int)f);    // 打印结果4

    CGFloat f = 4.65;
    NSLog(@"%d", (int)round(f));    // 打印结果5
```

#### 7、删除UISearchBar系统默认边框
```objectivec
    // 方法一
    searchBar.searchBarStyle = UISearchBarStyleMinimal;

    // 方法二
    [searchBar setBackgroundImage:[[UIImage alloc]init]];

    // 方法三
    searchBar.barTintColor = [UIColor whiteColor];

```

#### 8、为UICollectionViewCell设置圆角和阴影
```objectivec
cell.contentView.layer.cornerRadius = 2.0f;
cell.contentView.layer.borderWidth = 1.0f;
cell.contentView.layer.borderColor = [UIColor clearColor].CGColor;
cell.contentView.layer.masksToBounds = YES;

cell.layer.shadowColor = [UIColor lightGrayColor].CGColor;
cell.layer.shadowOffset = CGSizeMake(0, 2.0f);
cell.layer.shadowRadius = 2.0f;
cell.layer.shadowOpacity = 1.0f;
cell.layer.masksToBounds = NO;
cell.layer.shadowPath = [UIBezierPath bezierPathWithRoundedRect:cell.bounds cornerRadius:cell.contentView.layer.cornerRadius].CGPath;

```
#### 9、让正在滑动的scrollView停止滚动(不是禁止，而是暂时停止滚动)
```objectivec
[scrollView setContentOffset:scrollView.contentOffset animated:NO];
```


#### 10、让正在滑动的scrollView停止滚动(不是禁止，而是暂时停止滚动)
```objectivec
[scrollView setContentOffset:scrollView.contentOffset animated:NO];
```

#### 11、根据经纬度获取城市等信息
```objectivec
// 创建经纬度
    CLLocation *location = [[CLLocation alloc] initWithLatitude:latitude longitude:longitude];
    //创建一个译码器
    CLGeocoder *cLGeocoder = [[CLGeocoder alloc] init];
    [cLGeocoder reverseGeocodeLocation:userLocation completionHandler:^(NSArray *placemarks, NSError *error) {
        CLPlacemark *place = [placemarks objectAtIndex:0];
        // 位置名
    　　NSLog(@"name,%@",place.name);
    　　// 街道
    　　NSLog(@"thoroughfare,%@",place.thoroughfare);
    　　// 子街道
    　　NSLog(@"subThoroughfare,%@",place.subThoroughfare);
    　　// 市
    　　NSLog(@"locality,%@",place.locality);
    　　// 区
    　　NSLog(@"subLocality,%@",place.subLocality); 
    　　// 国家
    　　NSLog(@"country,%@",place.country);
        }
    }];

/*  CLPlacemark中属性含义
name                    地名

thoroughfare            街道

subThoroughfare        街道相关信息，例如门牌等

locality                城市

subLocality            城市相关信息，例如标志性建筑

administrativeArea      直辖市

subAdministrativeArea  其他行政区域信息（自治区等）

postalCode              邮编

ISOcountryCode          国家编码

country                国家

inlandWater            水源，湖泊

ocean                  海洋

areasOfInterest        关联的或利益相关的地标
*/
```


#### 12、如何防止添加多个NSNotification观察者？
```objectivec
// 解决方案就是添加观察者之前先移除下这个观察者
[[NSNotificationCenter defaultCenter] removeObserver:observer name:name object:object];
        [[NSNotificationCenter defaultCenter] addObserver:observer selector:selector name:name object:object];
```

#### 13、将一个xib添加到另外一个xib上
```objectivec
// 假设你的自定义view名字为CustomView，你需要在CustomView.m中重写 `- (instancetype)initWithCoder:(NSCoder *)aDecoder` 方法，代码如下：
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
    if ((self = [super initWithCoder:aDecoder])) {
        [self addSubview:[[[NSBundle mainBundle] loadNibNamed:@"CustomView" owner:self options:nil] objectAtIndex:0]];
    }
    return self;
}
```
![](http://upload-images.jianshu.io/upload_images/1432270-1baa2a4dd2e2d08f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 14、处理字符串，使其首字母大写
```objectivec
    NSString *str = @"abcdefghijklmn";
    NSString *resultStr;
    if (str && str.length > 0) {
        resultStr = [str stringByReplacingCharactersInRange:NSMakeRange(0,1) withString:[[str substringToIndex:1] capitalizedString]];
    }
    NSLog(@"%@", resultStr);
```

#### 15、判断一个UIAlertView/UIAlertController是否显示
```objectivec
// UIAlertView自带属性
if (alert.visible)
{
      NSLog(@"显示了");
} else {
      NSLog(@"未显示");
}

// UIAlertController没有visible属性，需要自己判断，添加一个全局变量 BOOL visible
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *alertAction = [UIAlertAction actionWithTitle:@"ActionTitle" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        self.visible = NO;
    }];
    UIAlertAction *calcelAction = [UIAlertAction actionWithTitle:@"calcelTitle" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        self.visible = NO;
    }];
    [alertController addAction:alertAction];
    [alertController addAction:calcelAction];
    [self presentViewController:alertController animated:YES completion:^{
        self.visible = YES;
    }];

```

#### 16、获取字符串中的数字
```objectivec
- (NSString *)getNumberFromStr:(NSString *)str
{
    NSCharacterSet *nonDigitCharacterSet = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
    return [[str componentsSeparatedByCharactersInSet:nonDigitCharacterSet] componentsJoinedByString:@""];
}

    NSLog(@"%@", [self getNumberFromStr:@"a0b0c1d2e3f4fda8fa8fad9fsad23"]); // 00123488923
```

#### 17、为UIView的某个方向添加边框
```objectivec
// 添加UIView分类

// UIView+WZB.h
#import <UIKit/UIKit.h>


/**
 边框方向

 - WZBBorderDirectionTop: 顶部
 - WZBBorderDirectionLeft: 左边
 - WZBBorderDirectionBottom: 底部
 - WZBBorderDirectionRight: 右边
 */
typedef NS_ENUM(NSInteger, WZBBorderDirectionType) {
    WZBBorderDirectionTop = 0,
    WZBBorderDirectionLeft,
    WZBBorderDirectionBottom,
    WZBBorderDirectionRight
};

@interface UIView (WZB)


/**
 为UIView的某个方向添加边框

 @param direction 边框方向
 @param color 边框颜色
 @param width 边框宽度
 */
- (void)wzb_addBorder:(WZBBorderDirectionType)direction color:(UIColor *)color width:(CGFloat)width;

@end

// UIView+WZB.m
#import "UIView+WZB.h"

@implementation UIView (WZB)

- (void)wzb_addBorder:(WZBBorderDirectionType)direction color:(UIColor *)color width:(CGFloat)width
{
    CALayer *border = [CALayer layer];
    border.backgroundColor = color.CGColor;
    switch (direction) {
        case WZBBorderDirectionTop:
        {
            border.frame = CGRectMake(0.0f, 0.0f, self.bounds.size.width, width);
        }
            break;
        case WZBBorderDirectionLeft:
        {
            border.frame = CGRectMake(0.0f, 0.0f, width, self.bounds.size.height);
        }
            break;
        case WZBBorderDirectionBottom:
        {
            border.frame = CGRectMake(0.0f, self.bounds.size.height - width, self.bounds.size.width, width);
        }
            break;
        case WZBBorderDirectionRight:
        {
            border.frame = CGRectMake(self.bounds.size.width - width, 0, width, self.bounds.size.height);
        }
            break;

        default:
            break;
    }
    [self.layer addSublayer:border];
}

```

#### 18、通过属性设置UISwitch、UIProgressView等控件的宽高
```objectivec
mySwitch.transform = CGAffineTransformMakeScale(5.0f, 5.0f);
progressView.transform = CGAffineTransformMakeScale(5.0f, 5.0f);

```

#### 19、自动搜索功能，用户连续输入的时候不搜索，用户停止输入的时候自动搜索(我这里设置的是0.5s，可根据需求更改)
```objectivec
// 输入框文字改变的时候调用
-(void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText{
    // 先取消调用搜索方法
    [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(searchNewResult) object:nil];
    // 0.5秒后调用搜索方法
    [self performSelector:@selector(searchNewResult) withObject:nil afterDelay:0.5];
}

```

#### 20、修改UISearchBar的占位文字颜色
```objectivec
    // 方法一（推荐使用）
    UITextField *searchField = [searchBar valueForKey:@"_searchField"];
    [searchField setValue:[UIColor blueColor] forKeyPath:@"_placeholderLabel.textColor"];

    // 方法二（已过期）
    [[UILabel appearanceWhenContainedIn:[UISearchBar class], nil] setTextColor:[UIColor redColor]];

    // 方法三（已过期）
    NSDictionary *placeholderAttributes = @{NSForegroundColorAttributeName : [UIColor redColor], NSFontAttributeName : [UIFont fontWithName:@"HelveticaNeue" size:15],};
    NSAttributedString *attributedPlaceholder = [[NSAttributedString alloc] initWithString:searchBar.placeholder attributes:placeholderAttributes];
    [[UITextField appearanceWhenContainedIn:[UISearchBar class], nil] setAttributedPlaceholder:attributedPlaceholder];
```

#### 21、某个界面多个事件同时响应引起的问题(比如，两个button同时按push到新界面，两个都会响应，可能导致push重叠)
```objectivec
// UIView有个属性叫做exclusiveTouch，设置为YES后，其响应事件会和其他view互斥(有其他view事件响应的时候点击它不起作用)
view.exclusiveTouch = YES;

// 一个一个设置太麻烦了，可以全局设置
[[UIView appearance] setExclusiveTouch:YES];

// 或者只设置button
[[UIButton appearance] setExclusiveTouch:YES];
```

#### 22、修改tabBar的frame
```objectivec
// 子类化UITabBarViewController，我这里以修改tabBar高度为例，重写viewWillLayoutSubviews方法
#import "WZBTabBarViewController.h"

@interface WZBTabBarViewController ()

@end

@implementation WZBTabBarViewController
- (void)viewWillLayoutSubviews {

    CGRect tabFrame = self.tabBar.frame;
    tabFrame.size.height = 100;
    tabFrame.origin.y = self.view.frame.size.height - 100;
    self.tabBar.frame = tabFrame;
}
@end
```

#### 23、修改键盘背景颜色
```objectivec
// 设置某个键盘颜色
textField.keyboardAppearance = UIKeyboardAppearanceAlert;
// 设置工程中所有键盘颜色
[[UITextField appearance] setKeyboardAppearance:UIKeyboardAppearanceAlert];
```

#### 24、修改image颜色
```objectivec
UIImage *image = [UIImage imageNamed:@"test"];
imageView.image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
CGRect rect = CGRectMake(0, 0, image.size.width, image.size.height);
UIGraphicsBeginImageContext(rect.size);
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextClipToMask(context, rect, image.CGImage);
CGContextSetFillColorWithColor(context, [[UIColor redColor] CGColor]);
CGContextFillRect(context, rect);
UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
UIImage *flippedImage = [UIImage imageWithCGImage:img.CGImage scale:1.0 orientation: UIImageOrientationDownMirrored];
imageView.image = flippedImage;
``` 

#### 25、动画执行removeFromSuperview
```objectivec
[UIView animateWithDuration:0.2
                     animations:^{
                         view.alpha = 0.0f;
                     } completion:^(BOOL finished){
                         [view removeFromSuperview];
                     }];
```
#### 26、启动页显示延时
```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
//  延时10s
    sleep(10);
    return YES;
}
```

#### 27、设置UIButton高亮时的背景颜色
```objectivec
// 方法一、子类化UIButton，重写setHighlighted:方法，代码如下
#import "WZBButton.h"

@implementation WZBButton

- (void)setHighlighted:(BOOL)highlighted {
    [super setHighlighted:highlighted];

    UIColor *normalColor = [UIColor greenColor];
    UIColor *highlightedColor = [UIColor redColor];
    self.backgroundColor = highlighted ? highlightedColor : normalColor;

}

// 方法二、利用setBackgroundImage:forState:方法
[button setBackgroundImage:[self imageWithColor:[UIColor blueColor]] forState:UIControlStateHighlighted];

- (UIImage *)imageWithColor:(UIColor *)color {
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();

    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);

    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();

    return image;
}
```

#### 28、关于图片拉伸
>推荐看这个博客，讲的很详细[http://blog.csdn.net/q199109106q/article/details/8615661](http://blog.csdn.net/q199109106q/article/details/8615661)


#### 29、利用runtime获取一个类所有属性
```objectivec
- (NSArray *)allPropertyNames:(Class)aClass
{
    unsigned count;
    objc_property_t *properties = class_copyPropertyList(aClass, &count);

    NSMutableArray *rv = [NSMutableArray array];

    unsigned i;
    for (i = 0; i < count; i++)
    {
        objc_property_t property = properties[i];
        NSString *name = [NSString stringWithUTF8String:property_getName(property)];
        [rv addObject:name];
    }

    free(properties);

    return rv;
}
```

#### 30、设置textView的某段文字变成其他颜色
```objectivec
- (void)setupTextView:(UITextView *)textView text:(NSString *)text color:(UIColor *)color {
    NSMutableAttributedString *string = [[NSMutableAttributedString alloc]initWithString:textView.text];
    [string addAttribute:NSForegroundColorAttributeName value:color range:[textView.text rangeOfString:text]];
    [textView setAttributedText:string];
}
```

#### 31、让push跳转动画像modal跳转动画那样效果(从下往上推上来)
```objectivec
- (void)push
{
TestViewController *vc = [[TestViewController alloc] init];
    vc.view.backgroundColor = [UIColor redColor];
    CATransition* transition = [CATransition animation];
    transition.duration = 0.4f;
    transition.type = kCATransitionMoveIn;
    transition.subtype = kCATransitionFromTop;
    [self.navigationController.view.layer addAnimation:transition forKey:kCATransition];
    [self.navigationController pushViewController:vc animated:NO];
}

- (void)pop
{
CATransition* transition = [CATransition animation];
    transition.duration = 0.4f;
    transition.type = kCATransitionReveal;
    transition.subtype = kCATransitionFromBottom;
    [self.navigationController.view.layer addAnimation:transition forKey:kCATransition];
    [self.navigationController popViewControllerAnimated:NO];
}
```

#### 32、上传图片太大，压缩图片
```objectivec
-(UIImage *)resizeImage:(UIImage *)image
{
    float actualHeight = image.size.height;
    float actualWidth = image.size.width;
    float maxHeight = 300.0;
    float maxWidth = 400.0;
    float imgRatio = actualWidth/actualHeight;
    float maxRatio = maxWidth/maxHeight;
    float compressionQuality = 0.5;//50 percent compression

    if (actualHeight > maxHeight || actualWidth > maxWidth)
    {
        if(imgRatio < maxRatio)
        {
            //adjust width according to maxHeight
            imgRatio = maxHeight / actualHeight;
            actualWidth = imgRatio * actualWidth;
            actualHeight = maxHeight;
        }
        else if(imgRatio > maxRatio)
        {
            //adjust height according to maxWidth
            imgRatio = maxWidth / actualWidth;
            actualHeight = imgRatio * actualHeight;
            actualWidth = maxWidth;
        }
        else
        {
            actualHeight = maxHeight;
            actualWidth = maxWidth;
        }
    }

    CGRect rect = CGRectMake(0.0, 0.0, actualWidth, actualHeight);
    UIGraphicsBeginImageContext(rect.size);
    [image drawInRect:rect];
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    NSData *imageData = UIImageJPEGRepresentation(img, compressionQuality);
    UIGraphicsEndImageContext();

    return [UIImage imageWithData:imageData];

}
```

