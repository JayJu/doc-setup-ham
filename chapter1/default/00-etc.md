# 기본 프로그램 설치
 * 설치환경
    * RHEL 7.4   
---
1. Subscription Manager 로그인
  ```
  # subscription-manager register --username autoeveridcadmin  --auto-attach
  # yum update
  ```
2. 사용자 추가 및 권한 설정
  * 사용자 추가
  ```
  # useradd ham -m -d /home/ham
  # passwd ham
  ```
  * sudoer 등록
  ```
  # visudo
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)       ALL
  ham     ALL=(ALL)       ALL
  ```

4. iftop \(network 모니터링\)
  ```
    $ apt-get install iftop
    $ su -
    # chmod +s $(which iftop)
  ```
5. htop \(서버 모니터링\)
  ```
    $ apt-get install htop
    $ su -
    # chmod +s $(which htop)
  ```



