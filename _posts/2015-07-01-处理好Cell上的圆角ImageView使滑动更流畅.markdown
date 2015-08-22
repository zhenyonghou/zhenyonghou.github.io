---
layout: post
date:   2015-07-01 19:14
categories: OS
---

常常有将ImageView设置成圆角的需求，很常见的就是头像。通常用代码生成圆角：
{% highlight cpp %}
avatarView.layer.cornerRadius = 36 / 2;
avatarView.layer.borderWidth = 0.5;
avatarView.layer.borderColor = [UIColor lightGrayColor].CGColor;
avatarView.clipsToBounds = YES;
{% endhighlight %}

当每个Cell上有两个圆角ImageView的时候，滚动会明显卡顿。Cell虽然在复用，但是每次复用后都要再计算并绘制圆角。看来这绘制圆角运算量蛮大的。

###尝试通过另外一种方法来完成圆角需求

ImageView不要计算圆角，让设计师再给出张空心圆角图，圆角图除去空心外的其他区域颜色同Cell背景色，遮盖在ImageView的Image上。先管这一层叫ImageCover层。

测试，卡顿减轻许多。

这说明，Cell上尽量避免计算、绘制操作。

这时候bug来了，当按下Cell，Cell高亮时，ImageCover的颜色要保持与Cell背景色一致。

还得让设计师出张高亮图，就是把上面白色区域替换成高亮色，Cell默认按下色是（127,127,127）。

实现代码：
{% highlight cpp %}
@interface HGRoundCornerImageView : UIImageView

@property (nonatomic, strong, readonly) UIImageView *coverImageView;

- (void)setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholderImage;

@end

@interface HGRoundCornerImageView()

@property (nonatomic, strong, readwrite) UIImageView *coverImageView;

@end

@implementation HGRoundCornerImageView

- (id)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super initWithCoder:aDecoder]) {
        [self commonInit];
    }
    return self;
}

- (id)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        [self commonInit];
    }
   
    return self;
}

- (void)commonInit {
    self.contentMode = UIViewContentModeScaleAspectFit;
   
    _coverImageView = [[UIImageView alloc] initWithFrame:self.bounds];
    _coverImageView.contentMode = UIViewContentModeScaleAspectFit;
    [self addSubview:_coverImageView];
}

- (void)setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholderImage
{
    [self sd_setImageWithURL:url placeholderImage:placeholderImage];     // 这里使用到了SDWebImageView
}

- (void)layoutSubviews
{
    [super layoutSubviews];
    _coverImageView.frame = self.bounds;
}

@end

{% endhighlight %}


###需要注意的地方
Cell里：
{% highlight cpp %}
- (void)setAvatarImage:(UIImage *)image
{  
    [self.avatarView setImageWithURL:nil placeholderImage:image];

    ...

    self.avatarView.coverImageView.highlighted = (self.isHighlighted || self.isSelected);
}

- (void)setHighlighted:(BOOL)highlighted animated:(BOOL)animated {
    [super setHighlighted:highlighted animated:animated];
   
    self.avatarView.coverImageView.highlighted = (self.isHighlighted || self.isSelected);
}
{% endhighlight %}

Controller里：
{% highlight cpp %}
- (void)tableView:(nonnull UITableView *)tableView didSelectRowAtIndexPath:(nonnull NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:NO];     // 这里一定要为NO
}
{% endhighlight %}



作者 [侯振永][1]     
写于2015 年 7月 1日

[1]: https://zhenyonghou.github.io/