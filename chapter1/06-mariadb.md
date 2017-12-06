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
  $ sudo yum -y install mariadb-server
  $ sudo systemctl enable mariadb
  $ sudo systemctl start mariadb
  $ sudo systemctl status mariadb
  $ su -
  # mysql_secure_installation
  ## Set root password? Y
  ## Remove anonymous users? Y
  ## Disallow root login remotely? N
  ## Remove test database and access to it? Y
  ## Reload privilege tables now? Y
  ```
  * 복제구성 준비
  ```
  $ sudo yum install -y rsync lsof
  ```
3. Galera 마스터 노드 구성
  * server.cnf 수정
  ```
  $ sudo vi /etc/my.cnf.d/server.cnf
  [galera]
  # Mandatory settings
  wsrep_on=ON
  wsrep_provider=/usr/lib64/galera/libgalera_smm.so
  wsrep_cluster_address="gcomm://10.6.180.69,10.6.180.70,10.6.180.68"
  binlog_format=row
  default_storage_engine=InnoDB
  innodb_autoinc_lock_mode=2
  # Allow server to accept connections on all interfaces.
  bind-address=0.0.0.0
  # Galera Cluster Configuration
  wsrep_cluster_name="hamcluster1"

  # Galera Synchronization Configuration
  wsrep_sst_method=rsync

  # Galera Node Configuration
  wsrep_node_address="10.6.180.69"
  wsrep_node_name="aehamdbp01"
  ```
