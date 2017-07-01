在开发iOS的SDK中，会遇到从宿主app跳转至SDK的情景。在SDK的开发中，开发者是按照自已的需求进行开发的，所以很容易出现对导航栏样式的改变。

而宿主app采用栈式导航将SDK压入到导航栈中，所有在栈中的视图控制器公用一个导航控制器，如果SDK在入栈时将导航栏视图控制器的外观改变，那么当出栈时如果没有恢复导航控制器的外观，那么宿主app的导航视图控制器的外观就会被改变，这是SDK开发者不想看到的，因为SDK本身的设计思想就是模块化开发，高度解耦。当宿主app调用SDK后，SDK调用完毕时改变了宿主app并不想改变的东西，就会造成耦合。

所以，需要SDK在调用完毕时恢复宿主app的导航视图控制器状态。看下面的一个例子：

````
//
//  SettingBaseVC.m
//  LanYunNews
//
//  Created by whf on 17/3/25.
//  Copyright © 2017年 apple. All rights reserved.
//

#import "SettingBaseVC.h"

@interface SettingBaseVC ()

@end

@implementation SettingBaseVC

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    [self setNavi];
}

- (void)dealloc {
    NSLog(@"%@---free",NSStringFromClass([self class]));
}

@end

````