![v2-5f53dbdf1201d338b07bd478a9183019_r](https://user-images.githubusercontent.com/30063579/121281781-0d4e7180-c90b-11eb-8be8-2642b895d1f8.jpg)
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

## 2.egg.js

### 2.1内置对象

**微注**：继承koa4个内置对象，context、request、respond、Application，及框架扩展的一些对象（Logger,Controller,Service,Helper,Config）

### 2.1.2Application

**事件**：启动自定义脚本进行监听工作。

```js
1，server：该事件一个worker进程只会触发一次，在一个http启动之后，提供http server 給开发者。
2，onerror：运行有任何的异常都会被error捕获。对错误对象和关联的上下文暴露给开发者。
3，request和response：当收到请求和相应请求的时候，会触发。开发者可以监听request和response进行日志记录。
```

## 3.TCP网络

### 3.1socket

**什么是socket** linux进程间的一种通信方式。

```js
1，不仅能同一机器跨进程通信，还能不同机器间跨进程通信。
2，internet socket :不同主机间通信，通过pid标记唯一进程，标识：ip+port+协议。socket:[tcp, 发送端ip,发送端port,接受端ip,接受端port]
3，socket:[tcp, 发送端ip,发送端port,接受端ip,接受端port]
4，
(1)流式套接字：保证通信数据传输是正确
(2)数据报套接字：使用UDP协议，提供无连接的服务，不保证数据，需要自身有保证数据的协议。
(3)原始套接字：可自行封装协议
```
![v2-5f53dbdf1201d338b07bd478a9183019_r](https://user-images.githubusercontent.com/30063579/121281821-1dfee780-c90b-11eb-920e-852a16b7eba5.jpg)
