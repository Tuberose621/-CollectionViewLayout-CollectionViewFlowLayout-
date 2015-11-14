-  Github地址：**[-CollectionViewLayout-CollectionViewFlowLayout-](https://github.com/Tuberose621/-CollectionViewLayout-CollectionViewFlowLayout-)**
-  **这里详解了三个demo去帮助大家更好的了解CollectionViewLayout和CollectionViewFlowLayout**
-----------------------------------
-  **自定义流水布局--CollectionViewFlowLayout---水平布局实现一个相册功能**


![](http://upload-images.jianshu.io/upload_images/739863-3bf78b2651535bb6.gif?imageMogr2/auto-orient/strip)
**一**
- 在UIScrollView的基础上进行循环利用
    +  那怎么去做循环利用呢？
-  第一种方案：
    +  实时监控ScrollView的滚动，一旦有一个家伙离开屏幕，我们就把它放进一个数组或者是集合里面去，到时候我要用，我就把它拿过去用
    +  但是这个是很麻烦的，因为你总是得判断它有没有离开屏幕
-  第二种方案：
    +  用苹果自带的几个类：TableView或者是CollectionView
    +  因为它们本来就具备循环利用的功能
    +  但是TableView一看就不符合要求，因为它默认就是上下竖直滚动，不是左右水平滚动
        +  当然我们也可以用非主流的方式，让TableView实现水平滚动
        +  让TableView的Transform来个90°，让它里面所有的cell也翻个90°，都转过来。但这种做法有点奇葩，开发中还是不要这么搞
        +  所以我们可以用CollectionView
    +  CollectionView在我们的印象中是展示像那种九宫格的样子，而且也是上下竖直滚动
    +  但是CollectionView和TableView的区别就是：
        +  CollectionView它默认就支持水平滚动，你只要修改它一个属性为水平方向就行了。而TableView默认支持竖直滚动，没有属性去支持它水平滚动，除非你去搞一些非主流的做法

![](http://upload-images.jianshu.io/upload_images/739863-6ac5441780f9e28f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-------------------------------------------------
**二**
- CollectionView一定要传一个不空的Layout那个参数，因为默认的布局是九宫格，它按这种方式排的原因是它有一个流水布局。正因为给它传了一个流水布局，所以它就一行满了，就流向下一行，流水一样流下去流过来 

```objc
    UICollectionView *collectionView = [[UICollectionView alloc] initWithFrame:frame collectionViewLayout:[UICollectionViewlayout alloc] init]];
``` 

- **数据源方法 - <UICollectionViewDataSource>**
- numberOfItemsInSection是告诉它一组有多少个格子

```objc
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return 50 ;
}
```
- cellForItemAtIndexPath告诉它每个格子长出来是怎样的一个cell，因为每个格子都是一个CollectionViewCell

```objc
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    // 先要注册
    UICollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:CYCellId forIndexPath:indexPath];
    cell.backgroundColor = [UIColor orangeColor]

    return cell;
}
```

![](http://upload-images.jianshu.io/upload_images/739863-c52259516018ece4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  TableView和CollectionView的排布有很大的区别
    +  TableView的排布是一行一行往下排布，而CollectionView的排布是完全取决于Layout，也就是说，你传给它的Layout不一样，它的排布就不一样。它的布局决定了cell的排布
    +  也就是说，今后你想要CollectionView的cell排布丰富多彩，你只需要改变它的布局就行了
- scrollDirection决定了它的滚动方向，设置它滚动的方向为水平

```objc
    // 水平滚动
    self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
```

- itemSize决定了CollectionView布局的里面的cell的大小

```objc
    layout.itemSize = CGSizeMake(100, 100);
```

- 你将CollectionView高度改小点，比如200，那么你的高度不够显示两排，就会如下显示：

![](http://upload-images.jianshu.io/upload_images/739863-846ee2f6d4618c4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 而且你会发现不用担心循环利用的问题，CollectionView内部已经帮你做好了

![](http://upload-images.jianshu.io/upload_images/739863-dd0be31ff3fb50de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

--------------------------------------
**三**

-  我们现在已经实现流水布局水平滚动，而且做好了循环利用。如果要做一层改进，那么我们就要自定义布局，自己来写一套布局，所以现在我们继承于UICollectionViewFlowLayout  
-  我们要自定义CollectionView的布局有两种方案
    +  1.继承UICollectionViewLayout
        +  一般是继承于UICollectionViewLayout就行了
        +  而且UICollectionViewFlowLayout继承于UICollectionViewLayout
        +  但是如果你自定义继承于UICollectionViewLayout，代表着你没有流水布局功能，也就是在你不想要流水布局功能的时候就选择继承UICollectionViewLayout
    +  2.继承UICollectionViewFlowLayout  
----------------------------------------
**四 **
- 所以我们自定义流水布局CYLineLayout
-  在CYLineLayout.h文件中

```objc
#import <UIKit/UIKit.h>

@interface CYLineLayout : UICollectionViewFlowLayout

@end
```
----------------------------------------------
-  在CYLineLayout.m文件中重写某些方法去实现：
    +  1.cell的放大与缩小
    +  2.停止滚动的时候：cell居中
-  进入头文件可以发现要重写的一些方法


![](http://upload-images.jianshu.io/upload_images/739863-6c5e9486b0bc9cdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
----------------------------------------------

- UICollectionViewLayoutAttributes 
    +  1.它是描述布局属性的
    +  2.一个cell对应一个UICollectionViewLayoutAttributes对象
    +  3.UICollectionViewLayoutAttributes对象决定了cell的展示样式（frame）说白了就是决定你的cell摆在哪里，怎么去摆  


![](http://upload-images.jianshu.io/upload_images/739863-302f9a4c012973e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
----------------------------------------------
----------------------------------------------
-  layoutAttributesForElementsInRect这个方法的返回值是一个数组（数组里面存放着rect范围内所有元素的布局属性）
-  这个方法的返回值决定了rect范围内所有元素的排布（frame）

```objc
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    // 获得super已经计算好的布局属性（在super已经算好的基础上，再去做一些改进）
    NSArray *array = [super layoutAttributesForElementsInRect:rect];

    // 计算collectionView最中心点的x值
    CGFloat centerX = self.collectionView.contentOffset.x + self.collectionView.frame.size.width * 0.5;

    // 在原有布局属性的基础上进行微调
    for (UICollectionViewLayoutAttributes *attrs in array) {
        // cell的中心点x和collectionView最中心点的x值 的间距
        CGFloat delta = ABS(attrs.center.x - centerX);

        // 根据间距值计算cell的缩放比例
        CGFloat scale = 1 - delta / self.collectionView.frame.size.width;

        // 设置缩放比例
        attrs.transform = CGAffineTransformMakeScale(scale, scale);
    }

    return array;
}
```
----------------------------------------------

![ ](http://upload-images.jianshu.io/upload_images/739863-0fb007b71b8883a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-  计算collectionView中心点的x值
    +  要记住collectionView的坐标原点是以内容contentSize的原点为原点
    +  计算collectionView中心点的x值，千万不要用collectionView的宽度除以2。而是用collectionView的偏移量加上collectionView宽度的一半
    +  坐标原点弄错了就没有可比性了，因为后面要判断cell的中心点与collectionView中心点的差值

```objc
CGFloat centerX = self.collectionView.contentOffset.x + self.collectionView.frame.size.width * 0.5;
```
----------------------------------------------

-  cell的中心点x 和CollectionView最中心点的x值 的间距

```objc
     CGFloat delta = ABS(attrs.center.x - centerX);
```

```objc
     ABS(A)
     // 表示取绝对值 
```
----------------------------------------------

-  我们再根据间距值delta去算cell的缩放比例scale
    +  间距值delta和缩放比例scale是成反比的
    +  间距值delta的范围为0--self.collectionView.frame.size.width * 0.5

```objc
CGFloat scale = 1 - delta / self.collectionView.frame.size.width;
// 用1-（），是因为间距值delta和缩放比例scale是成反比的
```
- 设置缩放比例

```objc
attrs.transform = CGAffineTransformMakeScale(scale, scale);
```
----------------------------------------------
- 但是设置后你会发现基本没啥反应，显示还乱七八糟的，这是什么原因呢？
    +  我们是想要稍微动一下就修改一下，但是现在没法达到我动一下就根据最新的中心点X来再算一遍一边比例。没有实现这个代码
    +  因为这里还需要实现一个方法
-  这个方法是shouldInvalidateLayoutForBoundsChange: 它的特点是：
    +  默认return NO
    +  当collectionView的显示范围发生改变的时候，判断是否需要重新刷新布局
    +  一旦重新刷新布局，就会重新调用下面的方法：
        + 1.prepareLayout
        + 2.layoutAttributesForElementsInRect:方法

```objc
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds
{
    return YES;
  
}
```
-  这样之后，你会发现，你稍微挪一下，它就重新算一遍，比例就会缩放， 达到了我们的要求
-  而且非常流畅，因为它有循环利用
----------------------------------------------

![](http://upload-images.jianshu.io/upload_images/739863-2ca70f472258ba2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-----------------------------------------
**五**
-  还要实现一个方法：targetContentOffsetForProposedContentOffset:()方法。它的返回值，就决定了collectionView停止滚动时的偏移量
-  这个方法在你手离开屏幕之前会调用，也就是cell即将停止滚动的时候 （记住这一点）

```objc
- (CGPoint)targetContentOffsetForProposedContentOffset:(CGPoint)proposedContentOffset withScrollingVelocity:(CGPoint)velocity
{
    // 计算出最终显示的矩形框
    CGRect rect;
    rect.origin.y = 0;
    rect.origin.x = proposedContentOffset.x;
    rect.size = self.collectionView.frame.size;

    // 获得super已经计算好的布局属性
    NSArray *array = [super layoutAttributesForElementsInRect:rect];

    // 计算collectionView最中心点的x值
    CGFloat centerX = proposedContentOffset.x + self.collectionView.frame.size.width * 0.5;

    // 存放最小的间距值
    CGFloat minDelta = MAXFLOAT;
    for (UICollectionViewLayoutAttributes *attrs in array) {
        if (ABS(minDelta) > ABS(attrs.center.x - centerX)) {
            minDelta = attrs.center.x - centerX;
        }
    }

    // 修改原有的偏移量
    proposedContentOffset.x += minDelta;
    return proposedContentOffset;
}
```
-----------------------------------------

-  获得super已经计算好的布局属性

```objc
    NSArray *array = [super layoutAttributesForElementsInRect:rect];
```
-  这里为什么不用self
    +  因为如果调self，又会来到layoutAttributesForElementsInRect:()方法的for循环中， 将transform再算一遍。而我们只想要拿到中心点X值。靠父类就行了
    +  我们调super这个方法，因为它当时已经算好了cell的中心点等X的值了。所以这里调super可能更好一点
-----------------------------------------

-  计算collectionView最中心点的x值
```objc
 CGFloat centerX = proposedContentOffset.x + self.collectionView.frame.size.width * 0.5;
```
    - 这里为什么不按前面
```objc
CGFloat centerX = self.collectionView.contentOffset.x + self.collectionView.frame.size.width * 0.5;
```
来算呢？
   +  因为targetContentOffsetForProposedContentOffset:()方法在你手离开屏幕之前会调用，也就是cell即将停止滚动的时候，这个时候我们要算的是最后停下来偏移量。
    +  假如我们用力往左边一甩，你的手已经离开，算的偏移量是你手离开时候的偏移量，而不是我们最终的偏移量，也就是说这么算的话，我们就算错了
    +  你是应该拿到最终停下来的cell和CollectionView的中心点的X值进行比较的。所以你应该最终的值，而不是手松开的那一刻的偏移量的值
        +  那我们怎么知道手松开的那一刻最终的偏移量X的值呢？
            +  这个方法返回的参数(CGPoint)proposedContentOffset，这是它本应该停留的位置，最终停留的的值。而(CGPoint)targetContentOffsetForProposedContentOffset:这个是你最终返回的值，也就是你要它停留到哪儿的值（这个参数决定你要cell最后停留在哪儿）
-----------------------------------------
- 同上面可知，我们最后拿到的矩形框也是不能乱传的，也是要拿到最终的哪一个矩形框（不明白，就想像一下，你往左边或者右边用手指一甩的时候，手离开的时候是一个值，最终停下来是一个值，而现在我们需要的是最终的值）

```objc
    // 计算出最终显示的矩形框 
    CGRect rect; rect.origin.y = 0; rect.origin.x = proposedContentOffset.x; rect.size = self.collectionView.frame.size;
```
-----------------------------------------

![ ](http://upload-images.jianshu.io/upload_images/739863-5b274362c53ac789.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-----------------------------------------
-----------------------------------------

-  然后我们要找最短的偏移量，找到它，然后就让他偏移它的那个值，让它的中心点回到collectionView的中心点，也就是说重合。这样就实现了不管你怎么去甩，等cell停下来的时候。都会有一个cell它会停留在矩形框CollectionView的中心

```objc
    // 存放最小的间距值
         CGFloat minDelta = MAXFLOAT; 
         for (UICollectionViewLayoutAttributes *attrs in array) {
               if (ABS(minDelta) > ABS(attrs.center.x - centerX)) {                     
              minDelta = attrs.center.x - centerX; 
              }
       } 
        // 修改原有的偏移量 
        proposedContentOffset.x += minDelta; 
        return proposedContentOffset;
```
-  一开始先保证minDelta是最大的，保证谁都比你小。 第一次算出来的绝对值就肯定比你小，然后把它赋值给你minDelta。这样就算出来了最小的间距值
-  算出来最小间距值后，你通过分析应该会发现，不管是往左偏还是往右偏，要想让cell回到中心点，最后你的偏移量应该是用：你本来应该 的偏移量+（cell的中心点X值—collectionView中心点X值）
-  所以上面在比较的时候用绝对值，计算的时候不用绝对值，minDelta最后就有正数也有负数
-  修改后让它回到中间
-  最后不管你怎么滑，它都会停在中间
-----------------------------------------------
**六**
- 有一个小缺陷，你会发现，一打开程序，你往左或往右滑到最左或者最右的时候，cell总是默认粘着边上，这个不太和谐，我们需要它距离左右两边都有一个距离，那我们该怎么做呢？
    +  这就是让我们把所有的cell，让它们往右边或者左边挪一段距离，所以就增加内边距就可以了。怎么添加内边距呢？
        +  collectionView是继承ScrollView的，所以设置它的ContentInset就可以了
        +  还一种方法通过这个布局它本来就有一个属性sectionInset ，这本来就是来控制内边距的，控制整个布局的。而且这个属性只需要设置一次
-  这里有一个给collectionView专门用来布局的方法---prepareLayout，这里一般是做初始化操作

```objc
/**
 * 用来做布局的初始化操作（不建议在init方法中进行布局的初始化操作--可能布局还未加到View中去，就会返回为空）
 */
- (void)prepareLayout
{
    [super prepareLayout];

    // 设置内边距
    CGFloat inset = (self.collectionView.frame.size.width - self.itemSize.width) * 0.5;
    self.sectionInset = UIEdgeInsetsMake(0, inset, 0, inset);
}

```
-----------------------------------------------------
**七**
- 总的来说我们若要继承自这个流水布局来实现这个功能的话，肯定是要重写一些方法，告诉它一些内部的行为，它才知道怎么去显示那个东西，我们用了一下的方法：
    +  我们首先得实现prepareLayout方法，做一些初始化
    +  然后，我们实现layoutAttributesForElementsInRect:方法。目的是拿出它计算好的布局属性来做一个微调，这样可以导致我们的cell可以变大或者变小
    +  然后实现targetContentOffsetForProposedContentOffset:方法。它的目的是告诉当我手松开，cell停止滚动的时候，他应该去哪儿，所以这个方法就决定了collectionView停止滚动时的偏移量
    +  最后shouldInvalidateLayoutForBoundsChange:这个方法的价值就是告诉它你只要稍微往左或者往右挪一下，你就重新刷新，只要你重新刷新，它就会重新根据你cell的中心点的X值距离你collectionView中心点的X值来决定你的缩放比例。这样就保证了我们每动一点点，比例都在变，所以我们要动一下刷新一下。也就是当collectionView的显示范围发生改变的时候，是否需要重新刷新布局，一旦重新刷新布局，就会重新调用下面的方法：1.prepareLayout2.layoutAttributesForElementsInRect:方法
- 关于做这个效果有一个挺牛逼的三方框架：**[iCarousel](https://github.com/nicklockwood/iCarousel)**大家可以参考一下

--------------------------------------------
**八**
- 在CYLineLayout.h文件中

```objc
#import <UIKit/UIKit.h>

@interface CYLineLayout : UICollectionViewFlowLayout

@end
```
- 在CYLineLayout.h文件中

```objc
#import "CYLineLayout.h"

@implementation CYLineLayout

- (instancetype)init
{
    if (self = [super init]) {
    }
    return self;
}

/**
 * 当collectionView的显示范围发生改变的时候，是否需要重新刷新布局
 * 一旦重新刷新布局，就会重新调用下面的方法：
 1.prepareLayout
 2.layoutAttributesForElementsInRect:方法
 */
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds
{
    return YES;
}

/**
 * 用来做布局的初始化操作（不建议在init方法中进行布局的初始化操作）
 */
- (void)prepareLayout
{
    [super prepareLayout];

    // 水平滚动 
    self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
    // 设置内边距
    CGFloat inset = (self.collectionView.frame.size.width - self.itemSize.width) * 0.5;
    self.sectionInset = UIEdgeInsetsMake(0, inset, 0, inset);
}

/**
 UICollectionViewLayoutAttributes *attrs;
 1.一个cell对应一个UICollectionViewLayoutAttributes对象
 2.UICollectionViewLayoutAttributes对象决定了cell的frame
 */
/**
 * 这个方法的返回值是一个数组（数组里面存放着rect范围内所有元素的布局属性）
 * 这个方法的返回值决定了rect范围内所有元素的排布（frame）
 */
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    // 获得super已经计算好的布局属性
    NSArray *array = [super layoutAttributesForElementsInRect:rect] ;

    // 计算collectionView最中心点的x值
    CGFloat centerX = self.collectionView.contentOffset.x + self.collectionView.frame.size.width * 0.5;

    // 在原有布局属性的基础上，进行微调
    for (UICollectionViewLayoutAttributes *attrs in array) {
        // cell的中心点x 和 collectionView最中心点的x值 的间距
        CGFloat delta = ABS(attrs.center.x - centerX);

        // 根据间距值 计算 cell的缩放比例
        CGFloat scale = 1 - delta / self.collectionView.frame.size.width;

        // 设置缩放比例
        attrs.transform = CGAffineTransformMakeScale(scale, scale);
    }
        return array;
}

/**
 * 这个方法的返回值，就决定了collectionView停止滚动时的偏移量

 */
- (CGPoint)targetContentOffsetForProposedContentOffset:(CGPoint)proposedContentOffset withScrollingVelocity:(CGPoint)velocity
{
    // 计算出最终显示的矩形框
    CGRect rect;
    rect.origin.y = 0;
    rect.origin.x = proposedContentOffset.x;
    rect.size = self.collectionView.frame.size;

    // 获得super已经计算好的布局属性
    NSArray *array = [super layoutAttributesForElementsInRect:rect];

    // 计算collectionView最中心点的x值
    CGFloat centerX = proposedContentOffset.x + self.collectionView.frame.size.width * 0.5;

    // 存放最小的间距值
    CGFloat minDelta = MAXFLOAT;
    for (UICollectionViewLayoutAttributes *attrs in array) {
        if (ABS(minDelta) > ABS(attrs.center.x - centerX)) {
            minDelta = attrs.center.x - centerX;
        }
    }

    // 修改原有的偏移量
    proposedContentOffset.x += minDelta;
    return proposedContentOffset;
}

@end
```

--------------------------------------------
**九**
- 假如我们要监听cell的点击，要怎么办呢？上面这讲的这些都和CollectionViewCell的点击没有关系，只是和布局有关。监听CollectionViewCell的点击和CollectionViewCell的布局没有任何关系，布局只负责展示，格子里面是什么内容，还是取决于cell
-  布局的作用仅仅是控制cell的排布
    +  控制器先成为CollectionViewCell的代理：UICollectionViewDelegate 
-  现在要把数据填充上去，让它显示相册了，所以自定义CollectionViewCell--CYPhotoCell,由于里面是固定死的，所以加一个Xib文件，里面加一个ImageView,拖线给一个属性，给ImageView一个标识photo
-  给cell里面的相片加上一个相册相框的效果--两种方案：
      +  第一种方案：在Xib的ImageView的布局上下左右都给一个10的间距，给一个white的背景颜色 
      +  第二种方案：给我们的ImageView加一个图层就可以了

```objc
- (void)awakeFromNib {
       self.imageView.layer.borderColor = [UIColor whiteColor].CGColor; 
       self.imageView.layer.borderWidth = 10;
 }
```

-  在CYPhotoCell.h文件中

```objc
#import <UIKit/UIKit.h>

@interface CYPhotoCell : UICollectionViewCell
/** 图片名 */
@property (nonatomic, copy) NSString *imageName;
@end
```
-  在CYPhotoCell.m文件中

```objc
#import "CYPhotoCell.h"

@interface CYPhotoCell()

@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@end

@implementation CYPhotoCell

- (void)awakeFromNib {
    self.imageView.layer.borderColor = [UIColor whiteColor].CGColor;
    self.imageView.layer.borderWidth = 10;
}

- (void)setImageName:(NSString *)imageName
{
    _imageName = [imageName copy];

    self.imageView.image = [UIImage imageNamed:imageName];
}

@end
```
-  在ViewController.m文件中

```objc
#import "ViewController.h"
#import "CYLineLayout.h"
#import "CYPhotoCell.h"

@interface ViewController () <UICollectionViewDataSource, UICollectionViewDelegate>
@end

@implementation ViewController

static NSString * const CYPhotoId = @"photo";

- (void)viewDidLoad {
    [super viewDidLoad];

    // 创建布局
    CYLineLayout *layout = [[CYLineLayout alloc] init];
    layout.itemSize = CGSizeMake(100, 100);

    // 创建CollectionView
    CGFloat collectionW = self.view.frame.size.width;
    CGFloat collectionH = 200;
    CGRect frame = CGRectMake(0, 150, collectionW, collectionH);
    UICollectionView *collectionView = [[UICollectionView alloc] initWithFrame:frame collectionViewLayout:layout];
    collectionView.dataSource = self;
    collectionView.delegate = self;
    [self.view addSubview:collectionView];

    // 注册
    [collectionView registerNib:[UINib nibWithNibName:NSStringFromClass([CYPhotoCell class]) bundle:nil] forCellWithReuseIdentifier:CYPhotoId];
}

#pragma mark - <UICollectionViewDataSource>
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return 20;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    CYPhotoCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:CYPhotoId forIndexPath:indexPath];

    cell.imageName = [NSString stringWithFormat:@"%zd", indexPath.item + 1];

    return cell;
}

#pragma mark - <UICollectionViewDelegate>
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath
{
    NSLog(@"------%zd", indexPath.item);
}
@end
```
- 最后就实现了：
![](http://upload-images.jianshu.io/upload_images/739863-3bf78b2651535bb6.gif?imageMogr2/auto-orient/strip)
---------------------------------------------------
**十**
####自定义流水布局
-  **自定义布局 - 继承UICollectionViewFlowLayout**  
-  **重写prepareLayout方法**
    - 作用：
          -  在这个方法中做一些初始化操作
    - 注意：
          -  一定要调用[super prepareLayout]

-  **重写layoutAttributesForElementsInRect:方法**
    - 作用：
          -  这个方法的返回值是个数组
          - 这个数组中存放的都是UICollectionViewLayoutAttributes对象
          - UICollectionViewLayoutAttributes对象决定了cell的排布方式（frame等）

-  **重写shouldInvalidateLayoutForBoundsChange:方法**
    - 作用：
          -  如果返回YES，那么collectionView显示的范围发生改变时，就会重新刷新布局
    - 一旦重新刷新布局，就会按顺序调用下面的方法：
          - prepareLayout
          - layoutAttributesForElementsInRect:

-  **重写targetContentOffsetForProposedContentOffset:方法**
    - 作用：
          -  返回值决定了collectionView停止滚动时最终的偏移量（contentOffset）
    - 参数：
          - proposedContentOffset：原本情况下，collectionView停止滚动时最终的偏移量
          - velocity：滚动速率，通过这个参数可以了解滚动的方向（根据X和Y的正负）




--------------------------------------------------

--------------------------------------------------

--------------------------------------------------

--------------------------------------------------

--------------------------------------------------
**自定义布局--CollectionViewLayout--格子布局**

![](http://upload-images.jianshu.io/upload_images/739863-a5e35123291b6ec4.gif?imageMogr2/auto-orient/strip)

-  分析一下这个布局的排布是有规律的：
    +  这里的相册布局和上面的流水布局不同
    +  我们较上面的不需要更改太多东西，只是修改它的布局方式就行了
    +  六个为一组
    +  对应cell相差两个高度
![](http://upload-images.jianshu.io/upload_images/739863-ba059d34d6cb2fb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-  一个这样的布局如何实现？
    +  首先这里不不好用流水布局，流水布局的ItemSize是一样大的
    +  肯定也牵扯到了循环利用，所以仍然用CollectionView，所以就用一个最根的布局--CollectionViewLayout
    +  CollectionViewLayout它不像流水布局，内部没有任何方法给你去排，所以你只有继承自它，然后自己去写一套排布方式，排布是由我们来算
    +  将上面文件中的CYLineLayout删除，New一个File--CYGridLayout继承自CollectionViewLayout
----------------------------------------
```objc
    // 创建UICollectionViewLayoutAttributes
    NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
    UICollectionViewLayoutAttributes *attrs = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
```
-  说白了我这个UICollectionViewLayoutAttributes是描述一个cell用的
-  indexPath代表了对应某个位置的cell，也就是说我这个UICollectionViewLayoutAttributes是描述哪个位置的cell
-  通过观察可以发现规律


![ ](http://upload-images.jianshu.io/upload_images/739863-1d889bc44e86c7ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
----------------------------------------
----------------------------------------
-  在ViewController.m文件中修改一下collectionView的frame和布局

```objc
#import "ViewController.h"
#import "CYGridLayout.h"
#import "CYPhotoCell.h"

@interface ViewController () <UICollectionViewDataSource, UICollectionViewDelegate>
@end

@implementation ViewController

static NSString * const CYPhotoId = @"photo";

- (void)viewDidLoad {
    [super viewDidLoad];

    // 创建布局
    CYGridLayout *layout = [[CYGridLayout alloc] init];

    // 创建CollectionView
    UICollectionView *collectionView = [[UICollectionView alloc] initWithFrame:self.view.bounds collectionViewLayout:layout];
    collectionView.dataSource = self;
    collectionView.delegate = self;
    [self.view addSubview:collectionView];

    // 注册
    [collectionView registerNib:[UINib nibWithNibName:NSStringFromClass([CYPhotoCell class]) bundle:nil] forCellWithReuseIdentifier:CYPhotoId];
}
```
-  CYGridLayout里面去实现collectionView具体的布局
-  在CYGridLayout.m文件中

```objc
#import "CYGridLayout.h"

@interface CYGridLayout()
/** 所有的布局属性 */
@property (nonatomic, strong) NSMutableArray *attrsArray;
@end

@implementation CYGridLayout

- (NSMutableArray *)attrsArray
{
    if (!_attrsArray) {
        _attrsArray = [NSMutableArray array];
    }
    return _attrsArray;
}

- (void)prepareLayout
{
    [super prepareLayout];

    [self.attrsArray removeAllObjects];

    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    for (int i = 0; i < count; i++) {
        // 创建UICollectionViewLayoutAttributes
        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        UICollectionViewLayoutAttributes *attrs = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];

        // 设置布局属性
        CGFloat width = self.collectionView.frame.size.width * 0.5;
        if (i == 0) {
            CGFloat height = width;
            CGFloat x = 0;
            CGFloat y = 0;
            attrs.frame = CGRectMake(x, y, width, height);
        } else if (i == 1) {
            CGFloat height = width * 0.5;
            CGFloat x = width;
            CGFloat y = 0;
            attrs.frame = CGRectMake(x, y, width, height);
        } else if (i == 2) {
            CGFloat height = width * 0.5;
            CGFloat x = width;
            CGFloat y = height;
            attrs.frame = CGRectMake(x, y, width, height);
        } else if (i == 3) {
            CGFloat height = width * 0.5;
            CGFloat x = 0;
            CGFloat y = width;
            attrs.frame = CGRectMake(x, y, width, height);
        } else if (i == 4) {
            CGFloat height = width * 0.5;
            CGFloat x = 0;
            CGFloat y = width + height;
            attrs.frame = CGRectMake(x, y, width, height);
        } else if (i == 5) {
            CGFloat height = width;
            CGFloat x = width;
            CGFloat y = width;
            attrs.frame = CGRectMake(x, y, width, height);
        } else {
            UICollectionViewLayoutAttributes *lastAttrs = self.attrsArray[i - 6];
            CGRect lastFrame = lastAttrs.frame;
            lastFrame.origin.y += 2 * width;
            attrs.frame = lastFrame;
        }

        // 添加UICollectionViewLayoutAttributes
        [self.attrsArray addObject:attrs];
    }
}
```

-----------------
-  运行程序：

![](http://upload-images.jianshu.io/upload_images/739863-932a895d2ae33ec4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  你会发现无法使它往上滚动，这是为啥呢？
    +  因为你现在时继承自最根本的布局CollectionViewLayout，很多东西是得自己去设置了才会有，来到头文件，你会发现
    +  要重写它的(CGSize)collectionViewContentSize方法，告诉它你这个CollectionView的内容尺寸，来决定它怎么滚。所以你现在无法滚动是因为CollectionView的ContentSize没有确定

```objc
/**
 * 返回collectionView的内容大小
 */
- (CGSize)collectionViewContentSize
{
    int count = (int)[self.collectionView numberOfItemsInSection:0];
    int rows = (count + 3 - 1) / 3;
    CGFloat rowH = self.collectionView.frame.size.width * 0.5;
    return CGSizeMake(0, rows * rowH);
}

- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    return self.attrsArray;
}
```
------------------------
-  这里在性能优化上是还有点小问题的，因为我们一口气把所有东西都算完了。你如果觉得费时，完全可以把计算放在子线程中，然后返回到主线程刷新UI（CollectionViewLayout布局中有一个刷新方法，你调一下就行了）
-  计算不是重点，你是可以总结出计算的规律的。重点是：继承自CollectionViewLayout你需要注意什么？
    +  **1.一旦你重写了layoutAttributesForElementsInRect这个方法，就意味着所有东西你得自己写了，你的Attributes对象得自己创建了，因为它的父类不会帮你创建**
    +  **2.一旦你继承自CollectionViewLayout，意味着你这个collectionViewContentSize都得告诉它了，这个是得你自己去算的**
    +  **3.如果你是希望一口气把所有东西算完，不希望它在滚动过程中再算，你可以在prepareLayout方法里面先算清楚，算完后尽管它传的矩形框都不一样，但是我返回的还是同一份。**


- **这里给出一个思想：**
    +  **以后，你凡事牵扯到内容是很多很多的，你想做什么循环利用，而且布局又乱七八糟的，我们用CollectionViewLayout就可以了。我们只有继承自这个CollectionViewLayout，然后我们实现layoutAttributesForElementsInRect这个方法，在那里去告诉它，你的cell怎么去排。并且继承自CollectionViewLayout，意味着很多东西都要重写，如：collectionViewContentSize**
-  这样就实现了：
![](http://upload-images.jianshu.io/upload_images/739863-a5e35123291b6ec4.gif?imageMogr2/auto-orient/strip)

--------------------------------------


--------------------------------------


--------------------------------------


--------------------------------------


--------------------------------------
**自定义布局--CollectionViewLayout--布局之间的切换**

![](http://upload-images.jianshu.io/upload_images/739863-965346be4f6c1d56.gif?imageMogr2/auto-orient/strip)
-  要求：
    +  实现一个环形布局和水平布局的相册，点击屏幕能够进行不同布局之间的切换
    +  点击cell的时候可以删除cell
-  首先通过分析，在上面第一个案例的基础上，再添加一个环形布局--CYCircleLayout，肯定也是只能继承自CollectionViewLayout
-  在这里CYCircleLayout里面就只需要实现prepareLayout方法和layoutAttributesForElementsInRect方法，不需再要重写实现collectionViewContentSize的方法，因为它不需要滚动，所以CollectionViewLayout里面所有方法的实现是看你的需求的
 
```objc
#import "CYCircleLayout.h"

@interface CYCircleLayout()
/** 布局属性 */
@property (nonatomic, strong) NSMutableArray *attrsArray;
@end

@implementation CYCircleLayout

- (NSMutableArray *)attrsArray
{
    if (!_attrsArray) {
        _attrsArray = [NSMutableArray array];
    }
    return _attrsArray;
}

- (void)prepareLayout
{
    [super prepareLayout];

    [self.attrsArray removeAllObjects];

    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    for (int i = 0; i < count; i++) {
        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        UICollectionViewLayoutAttributes *attrs = [self layoutAttributesForItemAtIndexPath:indexPath];
        [self.attrsArray addObject:attrs];
    }
}

- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    return self.attrsArray;
}
```
----------------------------------------------

![](http://upload-images.jianshu.io/upload_images/739863-03bf65edb38964fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  我们可以看出，每个相片cell的中心点都在一个圆上，所以我们要将它摆正，肯定不是设置它们的frame，而是去设置它center这个值，我只要保证它的center那个值在那个圆上就可以了
-  也就是说我们要算出每个相片cell的中心点的X和Y值，通过中心点来布局它，而不是通过frame的original的X和Y（这样太麻烦，不好算）
-  这里我们只要确定圆心就好算了
    +  圆心（X和Y值分别是CollectionView宽度和高度的一半）
    +  而且每张相片的中心点距离圆心的距离为半径
    +  你会发现每个相片cell的中心点的X，Y和圆心的X，Y之间的差值是有规律的：
          +  Y值--圆心点的Y值-(Y*cosa)= cell的Y值,X值同样道理去算
          +  角度a的大小取决于cell的个数（假如20个cell--->a = 360° / 20）



![](http://upload-images.jianshu.io/upload_images/739863-e27af3d8272db62c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
--------------------------------------

------------------------- 
![](http://upload-images.jianshu.io/upload_images/739863-7f3f9c861e7e3521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  所以我们只要算出平分角度就行了
-  比如说第一个cell为索引0，角度就是0，第二个为索引1，角度就是a, 第三个为索引2，角度就是a*2......第i个为索引i-1，角度就是a*(i-1 )
-  于是乎

![](http://upload-images.jianshu.io/upload_images/739863-5d33beae2ee43467.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----------------------------------------

-  这里记住：如果你是继承自CollectionViewLayout，如果你要换布局话，有一个方法是一定得实现的--layoutAttributesForItemAtIndexPath:方法。只有继承CollectionViewLayout才需要，流水布局不需要，因为流水布局内部早已经帮你实现了这个方法

```objc
/**
 * 这个方法需要返回indexPath位置对应cell的布局属性
 */
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath
{
    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    CGFloat radius = 70;
    // 圆心的位置
    CGFloat oX = self.collectionView.frame.size.width * 0.5;
    CGFloat oY = self.collectionView.frame.size.height * 0.5;

    UICollectionViewLayoutAttributes *attrs = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];

    attrs.size = CGSizeMake(50, 50);
    if (count == 1) {
        attrs.center = CGPointMake(oX, oY);
    } else {
        CGFloat angle = (2 * M_PI / count) * indexPath.item;
        CGFloat centerX = oX + radius * sin(angle);
        CGFloat centerY = oY + radius * cos(angle);
        attrs.center = CGPointMake(centerX, centerY);
    }

    return attrs;
}
```


-  点击屏幕切换布局

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    if ([self.collectionView.collectionViewLayout isKindOfClass:[CYLineLayout class]]) {
        [self.collectionView setCollectionViewLayout:[[CYCircleLayout alloc] init] animated:YES];
    } else {
        CYLineLayout *layout = [[CYLineLayout alloc] init];
        layout.itemSize = CGSizeMake(100, 100);
        [self.collectionView setCollectionViewLayout:layout animated:YES];
    }
}
```

-  点击cell就把cell删掉
    +  这里要注意的是：
         +  你要把cell删掉了，对应的模型或者说 数据也是得改变的
-  可变数组，先把所有图片名放进去

```objc
@interface ViewController () <UICollectionViewDataSource, UICollectionViewDelegate>
/** collectionView */
@property (nonatomic, weak) UICollectionView *collectionView;
/** 数据 */
@property (nonatomic, strong) NSMutableArray *imageNames;
@end

@implementation ViewController

static NSString * const CYPhotoId = @"photo";

- (NSMutableArray *)imageNames
{
    if (!_imageNames) {
        _imageNames = [NSMutableArray array];
        for (int i = 0; i<20; i++) {
            [_imageNames addObject:[NSString stringWithFormat:@"%zd", i + 1]];
        }
    }
    return _imageNames;
}

```
-  数据源里面的东西也是得改变的

```objc
#pragma mark - <UICollectionViewDataSource>
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return self.imageNames.count;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    CYPhotoCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:CYPhotoId forIndexPath:indexPath];

    cell.imageName = self.imageNames[indexPath.item];

    return cell;
}
```

-  你要把cell删掉，也得保证把模型也删掉了（不可能你cell删掉了，数据还是这么多，那就出问题了）

```objc
#pragma mark - <UICollectionViewDelegate>
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath
{
    [self.imageNames removeObjectAtIndex:indexPath.item];
    [self.collectionView deleteItemsAtIndexPaths:@[indexPath]];
}
```

-  删除到最后一个的时候，让最后一个cell的位置来到圆心

```objc
    if (count == 1) {
        attrs.center = CGPointMake(oX, oY);
    } else {
        CGFloat angle = (2 * M_PI / count) * indexPath.item;
        CGFloat centerX = oX + radius * sin(angle);
        CGFloat centerY = oY + radius * cos(angle);
        attrs.center = CGPointMake(centerX, centerY);
    }

```

--------------------------------------------

-  这样所有的逻辑就理清楚了

--------------------------------------------
-  在CYCircleLayout.m文件中

```objc
#import "CYCircleLayout.h"

@interface CYCircleLayout()
/** 布局属性 */
@property (nonatomic, strong) NSMutableArray *attrsArray;
@end

@implementation CYCircleLayout

- (NSMutableArray *)attrsArray
{
    if (!_attrsArray) {
        _attrsArray = [NSMutableArray array];
    }
    return _attrsArray;
}

- (void)prepareLayout
{
    [super prepareLayout];

    [self.attrsArray removeAllObjects];

    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    for (int i = 0; i < count; i++) {
        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        UICollectionViewLayoutAttributes *attrs = [self layoutAttributesForItemAtIndexPath:indexPath];
        [self.attrsArray addObject:attrs];
    }
}

- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    return self.attrsArray;
}

/**
 * 这个方法需要返回indexPath位置对应cell的布局属性
 */
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath
{
    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    CGFloat radius = 70;
    // 圆心的位置
    CGFloat oX = self.collectionView.frame.size.width * 0.5;
    CGFloat oY = self.collectionView.frame.size.height * 0.5;

    UICollectionViewLayoutAttributes *attrs = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];

    attrs.size = CGSizeMake(50, 50);
    if (count == 1) {
        attrs.center = CGPointMake(oX, oY);
    } else {
        CGFloat angle = (2 * M_PI / count) * indexPath.item;
        CGFloat centerX = oX + radius * sin(angle);
        CGFloat centerY = oY + radius * cos(angle);
        attrs.center = CGPointMake(centerX, centerY);
    }

    return attrs;
}
@end
```
-  在ViewController.m文件中

```objc
#import "ViewController.h"
#import "CYLineLayout.h"
#import "CYCircleLayout.h"
#import "CYPhotoCell.h"

@interface ViewController () <UICollectionViewDataSource, UICollectionViewDelegate>
/** collectionView */
@property (nonatomic, weak) UICollectionView *collectionView;
/** 数据 */
@property (nonatomic, strong) NSMutableArray *imageNames;
@end

@implementation ViewController

static NSString * const CYPhotoId = @"photo";

- (NSMutableArray *)imageNames
{
    if (!_imageNames) {
        _imageNames = [NSMutableArray array];
        for (int i = 0; i<20; i++) {
            [_imageNames addObject:[NSString stringWithFormat:@"%zd", i + 1]];
        }
    }
    return _imageNames;
}

- (void)viewDidLoad {
    [super viewDidLoad];

    // 创建布局
    CYCircleLayout *layout = [[CYCircleLayout alloc] init];

    // 创建CollectionView
    CGFloat collectionW = self.view.frame.size.width;
    CGFloat collectionH = 200;
    CGRect frame = CGRectMake(0, 150, collectionW, collectionH);
    UICollectionView *collectionView = [[UICollectionView alloc] initWithFrame:frame collectionViewLayout:layout];
    collectionView.dataSource = self;
    collectionView.delegate = self;
    [self.view addSubview:collectionView];
    self.collectionView = collectionView;

    // 注册
    [collectionView registerNib:[UINib nibWithNibName:NSStringFromClass([CYPhotoCell class]) bundle:nil] forCellWithReuseIdentifier:CYPhotoId];

    // 继承UICollectionViewLayout
    // 继承UICollectionViewFlowLayout
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    if ([self.collectionView.collectionViewLayout isKindOfClass:[XMGLineLayout class]]) {
        [self.collectionView setCollectionViewLayout:[[XMGCircleLayout alloc] init] animated:YES];
    } else {
        XMGLineLayout *layout = [[XMGLineLayout alloc] init];
        layout.itemSize = CGSizeMake(100, 100);
        [self.collectionView setCollectionViewLayout:layout animated:YES];
    }
}


#pragma mark - <UICollectionViewDataSource>
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return self.imageNames.count;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    CYPhotoCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:CYPhotoId forIndexPath:indexPath];

    cell.imageName = self.imageNames[indexPath.item];

    return cell;
}

#pragma mark - <UICollectionViewDelegate>
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath
{
    [self.imageNames removeObjectAtIndex:indexPath.item];
    [self.collectionView deleteItemsAtIndexPaths:@[indexPath]];
}
@end

```
----------------------------------------
-  这样就实现了
![](http://upload-images.jianshu.io/upload_images/739863-965346be4f6c1d56.gif?imageMogr2/auto-orient/strip)
