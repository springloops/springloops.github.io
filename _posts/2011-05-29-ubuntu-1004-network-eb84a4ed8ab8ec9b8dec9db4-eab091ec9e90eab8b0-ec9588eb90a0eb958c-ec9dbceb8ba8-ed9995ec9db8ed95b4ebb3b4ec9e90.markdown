---
author: springloops
comments: true
date: 2011-05-29 16:00:16+00:00
layout: post
link: https://springloops.wordpress.com/2011/05/29/ubuntu-1004-network-%eb%84%a4%ed%8a%b8%ec%9b%8d%ec%9d%b4-%ea%b0%91%ec%9e%90%ea%b8%b0-%ec%95%88%eb%90%a0%eb%95%8c-%ec%9d%bc%eb%8b%a8-%ed%99%95%ec%9d%b8%ed%95%b4%eb%b3%b4%ec%9e%90/
slug: ubuntu-1004-network-%eb%84%a4%ed%8a%b8%ec%9b%8d%ec%9d%b4-%ea%b0%91%ec%9e%90%ea%b8%b0-%ec%95%88%eb%90%a0%eb%95%8c-%ec%9d%bc%eb%8b%a8-%ed%99%95%ec%9d%b8%ed%95%b4%eb%b3%b4%ec%9e%90
title: '[ubuntu 10.04 network] 네트웍이 갑자기 안될때 일단 확인해보자..'
wordpress_id: 35
categories:
- Ubuntu 10.04
tags:
- Linux
- Network
- ubuntu10.04
- ubuntu10.04 network
---

아래 명령어 실행후 네트워크 정보가 등록 되어 있는지 확인해보자..  

나는 유동IP 이라서 다음과 같이 정리 해줬다.  

  
sudo vi /etc/network/interfaces  

  



auto lo




iface lo inet loopback




  




auto eth0




iface eth0 inet dhcp




  




#auto wlan0




#iface wlan0 inet dhcp




  

  
그리고 아래 명령어로 네트워크 재시작..


sudo /etc/init.d/networking restart  

  
  

유선 네트워크는 잘 작동 하는데  

무선은 아직...ㅡㅡ;  

  
갑자기 왜 이러냥...
