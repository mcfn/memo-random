# Memo todo-list


## JWT

http://jwt.io/
https://cnodejs.org/topic/53c652bfc9507b404446ee40

JSON Web Token is a compact URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature (JWS)

当使用一个API时，其中一个挑战就是认证（authentication）。在传统的web应用中，服务端成功的返回一个响应（response）依赖于两件事。一是，他通过一种存储机制保存了会话信息（Session）。每一个会话都有它独特的信息（id），常常是一个长的，随机化的字符串，它被用来让未来的请求（Request）检索信息。其次，包含在响应头（Header）里面的信息使客户端保存了一个Cookie。服务器自动的在每个子请求里面加上了会话ID，这使得服务器可以通过检索Session中的信息来辨别用户。这就是传统的web应用逃避HTTP面向无连接的方法。

API应该被设计成无状态的（Stateless）。这意味着没有登陆，注销的方法，也没有sessions，API的设计者同样也不能依赖Cookie，因为不能保证这些request是由浏览器所发出的。自然，我们需要一个新的机制。这篇文章关注于JSON Web Tokens，简写为JWTs，


一下几个通常的保护（secure）API的方法。
* 使用在HTTP规范中所制定的Basic Auth.  
 它需要在在响应中设定一个验证身份的Header。客户端必须在每个子响应是附加它们的凭证（credenbtial），包括它的密码。如果这些凭证通过了，那么用户的信息就会被传递到服务端应用。
* 使用应用自己的验证机制。  
通常包括将发送的凭证与存储的凭证进行检查。和Basic Auth相比，这种需要在每次请求（call）中发送凭证。
* 第三种是OAuth（或者OAuth2）。  
为第三方的认证所设计，但是更难配置。至少在服务器端更难。



JWTs是一认证机制。JWT是URL-safe的，意味着可以用来查询字符参数，也就是可以脱离URL，不用考虑URL的信息。一个JWT被分成了三个部分。
* Header
* Claim
* Signature

## Header
一个JSON对象。固定格式，alg定义签名算法
```json
{
  "typ" : "JWT",
  "alg" : "HS256"
}
```

更多的签名算法，可以参照：http://jwt.io/

## Claim
一个JSON对象，是token的核心。包含了一些信息。有一些是必须的，有一些是选择性的。一个实例如下：
```json
{
  "sub": 1234567890,
  "name": "mcfeng",
  "admin": true
}
```
## Signature
一个字符串。第三个也是最后一个部分，是JWT根据第一部分和第二部分的签名（Signature）

## 简单签名流程
一般是, header和claim的json通过base64进行encode成string，然后通过"."连接在一起，使用秘钥和相关加密算法进行加密得到signature字符串，再通过"."连接起来。
```php
$headerString = base64Encode($headerJSONString);
$claimString = base64Encode($claimJSONString)
$signContent = "$headerString.$claimString";
// header中定义的签名算法
$signature = encodeByAlgorithm(signContent);
// 最后的jwt
$jwtString = "$signContent.$signature";

```


可以通过在线网站decode，对JWT进行实时encode/decode。
比如： http://jwt.io/


客户端，存储者签名的秘钥。当收到服务器的jwt时候，通过秘钥对header和claim进行一次签名，
与服务器端的sigature验证是否一致。




# SSLv3
Poodle: Padding Oracle On Downgraded Legacy Encryption
是一个中间人攻击

避免方式：
* 实现TLS_FALLBACK_SCSV
防止downgrade dance.

在header中包含版本号，如果不符合就拒绝请求。防止protocol downgrade. (Attacks remain possible if
both parties allow SSL3.0 but one of them is not updated to support TLS_FALLBACK_SCSV, provided that client implements a downgrade dance down to SSL3.0)
* 关闭SSL 3.0的支持
服务器和客户端都要disable SSL3.0。If either side supports only SSL3.0, then all hope is gone, and a serious update required to avoid insecure encryption. If SSL 3.0 is neither disabled nor the only possible protocol version, then the attack is possible if the client uses a downgrade dance for
interoperability.

https://www.openssl.org/~bodo/ssl-poodle.pdf


比较新的协议：TLS1.0, TLS1.1

many TLS clients implement a downgrade dance: in a first handshake attempt, offer the highest protocol version supported by the client; if this handshake fails, retry (possibly repeatedly) with earlier protocol versions.

http://drops.wooyun.org/papers/3194
