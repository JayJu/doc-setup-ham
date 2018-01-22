# Kurento Media Server 설치 및 환경구성

  * 설치환경
    * RHEL 7.4
    * 참고자료: [Kurento Media Server and components](https://github.com/pkgs-cloud/kurento/blob/master/README.md)
   
---
1. 기본 프로그램 설치 및 사용자 생성
  * [기본 프로그램 설치 및 사용자 생성](./default/00-etc.md)
  
2. pkgs.cloud release repository 설치(ham 으로 로그인 후 진행)
  * 참고자료: [pkgs.cloud Release Repository – RPM packages for RHEL / CentOS 7](https://github.com/pkgs-cloud/release)
  
  * repository package 설치
  ```
  $ sudo yum install https://get.pkgs.cloud/release.rpm -y
  ```
  * 사용가능한 repository package 확인
  ```
  $ yum --disablerepo="*" --enablerepo="pkgs.cloud" list available
  ```

3. 확장 패키지 설치
  * epel package 설치
  ```
  $ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E '%{rhel}').noarch.rpm
  $ sudo yum install epel-release -y <-- CentOS인 경우
  $ yum repolist
  $ sudo yum update
  ```
4. Kurento 패키지 & KMS 설치
  * Kurento package 설치
  ```
  $ sudo yum install kurento-release -y
  $ sudo vi /etc/yum/pluginconf.d/search-disabled-repos.conf
  notify_only=0 으로 변경
  $ sudo yum install kms-6.6.3 -y
  $ sudo vi /etc/yum/pluginconf.d/search-disabled-repos.conf
  notify_only=1 으로 원복
  ```
  * 추가 패키지 설치(필요시)
  ```
  $ sudo yum install kms-filters-chroma -y
  $ sudo yum install kms-filters-crowddetector -y
  $ sudo yum install kms-filters-platedetector -y
  $ sudo yum install kms-filters-pointerdetector -y
  ```
5. STUN 서버 등록
  * 설정파일 위치: ``` /etc/kurento ```
  * 공개 STUN 서버 목록
    * https://gist.github.com/zziuni/3741933
  * STUN 서버 등록
  ```
  $ sudo vi /etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini
  stunServerAddress=74.125.23.127
  stunServerPort=19302
  ```
6. 방화벽 포트 오픈
  * firewall-cmd 설치
  ```
  $ sudo yum install firewalld
  $ sudo systemctl start firewalld
  ```
  * TCP/UDP 포트 오픈
    * [방화벽포트오픈](04-firewall.md) - 미디어서버 참조

7. 서비스 시작/종료
  ``` 
  $ sudo systemctl enable kms.service
  $ sudo systemctl start kms.service
  $ sudo systemctl restart kms.service
  ``` 
8. 데몬 limits 프로파일 변경
  ```
  $ ls /usr/lib/systemd/system/kms*
  $ sudo mkdir -p /etc/systemd/system/kms.service.d
  $ cd /etc/systemd/system/kms.service.d
  $ sudo vi limits.conf
  $ cat /etc/systemd/system/kms.service.d/limits.conf
    [Service]
    LimitNOFILE=50000
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart kms
  $ ps -ef |grep kurento
  $ cat /proc/{프로세스ID}/limits
  ```

9. 로그 확인
  * 로그파일 위치
    * ``` /var/log/kurento ```
    * ``` /etc/sysconfig/kms ```
