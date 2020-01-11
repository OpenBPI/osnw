# 开放式社交网络（OSN网络）之八

## 企业现有系统接入OSN网络的解决方案客户端篇
>&emsp;&emsp;企业现有系统已经有了自己的完整体系，要完成“跨界通信”需要在用户终端和服务器端进行一定的适配。
>&emsp;&emsp;客户端适配需要做的工作包括：生成OSN账户、添加跨界好友、发送和接收跨界消息。
>**生成OSN账户**
>客户端现有的账户体系需要与OSN账户做一个映射。
>OSN账户是ECDSA公钥的散列组合，ECDSA采用Prime256V1曲线。
>账号标志头：OSN
>账号主体：base58编码字符串，base58编码与bitcoin一致。以后的所有base58都使用此编码。
>>账号主体二进制定义：
>|字段尺寸|描述|数据类型|说明|
>|2byte|版本|uchar|账号使用版本号，初始为1000|
>|1byte|公钥标志|uchar|ECDSA公钥采用压缩模式和非压缩模式两种 04表示非压缩模式，公钥长度为64byte 03 02 表示压缩模式，公钥长度为32byte|
>|64/32byte|公钥|uchar|非压缩公钥长度为64byte，压缩公钥只保留X，长度为32byte|
>|32byte|shadow hash|uchar|一个地址的生成需要2对椭圆加密密钥对，后一对在地址中仅保留公钥的hash|
>
>客户端账户推荐在用户终端生成，生成以后由客户端发起与原有账户的绑定。
>&emsp;
>**添加跨界好友**
>&emsp;&emsp;由于每个企业的制度不同，通过关键字搜索添加好友不一定能成功，因此推荐由双方好友相互交换OSN账户来添加好友。
>客户端需要添加显示OSN账户，复制或者导出OSN账户，显示二维码等功能。
>&emsp;
>**客户端通信录格式**
>客户端通信录格式推荐字段
>|me|key|friend|randnum|randomstring|nickname|face|personalized|comment|phone|group|...|
>randomstring之后的字段可以为空字符串。
>&emsp;
>**发送和接收跨界消息**
>&emsp;&emsp;发送跨界消息需要加密，根据OSN账户的生成方式，可以从OSN账户中取出对方好友公钥。
>>**消息格式封装如下：**
>{
>“command”:”message”,
>“from”:”OSNXXXXXXXXXXXXXXXXXXXXXXXXXXXX”,
>“to”:” OSNXXXXXXXXXXXXXXXXXXXXXXXXXXXX”,
>“crpyto”:”yes/no”,
>“content”:”base58 string”,
>“description”: ”base58 string”,
>“hash”:”hash”,
>“timestamp”:””
>“sign”:”xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx”
>}
>**content的生成方式：**
>content使用base58编码，采用asn1格式，并分成key和cipher两部分。
>key中包含了随机密码，其生成方式为
>1.由客户端生成随机SHA256字符串。
>2.使用to的公钥对随机SHA256进行加密。
>**cipher的生成方式为**
>1.使用随机SHA256作为key对需要发送的明文进行加密。
>2.加密方式采用AES128cbc，hash前128bit作为key，后128bit作为vi。
>&emsp;
>key和cipher组成asn1格式以后使用base58编码，如果crypto为no，则解码以后无需解密。
>解密需要from的私钥先解码出key，然后用key解码cipher。
>&emsp;
>description生成方式同content。
>&emsp;
>**sign的生成方式：**
>使用通讯录中的key对hash进行签名。签名格式请参阅开放式社交网络通信协议中的签名。
>&emsp;
>**接收到消息后的步骤：**
>1.验证消息签名
>2.对content中的内容进行base58解码
>3.解码asn1
>4.使用私钥对密文进行解码以后取出随机密码
>5.使用AES解密密文。
>**发送消息回执**
>接收消息完成以后，需要给对方发送一个消息获取的回执。
>**回执格式：**
>{
>“command”:”complete”,
>“from”:”OSNXXXXXXXXXXXXXXXXXXXX”,
>“to”:”OSNXXXXXXXXXXXXXXXXXXXX”,
>“sign”:”xxxxxxxxxxxxxxxxxxxxxxxxxxxxx”,
>“msghash”:[“hash1”,”hash2”,”hash3”]
>}
>Sign:签名是将所有的hash链接成字符串以后签名。
>&emsp;
>接收到回执以后可以确认自己消息已经被对方成功接收。完成消息发送。
>
>**其他命令请参阅开放式社交网络通信协议。**

