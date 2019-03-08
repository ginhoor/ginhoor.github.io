# UIScrollView(UITableView、UICollectionView)Autolayout横屏frame自动变动

描述下问题出现的场景
【1】 ViewControllerA 是竖屏的VC，VC中有一个CollectionView。ViewControllerB是横屏的VC。
【2】跳转方式为 present

通过- (void)scrollViewDidScroll:(UIScrollView *)scrollView查看到，当VCA跳转到VCB的时候，collection view的frame、contentSize、contentOffset都发生了变化

跳转前
content size--->414.000000,2406.000000

跳转到VCB后
content size--->736.000000,1898.000000

***
可以得出的结论，这次变化是由横屏引起的。

通过调查，我发现引起这个的原因是Autolayout的方式设置View的约束在屏幕变化的时候会引起Frame的变化，即使你当前的VC是只支持竖屏。

于是我首先用手动管理frame代替了Autolayout

self.collectionView.frame = CGRectMake(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT - 40);

然后由于跳转VCB的时候又操作了status bar的隐藏，导致系统也会自动计算frame。

于是在VC中去掉了自动调整的部分

self.automaticallyAdjustsScrollViewInsets = NO;

最终frame变成：
self.collectionView.frame = CGRectMake(0, 64, SCREEN_WIDTH, SCREEN_HEIGHT - 40 - 64);
***
不过到此为止还是会有一点小问题，当VCA跳转VCB，status bar 被收起，页面还没完全跳转到VCB的时候，collection view 与 navigation bar 的中间会有20px的空白哦，暂时还没有解决，如果有同学解决了希望留言。

参考资料：
[Autolayout is changing UIScrollView's contentOffset on rotation](https://stackoverflow.com/questions/31844493/autolayout-is-changing-uiscrollviews-contentoffset-on-rotation)