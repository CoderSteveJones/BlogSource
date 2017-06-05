---
title: NSPredicate 使用小结
date: 2017-06-05 22:37:04
tags:
---
> NSPredicate是一个Foundation类，它指定数据被获取或者过滤的方式。它的查询语言就像SQL的WHERE和正则表达式的交叉一样，提供了具有表现力的，自然语言界面来定义一个集合被搜寻的逻辑条件。

## 一、NSPredicate 使用
#### （1）集合中使用NSPredicate

NSArray&NSSet，不可变的集合，有可以通过评估接收到的predicate来返回一个不可变集合的方法filteredArrayUsingPredicate:和filteredSetUsingPredicate:。

NSMutableArray&NSMutableSet，可变集合，可以使用方法filterUsingPredicate:，它可以通过运行接收到的谓词来移除评估结果为FALSE的对象。

#### （2）配合正则表达式使用

```objectivec
-(BOOL)checkPhoneNumber:(NSString *)phoneNumber{
    NSString *regex = @"^[1][3-8]\\d{9}$";
    NSPredicate *pred = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex];
    return [pred evaluateWithObject:phoneNumber];
}
```

#### （3）Core Data中使用NSPredicate

NSFetchRequest有一个predicate属性，它可以指定管理对象应该被获取的逻辑条件。谓词的使用规则在这里同样适用，唯一的区别在于，在管理对象环境中，谓词由持久化存储助理（persistent store coordinator）评估，而不像集合那样在内存中被过滤。

self.studentArray 添加200个数据，供筛选，以下是生成self.studentArray的代码

```objectivec
// 学生对象
@interface Student :NSObject
/** 名字 */
@property (nonatomic, strong) NSString *name;
/** 班级 */
@property (nonatomic, assign) NSUInteger class;
/** 分数 */
@property (nonatomic, assign) NSUInteger score;

- (instancetype)initWithName:(NSString *)name Class:(NSInteger)class Score:(NSInteger)score;
@end
@implementation Student

- (instancetype)initWithName:(NSString *)name
                       Class:(NSInteger)class
                       Score:(NSInteger)score{
    if (self = [super init]) {
        if (class > 6 || class < 1) class = 1;
        if (score > 100 || score < 0) score = 0;
        _name = name;
        _class = class;
        _score = score;
    }
    return self;
}

@end
```

```objectivec
@interface ViewController ()
@property (nonatomic, strong) NSMutableArray <Student *> *studentArray;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
}
#pragma mark - - lazy load

- (NSMutableArray<Student *> *)studentArray{
    if (!_studentArray) {
        _studentArray = [NSMutableArray arrayWithCapacity:0];
        for (int i = 0; i < 200; i ++) {
            NSString *name = [self randomStringWithLength:5];
            NSInteger score = arc4random() % 100;
            NSInteger class = arc4random() % 6;

            Student *student = [[Student alloc] initWithName:name
                                                       Class:class
                                                       Score:score];
            [_studentArray addObject:student];
        }
    }
    return _studentArray;
}


/** 获取随机字符串 */
- (NSString *)randomStringWithLength:(NSInteger)length{
    char data[length];
    for (int x=0;x<length;data[x++] = (char)('a' + (arc4random_uniform(26))));
    return [[NSString alloc] initWithBytes:data length:length encoding:NSUTF8StringEncoding];
}

@end
```
## 二、谓词语法

#### （1）替换

> - %@ 是对值为字符串，数字或者日期的对象的替换值。
- %K 是key path的替换值。
- $VARIABLE_NAME 是可以被NSPredicate -predicateWithSubstitutionVariables:替换的值。

```objectivec
  //（%K,%d =\== 使用）
    // 分数为满分
    NSPredicate *predicateA = [NSPredicate predicateWithFormat:@"%K = %d",@"score",100];
    NSArray *caseA = [self.studentArray filteredArrayUsingPredicate:predicateA];

    //（self、BEGINSWITH、[]、$ 使用）
    // 名字以 ‘a’开头
    NSPredicate *predicateB = [NSPredicate predicateWithFormat:@"self.name BEGINSWITH[az] $beginningChar"];
    NSArray *caseB = [self.studentArray filteredArrayUsingPredicate:[predicateB predicateWithSubstitutionVariables:@{@"beginningChar": @"a"}]];
```

#### （2）比较运算符

> =, == ：左边的表达式和右边的表达式相等。
>=, => ：左边的表达式大于或者等于右边的表达式。
<=, =< ：左边的表达式小于等于右边的表达式。
> ：左边的表达式大于右边的表达式。
< ：左边的表达式小于右边的表达式。
!=, <> ：左边的表达式不等于右边的表达式。
BETWEEN ：左边的表达式等于右边的表达式的值或者介于它们之间。右边是一个有两个指定上限和下限的数值的数列（指定顺序的数列）。比如，1 BETWEEN { 0 , 33 }，或者$INPUT BETWEEN { $LOWER, $UPPER }。

```objectivec
// 比较运算符 =, ==  >=, =>  <=, =< > < !=, <> BETWEEN 使用
    // 90~100 优等学生
    NSPredicate *predicateC = [NSPredicate predicateWithFormat:@"self.score BETWEEN {90,100}"];
    NSArray *caseC = [self.studentArray filteredArrayUsingPredicate:predicateC];
```

#### （5）集合运算符

> ANY，SOME：指定下列表达式中的任意元素。比如，ANY children.age < 18。
ALL：指定下列表达式中的所有元素。比如，ALL children.age < 18。
NONE：指定下列表达式中没有的元素。比如，NONE children.age < 18。它在逻辑上等于NOT (ANY ...)。
IN：等于SQL的IN操作，左边的表达必须出现在右边指定的集合中。比如，name IN { 'Ben', 'Melissa', 'Nick' }。


#### （6）直接量

>FALSE、NO：代表逻辑假
TRUE、YES：代表逻辑真
NULL、NIL：代表空值
SELF：代表正在被判断的对象自身
"string"或'string'：代表字符串
数组：和c中的写法相同，如：{'one', 'two', 'three'}。
数值：包括证书、小数和科学计数法表示的形式
十六进制数：0x开头的数字
八进制：0o开头的数字
二进制：0b开头的数字

#### （7）数组操作

> array[index] ：指定数组中特定索引处的元素。
array[FIRST] ：指定数组中的第一个元素。
array[LAST] ：指定数组中的最后一个元素。
array[SIZE] ：指定数组的大小。

####（8）布尔值谓词
>TRUEPREDICATE ：结果始终为真的谓词。
FALSEPREDICATE ：结果始终为假的谓词。

####（9）Block谓词

```objectivec
NSPredicate *shortNamePredicate = [NSPredicate predicateWithBlock:^BOOL(id evaluatedObject, NSDictionary *bindings) {
            return [[evaluatedObject firstName] length] <= 5;
        }];
NSLog(@"Short Names: %@", [people filteredArrayUsingPredicate:shortNamePredicate]);
```


