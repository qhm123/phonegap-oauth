phonegap-oauth
==============
oauth2验证基本是一样的，但是貌似国内各个开放平台提供又有些差别，这里针对新浪微博oauth2验证方式来说明一些phonegap做oauth验证的三种方式。实例代码采用PhoneGap Android平台演示。

第一种

使用sina weibo提供的android客户端的library，这是个最简单的方式，但是又是个不灵活，稍重的方式。这种方式是参照PhoneGap的Facebook Connect Plugin实现的。实现方式是在项目中先引用http://open.weibo.com/wiki/SDK#Android_SDK，然后就是调用这个SDK中封装好的OAuth认证方法。比如先要init，然后login，login的代码示意如下：

Weibo weibo = Weibo.getInstance();
weibo.authorize((Activity)ConnectPlugin.this.cordova.getActivity(), 
    new AuthDialogListener(ConnectPlugin.this));
然后在AuthDialogListener的onComplete回调中会得到access_token和expires_in。非常简单。

之前说它稍重，是因为sina weibo android sdk中搞了一堆没用的东西。不灵活是这样只能是新浪微博，改用别的比较麻烦。

第二种

采用childBrowser插件。这个也挺简单的，childBrowser是一个显示网页的单独的activity级别覆层，其实和第一种有些类似，不过是要自己写访问的oauth验证接口的逻辑。oauth验证都有个回掉的url，然后在这个url中带着access_token等参数，然后获取这个参数得到。新浪微博的oauth接口中有个可以根据appkey，appsecret等参数直接得到access_token的方法，文档地址，接口地址：

https://api.weibo.com/oauth2/authorize?client_id=YOUR_CLIENT_ID&response_type=token&redirect_uri=YOUR_REGISTERED_REDIRECT_URI

成功后回调YOUR_REGISTERED_REDIRECT_URI/#access_token=ACCESS_TOKEN&expires_in=3600，得到access_token。

这里的问题是phonegap是本地页面，这个回调地址没法回到你的本地页面，一个方法是使用childBrowser的onLocationChange方法，监听浏览器location改变事件，然后做拦截，这里就是对回调地址做个约定，比如说都叫http://wodeweibo.com，然后这里判断一下，如果满足，则取到access_token后，再切换都别的页面。

第三种

应该是urlchange，而且可以是本地文件，授权回调地址可以是file:///android_asset/www/index.html#oauth，这个样在跳转到这个页面的时候取一下access_token就行，或者随便一个页面，然后取一下token，在跳转到自己的页面就行了。这个方法的缺点是兼容性，ios的本地页面的前面部分肯定不是这样。则个方法还需要更改一下phonegap的配置，在config.xml中把access那里改成*，不然跳转到微博授权页面会新开浏览器。