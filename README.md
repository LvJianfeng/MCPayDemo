# MCPayDemo
• 支付宝 //文档idk都包含了安卓、iOS版
• 银 联
• 银联官网资料
• Demo
Demo给了一个订单号，做测试使用，若出现支付失败什么的，可能是已经被别人给支付了，或者是服务器订单过期了 ~
一、支付宝
1.1 请阅读支付宝文档和Demo
1.2 导入对应的库
将支付宝Demo中得这些东西全拷贝过来
1 localhost:alipay mac$ ls
2 APAuthV2Info.h        Order.h            libssl.a
3 APAuthV2Info.m        Order.m            openssl
4 AlipaySDK.bundle    Util
5 AlipaySDK.framework    libcrypto.a

导入系统
1.SystemConfiguration.framework
设置一下search paths
1 build setting ->搜索search path，然后你懂的完成后，编译一下，看有没有错，有错没错，还是下一步吧。
1.3 对接
支付宝对节前，你还是需要从服务器拿到一下一堆东西
支付宝接口文档中写了3p参数列表，--！ 总结下我用的到，或者说是Demo中提到的，别的就超出范围了
1.合作者身份ID     alipayPartner = @"2088一串数字";
2.接口名称         alipaySeller = @"tianticar@126.com";
3.签名            aliPayPrivateKey = @"很长很长的私钥";
4.公钥            alipayRSA_PUBLIC=@"一般长";  客户端不用服务器都给我了--~！
5.服务器异步通知页面路径  alipayNotifServerURL = @"一个网址"; //支付结果，支付宝会通知服务器

其他一些参数（与购买产品相关，设计到业务了，客户端/服务器谁提供均可）直接贴order代码了，具体看我的Demo示例.
1. Order \*order = [[Order alloc] init];
2. order.partner = alipayPartner ;
3. order.seller = alipaySeller;
4. order.tradeNO = tn; 订单ID（由商家自行制定）
5. order.productName = [NSString stringWithFormat:@"汽车服务充值-%@",@"支付"]; 商品标题
6. order.productDescription = [NSString stringWithFormat:@"%@:支付宝移动支付充值",@"xxxx"]; 商品描述
7. order.amount = _txtCNY.text; 商品价格
8. order.notifyURL =  alipayNotifServerURL; 回调URL
9. order.service = @"mobile.securitypay.pay";
10.order.paymentType = @"1";
11.order.inputCharset = @"utf-8";
12.order.itBPay = @"30m";
13.order.showUrl = @"m.alipay.com";
14.应用注册scheme,在AlixPayDemo-Info.plist定义URL types
15.NSString *appScheme = URLScheme;
上海-牛牛  10:52:57
https://github.com/poholo/MCPayDemo
上海-牛牛  10:53:17
调用支付宝
1.[[AlipaySDK defaultService] payOrder:orderString fromScheme:appScheme c 
   allback:^(NSDictionary *resultDic) {
2. NSLog(@"reslut = %@",resultDic);
3. if ([resultDic[@"resultStatus"] intValue]==9000) {
4. 进入充值列表页面
5. NSLog(@"支付成功");                             }
7. else{
8. NSString *resultMes = resultDic[@"memo"];
9. resultMes = (resultMes.length<=0?@"支付失败":resultMes);
10.NSLog(@"%@",resultMes);
11.                    }
12.     }];

你可能会发现回调不行->设置回调shema
1 上面支付时已经传给了支付宝客户端回调shema名称
2  NSString *appScheme = URLScheme;
3  具体设置shema方法此处就不再累赘，这儿需要处理来自支付宝shema回调，才能完成上面方法的block回调
4  在APPDelegate -
5   - (BOOL)application:(UIApplication *)application
6             openURL:(NSURL *)url
7   sourceApplication:(NSString *)sourceApplication
8          annotation:(id)annotation {
9       跳转支付宝钱包进行支付，处理支付结果
10     [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
11         NSLog(@"result = %@",resultDic);
12     }];
13     return YES;
14 }
