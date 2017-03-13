---
author: springloops
comments: true
date: 2011-05-23 07:56:34+00:00
layout: post
link: https://springloops.wordpress.com/2011/05/23/ubuntu-1004-grub-%eb%b3%b5%ea%b5%ac/
slug: ubuntu-1004-grub-%eb%b3%b5%ea%b5%ac
title: '[Ubuntu 10.04] GRUB 복구.'
wordpress_id: 33
categories:
- Ubuntu 10.04
---

펌 : http://www.ubuntuk.com/4023  

  
이것도 해봐야지..  

휴..   

  
여기서부터 ---------------------------------------  

  





우분투 or 쿠분투 v9.10버전으로 넘어오면서 기존 Grub복구방법이 더이상 통하지 않게되었습니다.




이번에 새로 나온 Grub2버전의 복구 방법입니다.




  



제가 직접 실험해보았습니다. 잘되네요.







********************************************************************************************************************




1. 우분투 라이브 시디로 부팅합니다.  

  
2. 터미널 띄웁니다.




sudo fdisk -l




  



저의 경우 이렇게 나오네요.


/dev/sda1 29 8369 66999082+ 83 Linux  

/dev/sda2 * 8370 13995 45190845 7 HPFS/NTFS  

/dev/sda3 13996 14593 4803435 5 Extended  

/dev/sda5 13996 14593 4803403+ 82 Linux swap / Solaris

  



3. 우분투가 인스톨되어 있는 /dev/sda1 를 마운트시키려 합니다.  



`sudo mount /dev/sda1 /mnt`  

`sudo mount --bind /dev /mnt/dev`  

`sudo mount --bind /proc /mnt/proc`

  



4. resolv.conf 라는 파일을 복사합니다.


`sudo cp /etc/resolv.conf /mnt/etc/resolv.conf`

  



5. 루트(root)로 들어갑니다.(루트로 들어간 이후엔 더이상 sudo 라는 명령이 필요치 않은 것 아시죠^^)  



`sudo chroot /mnt`

  



6. 만약 /etc/default/grub 파일을 편집할 필요가 있다면 편집합니다.  



nano -w /etc/default/grub

  



7. 이제 GRUB를 복구합니다.


grub-install /dev/sda

  



만약 설치가 안되고 에러가 나면


grub-install --recheck /dev/sda

  



8. 마운트된 볼륨들을 해제하고 종료합니다.  



exit  

sudo umount /mnt/dev  

sudo umount /mnt/proc  

sudo umount /mnt

  



9. 재부팅 합니다.




sudo reboot







  



*참고한 페이지




- [우분투 블로그](http://www.ubuntu-inside.me/2009/06/howto-recover-grub2-after-windows.html)  





