---
title: 七牛云使用Demo
date: 2022-09-21 18:12:02
tags: 
- java
- 七牛云
- spring
index_img: /img/article6.jpg
banner_img: /img/post_banner.jpg
categories:
- other
---

### 前言

<p class="note note-success">
    微服务中采用分布式架构，假设用户访问app时，服务可能通过集群配分配到海底或者山洞里的某个幸运服务器，这时图片单一部署在服务器需要采用统一URI，于是云存储图片服务获得URI进行部署即可解决问题。本次采用的是七牛云作为Demo测试（优惠）省。
</p>

### 七牛云前置工作环境搭建

#### 1.注册登录

#### 2.构建存储空间

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/20220924153231.png)

<p class="note note-success">
   <1>选择<font color="#FF00FF">对象存储</font></br>
       </p>


![新建存储空间](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220924153909853.png)

<p class="note note-success">
   <2>进入<font color="#FF00FF">新建空间</font>，存储空间名称即入口，请依照规范标准命名；存储区域根据自己地理位置进行选择；访问控制公开</br>
       </p>

![image-20220924154507736](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220924154507736.png)

<p class="note note-success">
   <3>进入构建空间，获取<font color="#FF00FF">系统分配的域名前缀</font>以便程序中使用</br>
       </p>

<p class="note note-success">
   <4>官方SDK工具中找到对应编程语言得SDK配置文档，嫌麻烦就直接找客服</br>
       </p>

<p class="note note-success">
    <5>点击<font color="#FF00FF">个人中心->密钥管理</font>获取AccessKey/SecretKey</br>
       </p>

### java案例

#### SDK工具类

```	java
public class QiniuyunUtils {
    // AccessKey 
    public static String accessKey = "ftn79KS0pDDPoGqtd9wNyNICRuMq_fqo9Jt3Gjj1";
    // SecretKey
    public static String secretKey = "pKUaRo5vxUKtZVSrCpy5J5kayUtpcvsu9ykbm1mh";
    // 云空间名称
    public static String bucket = "CloudDemo";

    public static void upload2Qiniu(String filePath,String fileName){
        //构造一个带指定Zone对象的配置类
        Configuration cfg = new Configuration(Zone.zone1());
        UploadManager uploadManager = new UploadManager(cfg);
        Auth auth = Auth.create(accessKey, secretKey);
        String upToken = auth.uploadToken(bucket);
        try {
            Response response = uploadManager.put(filePath, fileName, upToken);
            //解析上传成功的结果
            DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
        } catch (QiniuException ex) {
            Response r = ex.response;
            try {
                System.err.println(r.bodyString());
            } catch (QiniuException ex2) {
                //ignore
            }
        }
    }

    //上传文件
    public static void upload2Qiniu(byte[] bytes, String fileName){
        //构造一个带指定Zone对象的配置类
        Configuration cfg = new Configuration(Zone.zone1());
        //...其他参数参考类注释
        UploadManager uploadManager = new UploadManager(cfg);

        //默认不指定key的情况下，以文件内容的hash值作为文件名
        String key = fileName;
        Auth auth = Auth.create(accessKey, secretKey);
        String upToken = auth.uploadToken(bucket);
        try {
            Response response = uploadManager.put(bytes, key, upToken);
            //解析上传成功的结果
            DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
            System.out.println(putRet.key);
            System.out.println(putRet.hash);
        } catch (QiniuException ex) {
            Response r = ex.response;
            System.err.println(r.toString());
            try {
                System.err.println(r.bodyString());
            } catch (QiniuException ex2) {
                //ignore
            }
        }
    }

    //删除文件
    public static void deleteFileFromQiniu(String fileName){
        //构造一个带指定Zone对象的配置类
        Configuration cfg = new Configuration(Zone.zone1());
        String key = fileName;
        Auth auth = Auth.create(accessKey, secretKey);
        BucketManager bucketManager = new BucketManager(auth, cfg);
        try {
            bucketManager.delete(bucket, key);
        } catch (QiniuException ex) {
            //如果遇到异常，说明删除失败
            System.err.println(ex.code());
            System.err.println(ex.response.toString());
        }
    }
}

```

#### 测试类

```java
public class QinNiuTest {

    @Test
    public void testUpload(){
        String filePath = "C:\\Users\\0\\Desktop\\avatar.jpg";
        String fileName = "123.jpg";
        QiniuyunUtils.upload2Qiniu(filePath,fileName);
    }

    @Test
    public void testDelete(){
        String fileName = "123.jpg";
        QiniuyunUtils.deleteFileFromQiniu(fileName);
    }
}

```

