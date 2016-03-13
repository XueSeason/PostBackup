title: Nginx配置SSL教程
date: 2015-06-21 08:38:45
tags: Web
---

# 购买证书

在Namecheap购买SSL证书后，进入个人主页，你会看到如下信息：  

You have purchased SSL certificates and at least 1 certificate(s) are not yet activated. Please go to the SSL Certificates page and click on 'Activate Now' next to your SSL purchase item to start the issue process.

大致意思就是说你已经购买了SSL证书但没有被激活需要激活。点击超链接，进入证书激活页面。然后找到SSL证书列表找到你购买的产品，点击Activate Now。

在Select web server的选项中选择nginx服务器。然后就是填写你的CSR。此时我们暂时没有CSR。
<!-- more -->
# 生成SSL证书获取CSR

在你的电脑上执行命令：


```
sudo openssl genrsa -des3 -out server.key 2048
```


一种是2048位，一种是1024位。Namecheap要求是生成2048位的密钥文件。之后会让你输入加密密码。  
然后创建一个证书签名：


```
sudo openssl req -new -key server.key -out server.csr
```

输入刚才填写的密码，这里会让你填写一大堆信息。按照要求写就好了，需要注意的是里面common name填的是你网站的域名。完成后可以在当前目录下看到新生成的.csr和.key文件。  

# Namecheap SSL证书激活

执行cat server.csr获取里面的加密字符串，提供给Namecheap，确认后你会看到需要你确认接受邮件的邮箱地址。这里的联系人邮箱必须是关联的域名邮箱。  
确定后你会收到一封激活邮箱，按步骤激活成功后又会收到一封带附件的邮箱。附件里面的四个文件就是安装SSL证书必备的文件。  

# Nginx安装SSL证书

新建一个server.crt文件，然后按照文件顺序依次添加内容到该文件中，确保没有空格。
xxxx.crt、COMODORSADomainValidationSecureServerCA.crt、COMODORSAAddTrustCA.crt、AddTrustExternalCARoot.crt。  

将新生成的server.crt文件和之前的server.key文件上传到服务器。  

在服务器执行`nginx -V`查看http ssl模块是否支持。  

修改服务器的配置文件，可以参考如下内容：

```
listen       443;
ssl on;
ssl_certificate /root/server.crt;
ssl_certificate_key /root/server.key;
ssl_session_timeout 5m;
```

保存后执行`service nginx restart`重启nginx。

# 一些科普知识
关于生成的\*.key文件是你的私钥，通过\*.key文件生成的\*.csr为证书请求文件，你只要把CSR文件提交给证书颁发机构后，证书颁发机构使用其根证书私钥签名就生成了证书公钥文件即后来生成的\*.crt文件，也就是颁发给用户的证书。
