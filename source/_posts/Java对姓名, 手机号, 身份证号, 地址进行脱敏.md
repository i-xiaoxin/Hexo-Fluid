---
title: Java数据脱敏工具类
date: 2023-03-13 20:09:02
tags: 
- Java
index_img: 
banner_img: /img/article2.webp
categories:
- Java
comment: waline
---



# Java数据脱敏工具类

```java

package com.util;

import org.apache.commons.lang3.StringUtils;

/**
 * <p>
 * 数据脱敏工具类
 * </p>
 */
public class DataDesensitizedUtils {

    /**
     * 功能描述：姓名脱敏
     * 脱敏规则：只显示第一个汉字,比如李某某置换为李**, 李某置换为李*
     * @param fullName 完整的姓名
     * @return
     */
    public static String desensitizedName(String fullName) {
        if (StringUtils.isNotBlank(fullName)) {
            String name = StringUtils.left(fullName, 1);
            return StringUtils.rightPad(name, StringUtils.length(fullName), "*");
        }
        return fullName;
    }

    /**
     * 功能描述：手机号脱敏
     * 脱敏规则：保留前三后三, 比如15085375241置换为150*****241
     * @param phoneNumber 手机号
     * @return
     */
    public static String desensitizedPhoneNumber(String phoneNumber) {
        if (StringUtils.isNotBlank(phoneNumber)) {
            phoneNumber = phoneNumber.replaceAll("(\\w{3})\\w*(\\w{3})", "$1*****$2");
        }
        return phoneNumber;
    }
  	// 手机号码前三后四脱敏
    public static String mobileEncrypt(String mobile) {
        if (StringUtils.isEmpty(mobile) || (mobile.length() != 11)) {
            return mobile;
        }
        return mobile.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");


    /**
     * 功能描述：身份证号脱敏
     * 脱敏规则：保留前六后三, 适用于15位和18位身份证号
     * @param idNumber 身份证号
     * @return
     */
    public static String desensitizedIdNumber(String idNumber) {
        if (StringUtils.isNotBlank(idNumber)) {
            return StringUtils.left(idNumber, 6).concat(StringUtils.removeStart(StringUtils.leftPad(StringUtils.right(idNumber, 3), StringUtils.length(idNumber), "*"), "******"));
        }
        return idNumber;
    }

    /**
     * 功能描述：地址脱敏
     * 脱敏规则：从第4位开始隐藏,隐藏8位
     *         因地址位数是不确定的,所以结尾长度为总长度减去 前面保留长度和隐藏长度之和 address.length()-11
     * @param address 具体地址
     * @return
     */
    public static String desensitizedAddress(String address) {
        if (StringUtils.isNotBlank(address)) {
            return StringUtils.left(address, 3).concat(StringUtils.removeStart(StringUtils.leftPad(StringUtils.right(address, address.length() - 11), StringUtils.length(address), "*"), "***"));
        }
        return address;
    }

    public static void main(String[] args) {
        System.out.println(desensitizedName("张三"));
        System.out.println(desensitizedPhoneNumber("15085375241"));
        System.out.println(desensitizedIdNumber("122424205003164013"));
        System.out.println(desensitizedAddress("浙江省杭州市滨江区"));
    }
}


```





<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>
