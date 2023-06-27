title: 本地数字证书生成和tomcat开启SSL认证
author: Mario
date: 2023-06-27 13:46:14
tags:
---
记录一下本地数字证书生成的命令及过程，以及tomcat如何开启SSL认证

<!--more-->
#### 本地数字证书生成（jks格式证书生成）
``` bash
0、进入java中jdk\jre\bin目录

1、生成服务器证书（D:\apache-tomcat-8.5.56-https\conf为本地tomcat配置文件目录）
keytool -genkey -alias server -keypass 123456 -storepass 123456 -keyalg RSA -keystore D:\apache-tomcat-8.5.56-https\conf\server.keystore -validity 36500 

2、生成客户端证书jks文件
keytool -genkeypair -alias client -keyalg RSA -validity 36500 -keypass 123456 -storepass 123456 -keystore D:\apache-tomcat-8.5.56-https\conf\client.jks

3、生成p12文件
keytool -genkey -v -alias mykey -keyalg RSA -storetype PKCS12 -keystore D:\apache-tomcat-8.5.56-https\conf\mykey.p12

4、让服务器信任客户端证书
4.1
P12生成cer
keytool -export -alias mykey -keystore D:\apache-tomcat-8.5.56-https\conf\mykey.p12 -storetype PKCS12 -storepass 123456 -rfc -file D:\apache-tomcat-8.5.56-https\conf\mykey.cer

jks生成cer
keytool -export -alias client -file D:\apache-tomcat-8.5.56-https\conf\client.cer -storepass 123456 -keystore D:\apache-tomcat-8.5.56-https\conf\client.jks

4.2 将证书导入到服务器证书库
P12转换的cer导入到server库中
keytool -import -v -file D:\apache-tomcat-8.5.56-https\conf\mykey.cer -keystore D:\apache-tomcat-8.5.56-https\conf\server.keystore

将jks转换的cer导入到server库中
keytool -import -v -alias client -file D:\apache-tomcat-8.5.56-https\conf\client.cer -keystore D:\apache-tomcat-8.5.56-https\conf\server.keystore -storepass 123456

5、双向验证
导出服务器证书为cer
keytool -export -alias server -keystore D:\apache-tomcat-8.5.56-https\conf\server.keystore -storepass 123456 -file D:\apache-tomcat-8.5.56-https\conf\server.cer

6、双击安装mykey.p12导入客户端证书

7、导入服务器公钥证书（server.cer）,双击安装server.cer,将证书安装本地，选择目录“受信任的根证书颁发机构”

```
#### tomcat开启SSL认证
修改tomcat配置 tomcat\conf\server.xml，找到注释的https配置位置，修改为如下内容，证书生成时就放在和配置文件同一目录下，证书密码123456，在生成时设置的，具体配置含义可自行百度
``` bash
      <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                 maxHttpHeaderSize="8192"
                 maxThreads="150"
                 minSpareThreads="25" 
                 maxSpareThreads="75"
                 enableLookups="false" 
                 disableUploadTimeout="true"
                 acceptCount="100" 
                 SSLEnabled="true" 
                 scheme="https" 
                 secure="true" 
                 clientAuth="true" 
                 sslProtocol="TLS" 
                 keystoreType="JKS"
                 keystoreFile="conf/server.keystore"
                 keystorePass="123456"
                 truststoreFile="conf/server.keystore"
                 truststorePass="123456"
                 URIEncoding="GBK"/>

```
#### 访问及证书获取
``` bash
1、浏览器访问：tomcat启动后访问8443端口就会要求证书双向验证通过才能访问
2、程序代码获取证书信息，本地安装证书后，tomcat配置keystoreType="JKS"时，每次请求证书会在request对象中，获取证书的方式为：
X509Certificate[] certs = (X509Certificate[]) request.getAttribute("javax.servlet.request.X509Certificate");
获取证书后可以对特定证书进行解析，做特殊业务逻辑处理

```