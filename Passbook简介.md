### Pass的结构

既然一个票据就是一个Pass，那么什么是Pass呢？在iOS中一个Pass其实就是一个.pkpass文件，事实上它是一个Zip压缩包，只是这个压缩包要按照一定的目录结构来设计，下面是一个Pass包的目录结构（注意不同的票据类型会适当删减）：

Pass Package

├── icon.png

├── icon@2x.png

├── logo.png

├── logo@2x.png

├── thumbnail.png

├── thumbnail@2x.png

├── background.png

├── background@2x.png

├── strip.png

├── strip@2x.png

├── manifest.json

├── fr.lproj

│ └── pass.strings

├── de.lproj

│ └── pass.strings

├── pass.json

└── signature

也就是说在Passbook应用中显示的内容其实就是一个按照上面文件列表来组织的一个压缩包。在.pkpass文件中除了图标icon、缩略图thumbnail和logo外最重要的就是pass.json、manifest.json和signature。

- 1.pass.json

这个文件描述了Pass的布局、颜色设置、文本描述信息等，也就是说具体Pass包如何展示其实就是通过这个JSON文件来配置的，关于pass.json的具体配置项在此不再一一介绍，大家可以查看苹果官方帮助文档“[Pass Design and Creation](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/PassKit_PG/Chapters/Creating.html)”。这里主要说一下其中关键的几个配置项：

passTypeIdentifier:pass唯一标识，这个值类似于App ID，需要从开发者中心创建，并且这个标识必须以“pass”开头（例如下面的示例中取名为“pass.com.cmjstudio.mypassbook”）。

teamIdentifier：团队标识，申请苹果开发者账号时会分配一个唯一的团队标识（可以在苹果开发者中心--查看账户信息中查看”Team ID“）。

barcode:二维码信息配置，主要指定二维码内容、类型、编码格式。

locations:地理位置信息，可以配置相关位置的文本信息。

- 2.manifest.json

manifest.json 从名称可以看出这个文件主要用来描述当前Pass包中的文件目录组织结构。这个文件记录了除“manifest.json”、“signature”外的 文件和对应的sha1哈希值（注意：哈希值可以通过”openssl sha1 [ 文件路径]“命令获得）。

- 3.signature

signature是一个签名文件。虽然manifest.json存储了哈希值，但是大家都知道hash算法是公开的，如何保证一个pass包是合法的，未经修改的呢？那就是使用一个签名文件来验证。

------

## 功能开发

#### 前期准备

- 苹果开发者中心(itunes Contact)新建pass Type ID，基于该ID创建Passbook证书(Mac上的钥匙串中选择“从证书颁发机构请求证书”，生成证书请求文件，将此文件上传到对应的pass type ID下生成证书文件)，

- 

- 下载证书后，证书导入Mac中(此处配置的pass type id对应pass.json中的“passTypeIdentitifier”，用于生成签名文件)。

- Xcode开启Passbook

  ​     在Capabilities启用wallet服务，若勾选“allow All Team pass types”，则pass.json中的passTypeIdentifier和teamIdentifier就可以是任何团队创建，所以一般选择“allow subset of pass types”

#### 制作Pass

- 准备图片素材，icon，logo，strip等@2x大小

- 配置pass.json(注意下：pasTypeIdentifier和teamIdentifier)

  eg:{

  ​

  ​

  ​

  ​

  ​

  ​

  "formatVersion":1,

  "passTypeIdentifier":"pass.com.cmjstudio.mypassbook",

  "serialNumber":"54afe978584e3",

  "teamIdentifier":"JB74M3J7RY",

  "authenticationToken":"bc83dde3304d766d5b1aea631827f84c",

  "barcode":{"message":"userName KenshinCui","altText":"会员详情见背面","format":"PKBarcodeFormatQR","messageEncoding":"iso-8859-1"},

  "locations":[

  ​            {"longitude":-122.3748889,"latitude":37.6189722},{"longitude":-122.03118,"latitude":37.33182}],

  "organizationName":"CMJ Coffee",

  "logoText":"CMJ Coffee",

  "description":"",

  "foregroundColor":"rgb(2,2,4)",

  "backgroundColor":"rgb(244,244,254)",

  "storeCard":{

    "headerFields":[{"key":"date","label":"余额","value":"￥8888.50"}],

    "secondaryFields":[{"key":"more","label":"VIP会员","value":"Kenshin Cui"}],

    "backFields":[

  ​                {"key":"records","label":"消费记录(最近10次)","value":" 9/23   ￥107.00     无糖冰美式\n 9/21   ￥58.00     黑魔卡\n 8/25   ￥44.00     魔卡\n 8/23   ￥107.00     无糖冰美式\n 8/18   ￥107.00     无糖冰美式\n 7/29   ￥58.00     黑魔卡\n 7/26   ￥44.00     魔卡\n 7/13   ￥58.00     黑魔卡\n 7/11   ￥44.00     魔卡\n 6/20   ￥44.00     魔卡\n"},

  ​                {"key":"phone","label":"联系方式","value":"4008-888-88"},

  ​                {"key":"terms","label":"会员规则","value":"（1）本电子票涉及多个环节，均为人工操作，用户下单后，1-2个工作日内下发，电子票并不一定能立即收到，建议千品用户提前1天购买，如急需使用，请谨慎下单； \n（2）此劵为电子劵，属特殊产品，一经购买不支持退款（敬请谅解）； \n（3）特别注意：下单时请将您需要接收电子票的手机号码，填入收件人信息，如号码填写错误，损失自负；购买成功后，商家于周一至周五每天中午11点和下午17点发2维码/短信到您手机（周六至周日当天晚上发1次），请用户提前购买，凭此信息前往影院前台兑换即可； \n（4）订购成功后，（您在购买下单后的当天，给您发送电子券，系统会自动识别；如果您的手机能接收二维码，那收到的就是彩信，不能接收二维码的话，系统将会自动转成短信发送给您），短信为16位数，如：1028**********； 每个手机号码只可购买6张，如需购买6张以上的请在订单附言填写不同的手机号码，并注明张数（例如团购10张，1350755****号码4张，1860755****号码6张）；\n（5）电子票有效期至2016年2月30日，不与其他优惠券同时使用"},

  ​                {"key":"support","label":"技术支持","value":"http://www.cmjstudio.com\n\n               \n               \n               "}]

  },

  "labelColor":"rgb(87,88,93)"

  ​

  }

- 依据pass所需文件创建manifest.json，通过”openssl sha1 [文件路径]”分别获取所有文件的哈希值

eg:{

"pass.json":"3292f96c4676aefe7122abb47f86be0d95a6faaf",

"icon@2x.png":"83438c13dfd7c4a5819a12f6df6dc74b71060289",

"icon.png":"83438c13dfd7c4a5819a12f6df6dc74b71060289",

"logo@2x.png":"83438c13dfd7c4a5819a12f6df6dc74b71060289",

"logo.png":"83438c13dfd7c4a5819a12f6df6dc74b71060289",

"strip@2x.png":"885ff9639c90147a239a7a77e7adc870d5e047e2",

"strip.png":"885ff9639c90147a239a7a77e7adc870d5e047e2"

}

- 生成signature文件：【重要】

  a. 通过导出pass Type证书导出个人交换信息文件(.p12)并设置密码，保存;注意是导出证书而不是导出证书下的秘钥；

  b. 找到”Apple Worldwide Developer Relations CA - G2证书导出增强保密邮件(.pem);

  c. 将.p12证书转换为pem文件: 
  
  openssl pkcs12 -in passP12.p12 -clcerts -nokeys -out mypassbook.pem -passin pass:666666;

  注：上面pass必须为.p12证书内置的密码

  d. 证书p12文件导出私钥pem文件(passkey.pem) 

  openssl pkcs12 -in passP12.p12 -nocerts -out passkey.pem -passin pass:666666 -passout pass:999999；

  e. 根据AWDRCA.pem、mypassbook.pem、passkey.pem、manifest.json生成signature文件(按照提示输入passkey.pem导出时设置的密码999999)： 
  
  实际测试签名

  openssl smime -binary -sign -certfile AWDRCA.pem -signer mypassbook.pem -inkey passkey.pem -passin pass:999999

  以上命令都在当前签名文件夹操作,下面的命令是正式到处签名命令

  openssl smime -binary -sign -certfile AWDRCA.pem -signer mypassbook.pem -inkey passkey.pem -passin pass:999999 -in manifest.json -out signature -outform DER

- 将icon.png、icon@2x.png、logo.png、logo@2x.png、strip.png、strip@2x.png 、pass.json、manifest.json、signature压缩成pass包（这里命名为”mypassbook.pkpass“)。 

  zip -r mypassbook.pkpass manifest.json pass.json signature logo.png logo@2x.png icon.png icon@2x.png strip.png strip@2x.png。

#### 添加Pass到Wallet

将制作好的pass放到bundle中完成添加，实际开发中pass是由服务端动态生成，添加时会从服务端下载。iOS端提供了PassKit.Framework框架来进行开发

Demo:

------

### 参考资料 && tips

1.模拟在线制作Pass工具：[PassSource](https://www.passsource.com)

2.[官方文档Wallet Developer Guide](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/PassKit_PG/index.html#//apple_ref/doc/uid/TP40012195-CH1-SW1)

3.借鉴文章

<http://www.cnblogs.com/iCocos/p/4602804.html>

<http://blog.csdn.net/jolie_yang/article/details/51862285>

<http://blog.csdn.net/tonny_guan/article/details/8988035>