Title: FCPageControl custom lightweight UIPageControl replacement.
Date: 2014-07-20 10:20
Category: obj-c
Tags: objective-c, ui, components
Slug: fcpagecontrol-custom-uipagecontrol
Authors: jandro_es
Summary: FCPageControl is a lightweight replacement to UIPageControl which allows quite a few more customisations than the standard control.

UIPageControl is a great and simple to use control for *slider* or page based navigation, but it has very few appearance options. In some applications you need to customise it's appearance to match the look and feel of the application, or just match the designr required. For the latter i had to develop a custom control which let me change it's appearance deeply than the standard one lets.

It works exactly as a **UIPageControl** but with a few extra properties to modify it's visual appearance:

~~~~{.language-objectivec}
	@property (nonatomic, strong) UIColor *activeColor;
	@property (nonatomic, strong) UIColor *inactiveColor;
	@property (nonatomic) CGFloat dotDiameter;
	@property (nonatomic) CGFloat dotMargin;
~~~~

These two properties let us change the active and inactive color of the dots.

~~~~{.language-objectivec}
	@property (nonatomic, strong) UIColor *activeColor;
	@property (nonatomic, strong) UIColor *inactiveColor;
~~~~

And these two let us change the diameter of the dots and the margin between them.

~~~~{.language-objectivec}
	@property (nonatomic) CGFloat dotDiameter;
	@property (nonatomic) CGFloat dotMargin;
~~~~

There is also an optional delegate to notify the delegation object when a dot is touched:

~~~~{.language-objectivec}
	@protocol FCPageControlDelegate <NSObject>

	@optional
	- (void)pageControl:(FCPageControl *)pageControl Clicked:(NSUInteger)index;

	@end
~~~~

This delegate can easily be extended to allow even more interactions, when a slider is swiped, tapped, etc..

The component is already in **Cocoapods** to install it you just have to add to your **Podfile** the following line:

~~~~{.language-ruby}
	pod 'FCPageControl'
~~~~

The code is available in it's [Github repository](https://github.com/jandro-es/FCPageControl).