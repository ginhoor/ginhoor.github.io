## 为什么要组件化

随着业务的发展，App中的页面，网络请求，通用弹层UI，通用TableCell数量就会剧增，需求的开发人员数量也会逐渐增多。

如果所有业务都在同一个App中，并且同时开发人数较少时，抛开代码健壮性不谈，实际的开发体验可能并没有那么糟糕，毕竟作为一个开发，什么地方用什么控件，就跟在HashMap中通过Key获取Value那么简单。



那么当业务成长到需要分化到多个App的时候，组件化的重要性开始体现了。

##### 案例一

```objc
@interface CESettingsListCell : UITableViewCell

@property (strong, nonatomic) UILabel *titleLabel;
@property (strong, nonatomic) UITextField *textField;
@property (strong, nonatomic) UIImageView *arrowImgV;

@end
```

​	这是TableCell，其中有`标题`，`小图标`，`右箭头`，最初用于满足设置列表显示。





基类

BaseNavigationController，BaseViewController，BaseModel，BaseWebViewController等。





```objc
@implementation CEUserAuthViewController (CERouter)

+ (void)toUserAuth:(UINavigationController *)nav didAuthSuccessBlock:(void (^)(void))didAuthSuccessBlock
{
    CEUserAuthViewController *vc = [[CEUserAuthViewController alloc] init];
    vc.didAuthSuccessBlock = didAuthSuccessBlock;
    [nav pushViewController:vc animated:YES];
}

@end
```





目前是为每一个页面都做了Category，作为路由逻辑的封装。



学习了下网上流行的URLRouter，Protocol-Class和Target-Action方案，最终决定用Target-Action方案的思路。原因有这么几点：

- 上面有提到过，开始的时候我用Category封装了每一个页面的跳转逻辑，而页面的数量已经有了上百的规模，Protocal-Class方案会比Target-Action多出更多的代码量。
- Protocol-Class的方案还是存在Protocol与ViewController等组件的耦合，虽然不妨事，但是很碍眼。
- 路由方案在后期会考虑升级成路由表，在Target-Action的调度者中加入Url方案也比较容易，参数解析已经完成，不需要重复修改。





参考资料

[iOS 组件化之路由设计思路分析](https://cloud.tencent.com/developer/article/1029462)

[iOS开发——组件化及去Mode化方案](https://www.jianshu.com/p/9239669c3a96)