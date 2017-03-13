---
author: springloops
comments: true
date: 2011-05-22 08:41:28+00:00
layout: post
link: https://springloops.wordpress.com/2011/05/22/%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98-%ec%9a%b0%eb%b6%84%ed%88%acubuntu-agt-get-%ec%a0%80%ec%9e%a5%ec%86%8c-%ec%b6%94%ea%b0%80-%eb%b0%8f-%ec%9d%b8%ec%a6%9d%ed%82%a4-%eb%93%b1%eb%a1%9d/
slug: '%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98-%ec%9a%b0%eb%b6%84%ed%88%acubuntu-agt-get-%ec%a0%80%ec%9e%a5%ec%86%8c-%ec%b6%94%ea%b0%80-%eb%b0%8f-%ec%9d%b8%ec%a6%9d%ed%82%a4-%eb%93%b1%eb%a1%9d'
title: '[패키지 설치] 우분투(Ubuntu) agt-get 저장소 추가 및 인증키 등록 방법'
wordpress_id: 31
categories:
- Ubuntu 10.04
---

우분투로 패키지 설치시 애를 먹어서.. 100% 또 찾아볼 거란것을 알기때문에 글 남김.  

  

아래 블로그에서 가져왔음.   

[http://adsgear.tistory.com/129   

  

](http://adsgear.tistory.com/129)우분투에서 apt-get 을 이용하여 패키지 설치시 필요에 따라 관련 패키지에 대한 저장소를 등록을 해야 하며,   

어떤 경우는 공개키를 추가해 주어야 한다.  

  

virtualbox의 저장소를 예로해서 자료를 정리 해 놓으려고 등록 합니다.   




    
     
    // 아래  저장소 목록을 추가 한다. 
    $ sudo add-apt-repository "deb http://us.archive.ubuntu.com/ubuntu dapper main restricted"
    
    // 패키지 목록 업데이트 
    $ sudo apt-get update
    
    // 인증키가 등록되지 않아서 아래와 같은 오류가 나오실거예요.  
    // W: GPG 오류: http://download.virtualbox.org lucid Release: 다음 서명들은 공개키가 없기 때문에
    // 인증할수 없습니다 : NO_PUBKEY 54422A4B98AB5139
    
    //  위에서 나온 공개키 54422A4B98AB5139 복사한다음 공개키를 추가해 주세요. 
    $ gpg --keyserver keyserver.ubuntu.com --recv 54422A4B98AB5139
    $ gpg --export --armor 54422A4B98AB5139 | sudo apt-key add -
    
    //  공개키 추가후 다시 업데이트하면 오류없이 잘 넘어 갑니다. 
    $ sudo apt-get update
    
    //  버츄얼 박스 설치  
    $ apt-get install virtualbox-3.2
    
    
    
