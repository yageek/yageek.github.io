---

title: "Scrollable custom view inside a UIWebView"
date: "2015-02-18"
comments: true
published: true
tags: 
    - ios 
    - uiwebview
    - scrollview
---

Displaying HTML content is very easy on iOS. Just grab a `UIWebView` and call `loadHTMLString:baseURL` or  `loadData:MIMEType:textEncodingName:baseURL:` and enjoy the results.
This component would fit your needs 90% of the time. But in some cases, having a simple `UIWebView` is not enough.

For professional reasons, I had to implement a mail reader like screen. On such a screen, there is a fixed portion on the screen that scrolls along with the portion rendering the HTML. I did not build such a view before and start thinking how I could make it.

After thinking a little bit, I first thought about implementing the fixed portion in HTML/CSS. No offense to those technologies, but they are not really my cup of tea.

I start searching on the Internet to see if someone founds a nice way implementing this without using private APIS.

# Tweaking the internal `UIWebBrowserView`

Each `UIWebView` contains a `UIScrolView`. Inside this scroll view, the HTML is rendered by a Apple internal private `UIView` subclass named `UIWebBrowserView`. It is possible to get a reference to this subview.

To have a reference to it, simply loop over the subviews contained within the `UIScrollView` of the `UIWebView`:

```objective-c
for(UIView *subview in self.webView.scrollView.subviews)
        {
            NSString * className = NSStringFromClass([subview class]);
            if([className isEqualToString:@"UIWebBrowserView"])
            {
            	//YES ! Just grab a reference to it :)
                self.HTMLRender  = subview;

                //Set the frame of the view to fit your needs
                break;
            }
        }
```

Once you grab it, you can use and place this view by setting its `frame`.

This works great, but it has the smell of a big workaround :( Would it be possible to do it in a easier way ?

# Do it thanks to the documentation

I read the article in the great [objc.io](http://www.objc.io/) concerning [scroll views](http://www.objc.io/issue-3/scroll-view.html) and the start of a solution appears by reading the part **Tweaking the Window with Content Insets**.

The main idea is to use the `contentInset` property of the `UIScrollView` of the `UIWebView` to create a space at the top of the `UIWebView` and fill it with some view.

It is better to see with an example. Let's create a `UIWebView` subclass named `AwesomeWebView`. This will be a web view with a 100 points red view at this top.

```objective-c
#define TOP_SPACE_HEIGHT 100.0f

@interface AwesomeWebView : UIWebView
@end
```

All the work could be done in the initializer. The only thing to consider, is that adding a contentInset create a zone in the view where the coordinates are negatives.

```objective-c
- (id) initWithFrame:(CGRect)frame
{
    if(self = [super initWithFrame:frame])
    {
        [self setupScrollView:TOP_SPACE_HEIGHT]; // Setting the contentInset
    }
    return self;
}

- (void) setupScrollView:(CGFloat)headerHeight
{
	// Set the contentInset to create space for the top View
    self.scrollView.contentInset = UIEdgeInsetsMake(headerHeight, 0, 0, 0);

    // Set the contentSize
    self.scrollView.contentSize = (CGSize){CGRectGetWidth(self.frame), CGRectGetHeight(self.frame) + headerHeight};

    //Make the scroll indicator insets starts at the bottom of the top view
    self.scrollView.scrollIndicatorInsets = UIEdgeInsetsMake(headerHeight, 0, 0, 0);

    //Initialize the redTopView
    //-> Keep in mind that the zone up to the scroll view is negative.
    UIView * redView = [[UIView alloc] initWithFrame:CGRectMake(0, -headerHeight, CGRectGetWidth(self.frame), headerHeight)];
    redView.backgroundColor = [UIColor redColor];

    //Finally add it to the view;
    [self.scrollView addSubview:redView];
}
```

This methods seems to be better than the first one.

You can find an example of this method on [github](https://github.com/yageek/CustomWebview)
