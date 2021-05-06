# 记录学习瞬间（外部）

**说明** ：该文档用于记录开发中遇到的问题，如有需要协助或不规范的地方，请联系工作人员邱伟豪，为了您的阅读便利，我们会及时更正错误。感谢您的阅读。注：我发现每次想起曾经使用过的技术都是记得不太清楚，里面的核心大意理解了，但是等到说出来，可能就没有什么逻辑性，现在为了记清楚我们平时的技术细节，特意编写该文档。

## 1.crypto加密原理

**crypto功能**：例如open_ssl的哈希、HMAC、加密、解密
<br/>
**什么是hash算法**：hash就是散列算法，把任意长度的值变换成固定长度的值，例如sha1、md5
<br/>

### 1.1 hash算法，No.1表

| 属性名| 不可逆 |  加密长度|  特征|支持16进制|  支持Base64|应用场景|安全性|
| :--------  | :-----  | :----:  | :----:  | :----:  | :----:  | :----:  | :----:  |
|MD5 | 是 |固定|内容相同输出结果相同|是|是|明文传参加密、缓存|容易被串改，安全性不高|
|Hmac算法 | 是 |固定|加盐算法，哈希算法+密钥|是|是|API请求验证|密钥容易被劫持，可以通过随机数密钥|

### 1.2开放API接口验证

**接口安全问题**：请求身份是否合法、请求参数是否被串改、请求是否唯一
<br/>
**AccessKey&secretKey开放平台**：请求身份
<br/>
**防止篡改**：1，按照字母升序排列拼接字符串。2，加上密钥。3，通过MD5加密得到sign值
<br/>
**解决潜在问题**：1，重放攻击，使用唯一字符串nonce，每次记录下来，避免二次请求。2，记录nonce的时候，通过添加timestamp来避免大量累计nonce。3，例如：15分钟内的nonce有效，超过15分钟的清除，记录下来的nonce放在redis中。通过redis的expire来判断超时性。
<br/>
#### 1.2.1客户端：

```js
1. 生成当前时间戳timestamp=now和唯一随机字符串nonce=random
2. 按照请求参数名的字母升序排列非空请求参数（包含AccessKey)
   `stringA="AccessKey=access&home=world&name=hello&work=java&timestamp=now&nonce=random";`
3. 拼接密钥SecretKey
   `stringSignTemp="AccessKey=access&home=world&name=hello&work=java&timestamp=now&nonce=random&SecretKey=secret";`
4. MD5并转换为大写
   `sign=MD5(stringSignTemp).toUpperCase();`
5. 最终请求
   `http://api.test.com/test?name=hello&home=world&work=java&timestamp=now&nonce=nonce&sign=sign;
```

#### 1.2.2服务端：

图
![10418241-7643ad86154b365d](https://user-images.githubusercontent.com/30063579/117234786-3ab18680-ae58-11eb-9570-690083d02822.png)


### 1.3token&APPKey（APP）

**token身份验证**：1，客户端登录账户密码传给服务端。2，服务端验证成功返回token，客户端保存token在本地，每次请求携带token。3，服务端检查token的有效性，无效（过期）则拒绝
<br/>
**安全隐患**：token被劫持、伪造请求和篡改参数。解决方案：通过签名手段，避免二次请求。同时，用户没有sign，即使劫持token也请求不成功。sign的签名方式，请看crypto加密原理
<br/>
![10418241-7ba0289f4aec6c29](https://user-images.githubusercontent.com/30063579/117234794-400ed100-ae58-11eb-95db-13b1d76100e6.png)

