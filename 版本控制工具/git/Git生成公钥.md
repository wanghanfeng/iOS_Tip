git在进行同步仓库时，采用的SSH(Secure Shell)安全外壳协议，ssh的加密算法基于RSA,即非对称加密。所以当和git服务器通过ssh通信的时候需要向git服务器提前上传RSA的公钥，便与git服务器确认我们的身份。因为，我们用私钥加密信息，对方用公钥解密，而私钥只有我们有（私钥唯一性），所以对方只要能用公钥进行解密，若信息格式满足协议要求，就代表此信息是使用与公钥相匹配的私钥进行加密的，也就代表信息是我们上传的（信息的签名）。

今天，使用git上传信息时突然提示权限拒绝，这就是说明我们没有向git同步的权限。整体来看，是RSA的公钥和私钥出现了问题。

![Snip20170319_2.png](http://upload-images.jianshu.io/upload_images/2312315-8694231f10463268.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决办法:重新生成公钥，并将其上传至Github.

````
$ cd ~/.ssh
$ ssh-keygen -t rsa -C “wanghanfeng3@gmail.com”   #填写github上的默认邮箱
````
不需要设密码的话直接三个回车。

得到 id_rsa 和 id_rsa.pub 两个文件, id_rsa是RSA的私钥文件，id_rsa.pub是RSA的公钥文件。

将id_rsa.pub文件中的内容完整复制到Github的个人设置=》SSH and GPG keys中。

问题解决。
