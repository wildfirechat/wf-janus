webrtc DTLS需要证书进行加密通讯，需求签名生产1024位的证书（更高位数的证书容易出现DTLS Timeout，原因是超过单个最大MTU大小，导致无法发送成功，具体原因请参考[这里](https://github.com/meetecho/janus-gateway/issues/252)。默认我们提供有签名，如果需要更换，请按照下面说明生成签名。

1. 生成RSA密钥
```
openssl genrsa -des3 -out privkey.pem 1024
```
生成过程中会需要设置密码，需要记住密码，后面需要使用


2. 生成证书
```
openssl req -new -x509 -key privkey.pem -out cacert.pem -days 364635
```
生成过程中需要填写密钥密码，就是第一步生成密钥时设置的密码。这个命令将用上面生成的密钥privkey.pem生成一个数字证书cacert.pem，有效期是999年，


3. 配置新证书
把这两个文件放到janus的config目录下，然后修改```janus.jcfg```配置文件。
```
certificates: {
        cert_pem = "/var/janus/janus/etc/janus/cacert.pem"
        cert_key = "/var/janus/janus/etc/janus/privkey.pem"
        cert_pwd = "123456"
        #dtls_accept_selfsigned = false
        #dtls_ciphers = "your-desired-openssl-ciphers"
        #rsa_private_key = false
```

4. 重启janus服务，看看有没有报错，然后再测试是否正常工作。
