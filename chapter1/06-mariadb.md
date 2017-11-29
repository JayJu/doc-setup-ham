# MariaDB 설치 및 클러스터(Galera) 구성

  * 설치환경
    * RHEL 7.4
    * 참고자료: [Galera Cluster MariaDB Configuration On CentOS 7](https://linuxadmin.io/galeria-cluster-configuration-centos-7/)
    * Galera 클러스터는 3 node 설치가 기본이므로 rep, db1, db2 로 3개 노드 구성
   
---
1. 기본 프로그램 설치 및 사용자 생성
  * [기본 프로그램 설치 및 사용자 생성](./default/00-etc.md)

2. MariaDB 설치 (3 노드 모두)
  * MariaDB repository 추가
  ```
  # vi /etc/yum.repos.d/MariaDB10.1.repo
  ### 아래 내용 추가
  # MariaDB 10.1 RedHat repository list - created 2017-11-29 07:26 UTC
  # http://downloads.mariadb.org/mariadb/repositories/
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.1/rhel7-amd64
  gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  ```
  * DB 패키지설치(ham 계정, 클러스터 포함되어있음)
  ```
  # su - ham
  $ sudo yum remove mysql-libs
  $ sudo yum remove mariadb-common.x86_64 mariadb-config.x86_64
  $ sudo rm -rf /var/cache/yum*
  $ yum clean all
  $ sudo yum -y install MariaDB-server MariaDB-client MariaDB-common
  ```