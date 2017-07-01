````

````
一、定义block
     有返回值、有参数：返回类型 ^(blockName)(参数) ＝  ^返回类型（参数列表）｛///代码 ｝；
     无返回值、有参数：void ^(blockName)(参数) ＝ ^（参数列表）｛///代码 ｝；
     无返回值、无参数： void (^blockName)() = ^ { /// 代码实现; }； 
     上面这么多，也记不住：
     速记代码快：inlineBlock ，编译器会提示：（根据需要删减就好了）；
二、block引用外部变量
      在定义block时，如果使用了外部变量，block内部会默认对外部变量做一次copy；
      默认情况下，不允许在block内部修改外部变量的值；
      在外部变量声明时，使用__block修饰符，则可以在block内部修改外部变量的值；
三、数组的遍历&排序；
      遍历：enumerateObjectsUsingBlock:
                所有的参数都已经准备到位，可以直接使用
                效率比for高，官方推荐使用；
               举例：懒加载
               enumerateObjectsUsingBlock遍历：
               [tempArray enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL*_Nonnull stop) {
           NSDictionary *dict = (NSDictionary*)obj;            Heros *hero = [HerosherosWithDict:dict];
            [ArrMaddObject:hero]; 
        }];
        for—IN遍历： 
       for (NSDictionary*dict in tempArray) {            Heros *heros = [HerosherosWithDict:dict];            [ArrM addObject:heros];
        }

      排序：sortedArrayUsingComparator: