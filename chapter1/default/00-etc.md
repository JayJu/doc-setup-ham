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
3. DNS 설정
  ```
  # vi /etc/resolv.conf
  nameserver 8.8.8.8
  ```
4. Buld 도구 설치
  ```
  # yum groupinstall 'Development Tools'
  ```
5. iftop \(network 모니터링\)
  ```
    $ apt-get install iftop
    $ su -
    # chmod +s $(which iftop)
  ```
6. htop \(서버 모니터링\)
  ```
    # wget dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
    # rpm -ihv epel-release-7-11.noarch.rpm
    # yum groupinstall "Development Tools" <-- 기 설치 했음
    # yum install ncurses ncurses-devel
    # wget http://hisham.hm/htop/releases/2.0.2/htop-2.0.2.tar.gz
    # tar xvfvz htop-2.0.2.tar.gz
    # cd htop-2.0.2
    # ./configure
    # make
    # make install
    # cd ..
    # rm -rf ./htop*
    # htop
  ```



