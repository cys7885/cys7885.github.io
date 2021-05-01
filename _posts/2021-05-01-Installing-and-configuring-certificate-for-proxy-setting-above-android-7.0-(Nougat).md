---
title: Installing and configuring certificate for proxy setting above android 7.0 (Nougat)
author: Chanyoung So
date: 2021-05-01 17:30:00 +0800
categories: [Mobile, Android]
tags: [Security, Android]
---

Starting with Android 7.0 (Nougat) and later, SSL Pining is applied, and if you install a user private certificate locally, it is registered as a user certificate rather than a Root Certificate and cannot be used as a trusted certificate.

This means that you cannot install a local user private certificate on an existing Android and intercept SSL packets using a proxy tool SSL packets.

Therefore, Android 7.0 and later requires SSL Pinning bypass to use a proxy.

----



### 1. How to bypass SSL Pinning on Android?

Three methods below allow us to bypass SSL Pinning.

- Bypass SSL Pinning with System Certificate Installation
- Android SSL Pinning Bypass with Frida
- Bypass SSL Pinning via APK Repackaging



Each of the above three methods has its own strengths and weaknesses.



1. Bypass SSL Pinning with System Certificate Installation
   - Advantages
     - <u>**After the first installation of a certificate at the Android system level, proxy can be easily used from now on.**</u>
   - Disadvantages
     - Rooted Android Device Required
2. Android SSL Pinning Bypass with Frida
   - Advantages
     - <u>**In addition to Android, SSL-Pinning bypass is also available via hooking on IOS.**</u>
     - Many features supported by frida are available.
   - Disadvantages
     - Rooted Android Device Required

3. Bypass SSL Pining via APK Repackaging
   - Advantages
     - <u>**No Rooted Device Required**</u>
   - Disadvantages
     - Need to repackage for each analysis application



Users can selectively use the method they are familiar with according to the situation.



### 2. Bypass SSL Pinning with System Certificate Installation

#### Requirements

- Rooted Android Device ( >= Android 7.0)
- OpenSSL



#### Command Summary

```
# Convert DER to PEM 
openssl x509 -inform DER -in cacert.der -out cacert.pem

# Get subject_hash_old (or subject_hash if OpenSSL < 1.0)
openssl x509 -inform PEM -subject_hash_old -in cacert.pem
# Copy the hash value that appears on the first line of the result value. ex)9a5ba575

# Rename the certificate .pem to <hash>.0
mv cacert.pem 9a5ba575.0

# Remount and copy certificate to device
adb root
adb remount
adb push 9a5ba575.0 /system/etc/security/cacerts/
adb shell
chmod 644 /system/etc/security/cacerts/9a5ba575.0
reboot
```



### Tutorial

- #### Download Burp Certificate

  It connects to the local proxy server via the Web.

  It downloads Burp certificates by pressing `CA Certificate` button.

<img src="https://user-images.githubusercontent.com/19899140/116776867-e7c97f00-aaa5-11eb-804a-9156da31f99f.png" style="zoom:67%;" />



- #### Convert Burp certificates to pem format using OpenSSL

  It converts a Burp certificate of type der to a pem extension via openssl.

  Extract the subject_hash_old value and copy the value (`9a5ba575`) as shown below.

<img src="https://user-images.githubusercontent.com/19899140/116776842-bbadfe00-aaa5-11eb-9c9d-cf70729e3da6.png" style="zoom:50%;" />



- #### Rename pem certificate

  Rename of the pem certificate to `<hash>.0` with the hash value obtained above. (The extension becomes .0)

  ![](https://user-images.githubusercontent.com/19899140/116776889-0f204c00-aaa6-11eb-9a1b-1c6ff4d4b977.png)

- #### Remount device's disk and Copy the certificate to the system

  Because the system directory is 'Read-only file system', the disk must be remounted and copied.

  ![](https://user-images.githubusercontent.com/19899140/116776902-1cd5d180-aaa6-11eb-98e0-01e0ff30dc42.png)

  

- #### Register Burp Certificate

  Check the list of trusted certificates in `Settings -> Security & Location -> Encryption & Credentials -> Trusted Credentials` on the Android device.

  `PortSwigger` is registered as a trusted certificate.

  ![](https://user-images.githubusercontent.com/19899140/116776875-f9ab2200-aaa5-11eb-9c15-43215d432d02.png)

  

### 3. Android SSL Pinning Bypass with Frida

- Scheduled to be filled out later

### 4. Bypass SSL Pinning via APK Repackaging

- Scheduled to be filled out later

### Reference

1. https://blog.ropnop.com/configuring-burp-suite-with-android-nougat
2. https://www.mcafee.com/enterprise/en-us/assets/misc/ms-bypass-ssl-pinning-android-4-6.pdf
3. https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/
4. https://android-developers.googleblog.com/2016/07/changes-to-trusted-certificate.html

