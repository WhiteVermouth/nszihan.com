---
title: "密码的哈希存储"
date: "2018-09-22 14:41:37"
---

## 前言

密码的存储关乎安全问题，明文存储密码会带来极大的安全隐患。因此，需要存储密码哈希值来保证安全性。

## MD5

`MD5`(Message-Digest Algorithm)算法会产生一个 128bit（16byte）的哈希值，用十六进制表示为长度 32 字符。

### 示例

```java
String passwordToHash = "password";
String generatedPassword = null;
try {
    // Create MessageDigest instance for MD5
    MessageDigest md = MessageDigest.getInstance("MD5");
    //Add password bytes to digest
    md.update(passwordToHash.getBytes());
    //Get the hash's bytes
    byte[] bytes = md.digest();
    //This bytes[] has bytes in decimal format;
    //Convert it to hexadecimal format
    StringBuilder sb = new StringBuilder();
    for(int i=0; i< bytes.length ;i++)
    {
        sb.append(Integer.toString((bytes[i] & 0xff) + 0x100, 16).substring(1));
    }
    //Get complete hashed password in hex format
    generatedPassword = sb.toString();
}
catch (NoSuchAlgorithmException e)
{
    e.printStackTrace();
}
```

### MD5 的缺陷

- 容易遭受暴力破解(`brute-force`)和字典攻击(`dictionary attack`)
- 通过彩虹表(rainbow table)可以快速得到密码原文
- 不能抵御哈希碰撞，不同的密码可能产生相同的哈希值

### 通过加盐（salt）增加 MD5 的安全性

维基百科对盐（salt）的定义如下

> 盐（Salt），在密码学中，是指在散列之前将散列内容（例如：密码）的任意固定位置插入特定的字符串。这个在散列中加入字符串的方式称为“加盐”。其作用是让加盐后的散列结果和没有加盐的结果不相同，在不同的应用情景中，这个处理可以增加额外的安全性。

#### 盐的产生方式

- 盐可以是特定字符串
- 也可以是随机字符串
- 随机产生的盐需要存储起来用作下次验证
  - 随机产生盐的示例
  ```java
  private static byte[] getSalt() throws NoSuchAlgorithmException{
      //Always use a SecureRandom generator
      SecureRandom sr = SecureRandom.getInstance("SHA1PRNG");
      //Create array for salt
      byte[] salt = new byte[16];
      //Get a random salt
      sr.nextBytes(salt);
      //return salt
      return salt;
  }
  ```

### MD5 加盐示例

```java
private static String getSecurePassword(String passwordToHash, byte[] salt)
{
    String generatedPassword = null;
    try {
        // Create MessageDigest instance for MD5
        MessageDigest md = MessageDigest.getInstance("MD5");
        //Add password bytes to digest
        md.update(salt);
        //Get the hash's bytes
        byte[] bytes = md.digest(passwordToHash.getBytes());
        //This bytes[] has bytes in decimal format;
        //Convert it to hexadecimal format
        StringBuilder sb = new StringBuilder();
        for(int i=0; i< bytes.length ;i++)
        {
            sb.append(Integer.toString((bytes[i] & 0xff) + 0x100, 16).substring(1));
        }
        //Get complete hashed password in hex format
        generatedPassword = sb.toString();
    }
    catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
    return generatedPassword;
}
```

## SHA

`SHA`(Secure Hash Algorithm) 是一个能够产生更强安全性的散列函数算法。`SHA`不能避免碰撞，但相比`MD5`碰撞出现的概率要低得多。`Java`实现了 4 种`SHA`算法：

- `SHA-1` (160bit)
- `SHA-256` (256bit)
- `SHA-384` (384bit)
- `SHA-512` (512bit)

安全性依此增强。

### 使用方法

```java
MessageDigest md = MessageDigest.getInstance("SHA-1");
MessageDigest md = MessageDigest.getInstance("SHA-256");
MessageDigest md = MessageDigest.getInstance("SHA-384");
MessageDigest md = MessageDigest.getInstance("SHA-512");
```

其余代码同`MD5`示例一致。

## PBKDF2WithHmacSHA1

// TODO

本文翻译自（未全部翻译）: https://howtodoinjava.com/security/how-to-generate-secure-password-hash-md5-sha-pbkdf2-bcrypt-examples/
