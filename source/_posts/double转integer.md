---
title: double转integer
date: 2019-09-28 11:13:36
tags: JAVA
categories: JAVA
---
（1）把double先转化成int类型
  ```
    Double  reseve3=Double.parseDouble(bddet[0].getReserve3());
    int b=reseve3.intValue();
 ```
      
  
（2）再把int类型转化为Integer类型
  ```
      Integer rentCount=Integer.valueOf(b);
  ```