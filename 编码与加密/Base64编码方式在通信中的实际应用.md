> 加密就是为了安全通信而诞生的。没有通信，加密也没有太大存在的意义。

虽说Base64算不上一种加密，只是一种具有固定标准的编码方式，但如果适当的变换一下编码字典，不采用标准的编码方式，那对于不知道编码字典的第三方来说就是一种加密方式。

### 用途
在介绍Base64编码之前，我想先介绍下Base64的实际应用：

1）在前端中，当网页上有数目庞大的图片或其他资源时，浏览器会建立大量的http请求去获取资源。这样对服务器来说会产生大量的请求，浪费服务器性能。对于客户端来说，会多次请求资源，网络利用率不高，用户体验不好。所以，前端上有一个解决办法就是，对于大量的较小图片采用雪碧图（Sprite）,一次将一张较大图片加载回来，然后进行选择展示。但是这样会在js、css、html代码中增加很多附加代码，不利于后期操作与维护。于是，Base64的优势便显现出来，可以将图片进行Base64编码后嵌入到网页中，在展示时在将其进行Base64解码，显示在网页上。举一个具体的实例：在google的首页搜索框的搜索小图标就是在用这样的方式实现。

2）迅雷的种子：），大家一定懂得。为了不直接使下载链接暴露出来，迅雷（旋风等下载工具）都会对链接进行Base64编码。因为Base64编解码规则是公开的，所以你只需要对迅雷的种子进行Base64解码之后拿到下载链接，在别的地方下载，但是迅雷不会这样单纯的。一般来说迅雷会对编码表进行定制，在对编码方式进行一个修改，这样不知道规则的人就无法解密了。

3）在只能发送字符的通信方式进行文件传输。文件是由一系列二进制数据组成，而字符集的编码数量是小于byte的编码数量的（ASCII:0~127、BYTE:0~255）。所以通过只能发字符的通信方式进行文件的传输看上去有些不可能，但是Base64恰恰解决的这个问题，Base64可以将byte映射到64个可见字符上。使用Base64将文件进行编码，形成一个字符串，在将字符串传给对方，对方收到后在进行解码，就可以还原出这个文件。

4）简单的加密通信。本文开始时，提到Base64使用非标准的字典进行编码时，对于第三方来说就是一种加密方式，对于第三方来说，想要破解，只能通过概率论来将编码字典得出。对于一些对加密效率要求高，而不要求太高的安全性的应用来说是一个比较好的选择。

### Base64是什么
Base64是基于64个可见字符的编码方式，从Base64这个名字上可以得出。RCF文档中对标准Base64的编码字典规定为：
"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
‘=’最为填充字符
可以将任意二进制数映射到这64个字符中。

### Base64的原理
举一个实际的例子来说明：
###### 情况一
对二进制 `0x12` 、`0x34`、`0x56`，进行Base64编码。
1）`00010010 `、`00110100`、`01010110`，每六个一组分为：`000100`、`100011`、`010001`、`010110`。
2）在计算机看来，上述新分组后的值为：`00000100`、`00100011`、`00010001`、`000101110`。相当于对每组数的高两位进行了零填充，也就是说我们分完组之后的数所能表示的最大的范围就是2^6 : 0 ~ 63，共64个数。
3) 按照一一对应的方式，在编码字典中选取响应位置上的字符，分别是第`12`、`33`、`17`、`46`位，查编码字典为：`M`、`h`、`R`、`u`。
4）编码后为：`"MhRu"`

最理想的情况是，待编码的字节数是3的倍数，编码后的字节数是4的倍数，而上述展示的就是最理想的情况。

###### 情况二
下面展示除3余1的情况：
对二进制`0x12` 、`0x34`、`0x56`、`0x78`，进行Base64编码。
1）`00010010 `、`00110100`、`01010110`、`01111000`，每六个一组分为：`000100`、`100011`、`010001`、`010110`、`011110`、`000000`
2) 在计算机看来，上述新分组后的值为：`00000100`、`00100011`、`00010001`、`000101110`、`00011110`、`00000000`。
3) 按照一一对应的方式，在编码字典中选取响应位置上的字符，分别是第`12`、`33`、`17`、`46`、`0`位，查编码字典为：`M`、`h`、`R`、`u`、`e`、`A`
4) 为了满足编码后字节数为4的倍数，需要在编码后的字符后面添加'='填充符，`M`、`h`、`R`、`u`、`e`、`A`、`=`、`=`
5) 编码后为：`"MhRueA=="`

###### 情况三
下面展示除3余2的情况：
对二进制`0x12` 、`0x34`、`0x56`、`0x78`，`0x9a`进行Base64编码。
1) `00010010 `、`00110100`、`01010110`、`01111000`、`10011010`，每六个一组分为：`000100`、`100011`、`010001`、`010110`、`011110`、`001001`、`101000`
2) 在计算机看来，上述新分组后的值为：`00000100`、`00100011`、`00010001`、`000101110`、`00011110`、`00001001`、`00101000`。
3) 按照一一对应的方式，在编码字典中选取响应位置上的字符，分别是第`12`、`33`、`17`、`46`、`30`、`9`、`40`位，查编码字典为：`M`、`h`、`R`、`u`、`e`、`J`、`o`
4）为了满足编码后字节数为4的倍数，需要在编码后的字符后面添加'='填充符，`M`、`h`、`R`、`u`、`e`、`J`、`o`、`=`
5) 编码后为：`"MhRueJo="`

##### 而解码就是编码的逆过程

### 总结
可以看出，编码前和编码后的空间占用比在文件很大时为3：4的关系，小文件的占用比例会略大。所以，Base64编码的缺点就是空间占用比增大，消耗了cpu的资源进行编码。由于Base64编码使用的都是可打印的普通字符，所以就能极大的减少在传输转换中的错误率。

### 代码实现
```

public class HFBase64 {
//编码表，可以自定义，这样只要双方都知道编码表，就变成一种加密方式了
private static String encodingTable = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

public HFBase64() {

}
//编码方法
public String enCode(byte[] datas) {
if (datas == null) {
return null;
}
String ciphertext = "";
Integer byteIndex = 0;
while (byteIndex < datas.length) {
Integer index = 0;
byte[] temByte = new byte[3];
byte temp;
while (index < 3 && byteIndex < datas.length)
temByte[index++] = datas[byteIndex++];
ciphertext += encodingTable.charAt((temByte[0] & 0xfc) >> 2);
ciphertext += encodingTable.charAt(((temByte[0] & 0x03) << 4) | ((temByte[1] & 0xf0) >> 4));
if (index > 1) {
ciphertext += encodingTable.charAt(((temByte[1] & 0x0f) << 2) | (temByte[2] & 0xc0) >> 6);
} else {
ciphertext += '=';
}

if (index > 2) {
ciphertext += encodingTable.charAt((temByte[2] & 0x3f));
} else {
ciphertext += '=';
}
}

return ciphertext;
}
//解码方法
public byte[] deCode(String datas) {
if (datas == null) {
return null;
}
Integer len = (datas.length()) / 4 * 3;
byte[] plainBytes = new byte[len];

Integer charCount = 0;
Integer index = 0;

while (charCount < datas.length()) {

plainBytes[index++] = (byte) ((this.convertByte(datas.charAt(charCount)) << 2) | (this.convertByte(datas.charAt(++charCount))) >> 4);
if (datas.charAt(charCount + 1) == '=') {
//				plainBytes[--index] = (byte) (this.convertByte(datas.charAt(charCount)) >> 4);
break;
}
plainBytes[index++] = (byte) ((this.convertByte(datas.charAt(charCount)) << 4) | (this.convertByte(datas.charAt(++charCount))) >> 2);
if (datas.charAt(charCount + 1) == '=') {
//				plainBytes[--index] = (byte) (this.convertByte(datas.charAt(charCount)) >> 2);
break;
}
plainBytes[index++] = (byte) ((this.convertByte(datas.charAt(charCount)) << 6) | (this.convertByte(datas.charAt(++charCount))));
charCount++;
}

byte[] newBytes = new byte[index];
for (int i = 0; i < index; i++) {
newBytes[i] = plainBytes[i];
}

return newBytes;
}

private byte convertByte(char EncodeChar) {
byte index = (byte) encodingTable.indexOf(EncodeChar);
return index;
}
}

```
##### [代码地址 点我](https://github.com/wanghanfeng/HFBase64.git)
