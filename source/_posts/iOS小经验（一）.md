---
title: iOS小经验（一）
date: 2016-02-10 09:53:06
categories: 
	- 小经验
---
#### 1、禁止手机睡眠
```objectivec
[UIApplication sharedApplication].idleTimerDisabled = YES;
```

#### 2、隐藏某行cell
```objectivec
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
// 如果是你需要隐藏的那一行，返回高度为0
    if(indexPath.row == YouWantToHideRow)
        return 0; 
    return 44;
}

// 然后再你需要隐藏cell的时候调用
[self.tableView beginUpdates];
[self.tableView endUpdates];
```

#### 3、禁用button高亮
```objectivec
button.adjustsImageWhenHighlighted = NO;
或者在创建的时候
 UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
```

#### 4、tableview遇到这种报错failed to obtain a cell from its dataSource
> 是因为你的cell被调用的早了。先循环使用了cell，后又创建cell。顺序错了
> 可能原因：1、xib的cell没有注册 2、内存中已经有这个cell的缓存了(也就是说通过你的cellId找到的cell并不是你
> 想要的类型)，这时候需要改下cell的标识

#### 5、cocoa pods报这个错误：unable to access '[https://github.com/facebook/pop.git/](https://github.com/facebook/pop.git/)': Operation timed out after 0 milliseconds with 0 out of 0 bytes received
> 解决办法：原因可能是网络问题，网络请求超时了，只需要重试就行了

#### 6、cocoa pods 出现ERROR: While executing gem ... (Errno::EPERM)
> [解决办法：https://segmentfault.com/q/1010000002926243](https://segmentfault.com/q/1010000002926243)

#### 7、动画切换window的根控制器
```objectivec
// options是动画选项
[UIView transitionWithView:[UIApplication sharedApplication].keyWindow duration:0.5f options:UIViewAnimationOptionTransitionCrossDissolve animations:^{
        BOOL oldState = [UIView areAnimationsEnabled];
        [UIView setAnimationsEnabled:NO];
        [UIApplication sharedApplication].keyWindow.rootViewController = [RootViewController new];
        [UIView setAnimationsEnabled:oldState];
    } completion:^(BOOL finished) {

    }];
```

#### 8、去除数组中重复的对象
```objectivec
NSArray *newArr = [oldArr valueForKeyPath:@“@distinctUnionOfObjects.self"];
```

#### 9、编译的时候遇到 no such file or directory: ／users／apple／XXX
> 是因为编译的时候，在此路径下找不到这个文件，解决这个问题，首先是是要检查缺少的文件是不是在工程中，如果不在工程中，需要从本地拖进去，如果发现已经存在工程中了，或者拖进去还是报错，这时候需要去build phases中搜索这个文件，这时候很可能会搜出现两个相同的文件，这时候，有一个路径是正确的，删除另外一个即可。如果删除了还是不行，需要把两个都删掉，然后重新往工程里拖进这个文件即可

#### 10、iOS8系统中，tableView最好实现下-tableView: heightForRowAtIndexPath:这个代理方法，要不然在iOS8中可能就会出现显示不全或者无法响应事件的问题

#### 11、iOS8中实现侧滑功能的时候这个方法必须实现，要不然在iOS8中无法侧滑
```objectivec
// 必须写的方法，和editActionsForRowAtIndexPath配对使用，里面什么不写也行
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath {

}
```

#### 12、三个通知
>NSSystemTimeZoneDidChangeNotification监听修改时间界面的两个按钮状态变化
UIApplicationSignificantTimeChangeNotification 监听用户改变时间 （只要点击自动设置按钮就会调用） NSSystemClockDidChangeNotification 监听用户修改时间（时间不同才会调用）


#### 13、SDWebImage本地缓存有时候会害人。如果之前缓存过一张图片，即使下次服务器换了这张图片，但是图片url没换，用sdwebimage下载下来的还是以前那张,所以遇到这种问题，不要先去怼服务器，清空下缓存再试就好了。

#### 14、上线前注意：
> 1）、删掉代码中所有的测试代码
2）、如果后台有审核模式，提醒后台开启此模式
3）、主流程再跑一跑
4）、全局搜索waring，检查所有标记waring的地方

#### 15、跳进app权限设置

```objectivec
// 跳进app设置
            if (UIApplicationOpenSettingsURLString != NULL) {
                UIApplication *application = [UIApplication sharedApplication];
                NSURL *URL = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
                if ([application respondsToSelector:@selector(openURL:options:completionHandler:)]) {
                    [application openURL:URL options:@{}
                       completionHandler:nil];
                } else {
                    [application openURL:URL];
                }
            }
```

#### 16、给一个view截图
```objectivec
UIGraphicsBeginImageContextWithOptions(view.bounds.size, YES, 0.0);
    [view.layer renderInContext:UIGraphicsGetCurrentContext()];
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
```

#### 17、开发中如果要动态修改tableView的tableHeaderView或者tableFooterView的高度，需要给tableView重新设置，而不是直接更改高度。正确的做法是重新设置一下tableView.tableFooterView = 更改过高度的view。为什么？其实在iOS8以上直接改高度是没有问题的，在iOS8中出现了contentSize不准确的问题，这是解决办法。

#### 18、注意对象为nil的时候，调用此对象分类的方法不会执行

#### 19、collectionView的内容小于其宽高的时候是不能滚动的，设置可以滚动：
```objectivec
collectionView.alwaysBounceHorizontal = YES;
collectionView.alwaysBounceVertical = YES;
```

#### 20、设置navigationBar上的title颜色和大小
```objectivec
    [self.navigationController.navigationBar setTitleTextAttributes:@{NSForegroundColorAttributeName : [UIColor youColor], NSFontAttributeName : [UIFont systemFontOfSize:15]}]
```

#### 21、颜色转图片
```objectivec
+ (UIImage *)cl_imageWithColor:(UIColor *)color {
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
#### 22、view设置圆角

```objectivec
#define ViewBorderRadius(View, Radius, Width, Color)\
\
[View.layer setCornerRadius:(Radius)];\
[View.layer setMasksToBounds:YES];\
[View.layer setBorderWidth:(Width)];\
[View.layer setBorderColor:[Color CGColor]] // view圆角
```

#### 23、强／弱引用
```objectivec
#define WeakSelf(type)  __weak typeof(type) weak##type = type; // weak
#define StrongSelf(type)  __strong typeof(type) type = weak##type; // strong
```

24、由角度转换弧度
```objectivec
#define DegreesToRadian(x) (M_PI * (x) / 180.0)
```

25、由弧度转换角度
```objectivec
#define RadianToDegrees(radian) (radian*180.0)/(M_PI)
```

26、获取图片资源
```objectivec
#define GetImage(imageName) [UIImage imageNamed:[NSString stringWithFormat:@"%@",imageName]]
```

27、获取temp
```objectivec
#define PathTemp NSTemporaryDirectory()
```

28、获取沙盒 Document
```objectivec
#define PathDocument [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
```

29、获取沙盒 Cache
```objectivec
#define PathCache [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject]
```

30、GCD代码只执行一次
```objectivec
#define kDISPATCH_ONCE_BLOCK(onceBlock) static dispatch_once_t onceToken; dispatch_once(&onceToken, onceBlock);
```

