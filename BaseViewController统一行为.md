#统一行为
当你看到第一篇BaseViewController设计时候会觉得这种方式很不错，今天我们来探讨不一样的角度去优化这个设计。


##Example 1:
在构建一个app的时候，会有很多相同行为。例如， 任何需要网络请求的界面，总会需要一个indicator来告诉用户，“噢，我正在操作，请等我一下”。或者某个时候我们需要给用户一点小提示，来对他们的操作进行反馈等等。每一个ViewController都会有这样的行为。这是我们很直接地想到我们需要一个父类。

于是，我们创建了一个基类， BaseViewController继承自UIViewController：

<pre><code>@interface BaseViewController : UIViewController

/**

 *  a little pop window used to replace laertView

 *

 *  @param tips tips will show to user

 */

- (void)showTips:(NSString *)tips;

/**

 *  When some operation need user waiting,use this method to show a HDU.

 *

 *  @param title will show to user

 */

- (void)showIndicatorWithTitle:(NSString *)title;

- (void)hideIndicator;

@end
</code></pre>
好了， 所有的业务相关的ViewController都继承自BaseViewController,并且在调用网络请求前轻松地写一句[self showIndicatorWithTitle:@"加载中..."]，并且在网络请求完成后的回调里写下[self hideIndicator]就可以轻松地完成指示器的显示隐藏了.

##Example 2:
再举个例子。项目里访问服务器的时候需要Token，Token在某些情况下有可能过期。Token过期的时候会返回一个Token过期的状态码，此时需要用户重新登录。这个情况我们可以总结成：

1.绝大多数ViewController都会发送网络请求

2.每一个网络请求都会返回Token过期

3.每个ViewController都需要处理Token过期的情况

因此我们可以将这一行为抽取出来，写在BaseViewController中。这样只需要在各个业务模块的的Model层收到Token过期消息的时候，用Notification转发给BaseViewController就可以了。其他诸如网络请求失败等情况都是如此。

每个项目有不同的统一行为，我们需要根据不同的行为抽取不同的基类。

但是继承是不是就真这么优美呢？

##使用协议
例如LoginViewController需要和BaseViewController一样的显示/隐藏Indicator的接口，但却没有其他相同的行为。这时继承就显得不太合适了.

想象一下BaseViewController因为Token过期Present了LoginViewController,而LoginViewController因为继承了BaseViewController又不停地Present另一个LoginViewController...这时候如果使用继承的话，就不得不写一些脏代码来避免这样的情况。

此时我们可以用协议@protocol来替代继承。

<pre><code>
@protocol PSIndicator <NSObject> 

- (void)showTips:(NSString *)tips;

- (void)showIndicatorWithTitle:(NSString *)title;

- (void)hideIndicator; 

@end
</code></pre>

然后在LoginViewController中：

<pre><code>@interface LoginViewController : UIViewController<PSIndicator> 

@end
</code></pre>


