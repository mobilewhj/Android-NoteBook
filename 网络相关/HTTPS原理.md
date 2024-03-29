## HTTPS

加密通信的HTTP。

数字证书：证书+数字签名

数字签名：对证书进行hash（摘要信息）之后用私钥进行加密产生的一个字符串，可以理解为数字证书编号。

连接过程：

* 客户端请求建立TLS连接。

  * 发送Client Hello 、TLS版本集合、加密套件集合（可选的对称加密算法、可选的非对称加密算法、可选的hash算法）、客户端生成的随机数等数据。
* 服务器将证书返回。

  * 发送Server Hello、TSL版本、加密套件（对称加密算法、非对称加密算法、hash算法）、服务器生成的随机数。
  * 再次发送服务器证书（数字证书+数字签名）。核心意义是将服务器的公钥发送给客户端，为了让客户端使用公钥加密数据发送给服务器使用。
  * 再次发送交换秘钥请求，并返回一些服务器参数，主要为了生成秘钥使用。
  * 发送Serve Hello Done 标识握手完毕。
* 客户端验证服务器证书。

  * 验证证书的合法性：
    * 客户端本地的CA机构公钥对服务器返回的证书里的数字签名进行解密得到摘要信息。
    * 使用同样的hash算法对服务器返回的证书进行计算，得到摘要信息，和上一步的摘要信息做对比，相同则证明来源一致，没有被劫持。
    * 验证证书基本信息（证书链）
* 客户端验证通过之后，会进行和服务端协商加密过程（交换加密秘钥过程）：

  * 客户端会生成一个premaster secret，并使用服务器返回的公钥进行加密，发送给服务端。
  * 服务器使用私钥解密，获取到premaster secret，这样客户端和服务端各自有三个随机数据。
    * 客户端随机数
    * 服务器随机数
    * 客户端随机字符串
  * 客户端和服务器会通过共同的加密算法通过三个随机数合成一个mastersecret，mastersecret会生成秘钥套件：
    * 客户端加密秘钥
    * 服务端加密秘钥
    * 客户端MAC secret
    * 服务器MAC secret
    * MACsecret 是通过master secret使用秘钥派生函数生成的如HMAC-SHA256。
  * 客户端会发个消息会通知服务器，开始使用加密方式通信。
  * 客户端发送个Finished:
    * 将之前通信的数据组合成数据，然后使用客户端加密秘钥加密，然后再通过客户端MAC secret做计算，然后合并一起发送给服务器。
    * 服务器收到消息，使用客户端秘钥解密出通信数据，然后同样使用MACsecret做计算，并将对比结果。
  * 服务器会发个消息会通知客户端，开始使用加密方式通信：
    * 将之前通信的数据组合成数据，然后使用服务器加密秘钥加密，然后再通过服务器MAC secret做计算，然后合并一起发送给客户端。
    * 客户端收到消息，使用服务器秘钥解密出通信数据，然后同样使用MACsecret做计算，并将对比结果。
  * 双方都验证了加密解密没问题，后续通信就是用对称加密进行http请求。
