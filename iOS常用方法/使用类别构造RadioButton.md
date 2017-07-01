在iOS中如果要使用类似安卓中的RadioButton，就会发现比较麻烦，因为在iOS中没有自带的RadioButton的官方控件，只能自己实现或者使用第三方库。

先上一张图：
![Simulator Screen Shot 2016年8月21日 下午9.36.28.png](http://upload-images.jianshu.io/upload_images/2312315-360cf22a95651202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就拿上图来举例，上图用到了两个RadioButton了，在每一类里只允许有一个选项，如果用户点击了新的选项，那么以前的选项就失效了，并且高亮选择的是最新的选项。

由于功能比较简单，我选择了自己实现RadioButton,虽然cocoapods上有很多已经造好了的轮子，但是我认为像这种很简单的控件能自己写的尽量还是不要用第三放的为好，闲话不多说了。

RadioButton的实现思路是做一个遍历，每当同一组的RadioButton的状态发生改变之后对所有的button进行一个遍历，把最新点击的button的selected属性更新为YES，把其他的button的selected属性更新为NO。

````
self.timeBtnArr = [[NSMutableArray alloc] initWithCapacity:0];
for (int i = 1; i<=4; i++) {
        UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        button.tag = baseTag + i;
        button.titleLabel.font = [UIFont systemFontOfSize:9];
        [button setTitleColor:[UIColor colorFromHexRGB:@"595959"] forState:UIControlStateNormal];
        [button setTitleColor:[UIColor colorFromHexRGB:@"ed7e2d"] forState:UIControlStateSelected];
        button.layer.cornerRadius = 3;
        button.layer.borderWidth = 0.5;
        button.layer.borderColor = [UIColor colorFromHexRGB:@"595959"].CGColor;
        button.layer.masksToBounds = YES;
        
        [button addTarget:self action:@selector(btnDidClicked:) forControlEvents:UIControlEventTouchUpInside];
        
        [self.view addSubview:button];
        
        switch (i) {
            case 1:
                [button setTitle:@"不限" forState:UIControlStateNormal];
                button.additionInfo = @"";
                break;
            case 2:
                [button setTitle:@"十五天内" forState:UIControlStateNormal];
                button.additionInfo = @"15";
                break;
            case 3:
                [button setTitle:@"十天内" forState:UIControlStateNormal];
                button.additionInfo = @"10";
                break;
            case 4:
                [button setTitle:@"五天内" forState:UIControlStateNormal];
                button.additionInfo = @"5";
                break;
            default:
                break;
        }
        [self.timeBtnArr addObject:button];
}
````
用for循环添加了一个具有四个button的RadioButton，以后在_timeBtnArr中对RadioButton的状态进行操作。

看一下触发按钮状态的代码：
````
- (void)btnDidClicked:(UIButton *) btn{
        for (UIButton *button in self.timeBtnArr) {
            if (button.tag == btn.tag) {
                button.layer.borderColor = [UIColor colorFromHexRGB:@"ed7e2d"].CGColor;
                button.selected = YES;
            }
            else{
                button.layer.borderColor = [UIColor colorFromHexRGB:@"595959"].CGColor;
                button.selected = NO;
            }
        }
}
````
每当RadioButton的状态改变时，都会从新更新一下RadioButton中button的状态。

但是，上面这么做跟业务的要求还是有所偏差，因为需要将RadioButton的选择状态保存下来。虽然可以直接将选中状态的title保存下来，但是这样显然会增加数据存储量，而且也不够优雅！所以，我选择将每个button增加一个附加属性，用来设置button所代表的含义。于是，需要为UIButton类增加一个类别，来添加一个additionInfo的属性。

````
//
//  UIButton+HFExtend.h
//  xStore
//
//  Created by apple on 16/8/10.
//  Copyright © 2016年 apple. All rights reserved.
//

#import <UIKit/UIKit.h>

@interface UIButton (HFExtend)

@property(copy, nonatomic) NSString *additionInfo;

@end
````

````
//
//  UIButton+HFExtend.m
//  xStore
//
//  Created by apple on 16/8/10.
//  Copyright © 2016年 apple. All rights reserved.
//

#import "UIButton+HFExtend.h"

#import "objc/runtime.h"

static void *additionInfoKey = &additionInfoKey;

@implementation UIButton (HFExtend)

- (void)setAdditionInfo:(NSString *)additionInfo{
    objc_setAssociatedObject(self, &additionInfoKey, additionInfo, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *) additionInfo{
 
    return objc_getAssociatedObject(self, &additionInfoKey);
}

@end
````

这样以后就能通过Button的additionInfo的属性来存储附加信息了。以后保存RadioButton的状态直接保存选择button的additionInfo属性即可。