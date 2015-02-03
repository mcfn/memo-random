
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
    
