# weixin
微信各种坑
相关转载链接：
http://www.cnblogs.com/xueranzp/p/5287691.html
http://kf.qq.com/faq/140225MveaUz150413VNj6nm.html

坑的情况：

其实不少问题，腾讯都给出了答案，只是腾讯的文档不是很集中，一般看不到。整理下我看到的官方文档链接。

http://kf.qq.com/faq/140225MveaUz150413VNj6nm.html  【常见问题】

http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html  【JSSDK说明及常见问题】

https://mp.weixin.qq.com/wiki/6/01405db0092f76bb96b12a9f954cd866.html  【报错排查指引】

http://mp.weixin.qq.com/wiki/17/fa4e1434e57290788bde25603fa2fcbd.html  【全局返回码说明】

开发步骤：

一、申请公众号，并开通支付等接口(公司行政处理的，我只管拿来用)

二、获取appid、商户号、AppSecret等信息

三、自行设置API安全密钥

1、登录微信商户平台(http://pay.weixin.qq.com)，依次进入  账户设置-->API安全-->API证书-->下载证书  下载并保存证书到本地。

2、账户设置-->API安全-->设置API密钥  ，自己设置一个密钥，32位字符。自行保存（设置后就不可见了，只能修改）

四、本地调试准备工作（支付测试，需要外网域名，准备工作主要就是在本地设置外网可访问的域名）

方法1、ngrok 内网穿透工具，此工具可以方便的为本机设置一个外网域名，不过由于是国外的产品，所以可能被墙，参考 http://www.tuicool.com/articles/e63Ebm6

使用方法： 1)解压压缩包(如：d:/ngrok)  2)CMD切换到d:/ngrok目录， 运行命令“ngrok -config ngrok.cfg -subdomain 你想要的域名 服务端口”  3）浏览器输入 http://你想要的域名.ngrok.natapp.cn/  即可访问默认页面。

方法2、  微信官方推荐的方法，详情参考链接： http://blog.qqbrowser.cc/start/   http://blog.qqbrowser.cc/wei-xin-gong-zhong-hao-ben-di-diao-shi/

五、相关域名设置

1、支付目录设置，根据 https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_3 配置授权目录，需要特别注意的是，目录要配置到精确目录，而不是二级或者三级目录，并且区分大小写。目录指的是确认付款的页面目录而不是调用支付的接口目录。

比如商品支付页面链接为 http://www.XXX.com.cn/pay/payorder/a.html  需要配置为 www.XXX.com.cn/pay/payorder/

2、授权回调权限域名设置。 公众平台--》开发--》接口权限--》网页授权获取用户基本信息--》修改  输入域名即可（只需输入 www.XXX.com.cn）

3、 JS接口安全域名设置。 公众平台--》设置--》公众号设置--》功能设置--》JS接口安全域名。 同上，只需输入www.XXX.com.cn

主要坑及解决方案：

各种坑，有很多，个人搜集比较经典的参考链接如下：、

1） error: invalid signature

https://segmentfault.com/q/1010000002502269/a-1020000002549180

https://segmentfault.com/q/1010000002520634

摘录以做备份：

signature 的值是用多个参数 sha1 加密的结果，详细流程即：

1， 通过 appid + appsecert 获取公众号的 access_token（不是用户的 access_token）官方文档 http://mp.weixin.qq.com/wiki/15/54ce45d8d30b6bf6758f68d2e95bc627.html
2， 根据 1 的access_token 来获取 jsapi_ticket
3， 生成一个随机字符串 nonceStr（16）位
4， 生成一当前时间缀 timestamp
5， 获取当前网页 URL（#号后不要）

获取到以上 5 步之后，将 jsapi_ticket，nonceStr， timestamp，URL 组成 Query String（GET 参数），即：

$queryString = "jsapi_ticket=XXX&noncestr=XXX&timestamp=XXX&url=XXX";

生成 Query String 要注意：
1，Query String 的顺序不能变（按我给的示例）
2，Query String 中的 key 要全小写
3，Query String 中的 value 区分大小写
4，URL 要确保只获取 # 号之前部分（有 # 号的话）
5，Query String 要确保没有被 urlencode（如果使用 http_build_query 的话需要 urldecode 一次）

signature 的值就是 sha1 加密后的结果，即：

$signature = sha1($queryString);

详见微信官方文档 - JS-SDK使用权限签名算法：
http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd

本人认为比较关键的地方还有： 随机字符串以及时间戳，最好和支付签名保持一致。

"errMsg":"chooseWXPay:fail"

参考链接列表：

http://tieba.baidu.com/p/3961646394

http://www.liball.me/wxpay-is-shit/

这个问题，主要是支付目录配置有误或者参数有误。 网上主要搜集的解决方案如下：

1、配置授权 测试目录的时候地址的大小写要和代码里保持一致
2、运行测试不行，必须正式打apk包才能用
3、一个是支付授权目录的配置，一个是生成签名时用timeStamp,前端js用timestamp。 
4、支付目录配到二级或三级是不行的,一定要配到最后一级
5、引用微信js <script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
6、要在 微信公众平台 ---》微信支付----》开发配置----下面设置测试的路径即可
7、开发者中心 -》网页授权获取用户基本信息-》js安全域名
8、这个问题产生的原因很有可能是微信授权域名和微信支付域名设置的不一样，比如一个有www一个没有。
9、支付目录，指的是付款页面目录，而不是后台支付链接目录。

最终前端调用代码如下：
 
// js-sdk配置<br>wx.config({
    debug: true,
    appId: '', // 必填，公众号的唯一标识
    timestamp: '', // 必填，生成签名的时间戳,后端注意返回string类型
    nonceStr: '', // 必填，生成签名的随机串,自己生成，最长32位。
    signature: '', // 必填，微信签名，这个签名，和下面的paySign,所需用到的随机字符串和时间戳，最好和生成paySgin的保持一致。不是同一个。生成方法参考 http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html，可在页面 http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign 进行校验。
    jsApiList: [
      'chooseWXPay'
    ] // 必填，需要使用的JS接口列表，列表可选参数，参考 http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html 附录2.
});<br>// js-sdk配置验证成功
wx.ready(function(){<br>　　　　　// 调用支付函数
    wx.chooseWXPay({
        timestamp: '', // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
        nonceStr: '', // 支付签名随机串，不长于 32 位
        package: '', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
        signType: 'MD5', // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
        paySign: '', // 支付签名
        success: function (res) {
            // 支付成功后的回调函数
            alert('pay success');
        },
        cencel:function(res){<br>　　　　　　　　　　　　　　// 支付取消回调函数
            alert('cencel pay');
        },
        fail: function(res){<br>　　　　　　　　　　　　　　// 支付失败回调函数
            alert('pay fail');
            alert(JSON.stringify(res));
        }
    });
});<br>// js-sdk调用异常回调函数
wx.error(function(res){
    alert(res.err_msg);
});
调用统一下单接口，抛出超时异常 (28,'Rosolving timed out after 30014 milliseconds')

解决办法： 将DNS设置为 182.254.116.116， 参考链接：https://mp.weixin.qq.com/wiki/6/01405db0092f76bb96b12a9f954cd866.html
