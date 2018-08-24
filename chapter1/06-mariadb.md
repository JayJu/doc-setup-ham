# MariaDB 설치 및 클러스터(Galera) 구성

  * 설치환경
    * RHEL 7.4
    * 참고자료: [Galera Cluster MariaDB Configuration On CentOS 7](https://linuxadmin.io/galeria-cluster-configuration-centos-7/)
    * Galera 클러스터는 3 node 설치가 기본이므로 rep, db1, db2 로 3개 노드 구성
   
---
1. 기본 프로그램 설치 및 사용자 생성
  * [기본 프로그램 설치 및 사용자 생성](./default/00-etc.md)

2. MariaDB 설치 (3 노드 모두)
  * MariaDB repository 추가(10.1)
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
  
  * MariaDB repository 추가(10.3)
  ```
  # vi /etc/yum.repos.d/MariaDB10.3.repo
  ### 아래 내용 추가
  # MariaDB 10.3 CentOS repository list - created 2018-08-23 09:17 UTC
  # http://downloads.mariadb.org/mariadb/repositories/
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.3/centos7-amd64
  gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  ```

  * DB 패키지설치(ham 계정, 클러스터 포함되어있음)
  ```
  # su - ham
  $ sudo yum -y install mariadb-server mariadb-client
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
    * galera 섹션 추가
    ```
    $ sudo vi /etc/my.cnf.d/server.cnf
    [galera]
    # Mandatory settings
    wsrep_on=ON
    wsrep_provider=/usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address="gcomm://10.6.xx.xx,10.6.xx.xx,10.6.xx.xx"
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
    wsrep_node_address="10.6.xx.xx"
    wsrep_node_name="aehamdbp01"
    ```
    * mysqld 섹션에 로그경로 추가
    ```
    [mysqld]
    log_error=/var/log/mariadb.log
    collation-server = utf8_unicode_ci
    init-connect='SET NAMES utf8'
    character-set-server = utf8
    ```
    * 더미로그 생성
    ```
    $ sudo touch /var/log/mariadb.log
    $ sudo chown mysql:mysql /var/log/mariadb.log
    ```
  * mysql-clients.cnf 수정
    ```
    [mysql]
    default-character-set=utf8
    [mysqldump]
    default-character-set=utf8
    ```
4. 서비스 시작
  * 마스터노드 클러스터 서비스 시작 및 포트확인
  ```
  $ su -
  # galera_new_cluster
  # lsof -i:4567
  COMMAND  PID  USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
  mysqld  1349 mysql   11u  IPv4 8741428      0t0  TCP *:tram (LISTEN)
  # lsof -i:3306
  COMMAND  PID  USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
mysqld  1349 mysql   26u  IPv4 8741444      0t0  TCP *:mysql (LISTEN)
  ```
  * 서비스 확인
  ```
  # mysql -uroot -p
  MariaDB [(none)]> SHOW STATUS LIKE 'wsrep_cluster_size';
  +--------------------+-------+
  | Variable_name      | Value |
  +--------------------+-------+
  | wsrep_cluster_size | 1     |
  +--------------------+-------+
  ```
  * DB 생성
  ```
  MariaDB [(none)]> create database hamdb;
  MariaDB [(none)]> grant all privileges on hamdb.* to 'ham'@'localhost';
  MariaDB [(none)]> grant all privileges on hamdb.* to 'ham'@'%';
  MariaDB [(none)]> flush privileges;

출처: http://ora-sysdba.tistory.com/entry/MariaDB-Maria-DB-데이터베이스-생성-권한-부여-접속 [Welcome To Ora-SYSDBA]
  ```
5. 방화벽 오픈
  * [방화벽포트오픈](04-firewall.md) - MariaDB 참조
  * 방화벽 재시작
  ```
  $ sudo firewall-cmd --reload
  ```
  * 방화벽 확인
  ```
  $ sudo iptables -L --line-numbers
  ```
6. 노드 추가
  * 1번 노드에서 설정 했던 server.cnf 의 [galera] 섹션의 내용 복사하여 추가노드에 붙여넣기
  ```
  $ sudo vi /etc/my.cnf.d/server.cnf
  [galera]
  ..
  ```
  * wsrep_node_address 와 wsrep_node_name 값 추가할 노드 정보로 수정
  ```
  wsrep_node_address="10.6.xx.xx"
  wsrep_node_name="aehamdbp02"
  ```
  * mysqld 섹션에 로그경로 추가
  ```
  [mysqld]
  log_error=/var/log/mariadb.log
  ```
  * 더미로그 생성
  ```
  $ sudo touch /var/log/mariadb.log
  $ sudo chown mysql:mysql /var/log/mariadb.log
  ```
  * 방화벽 확인
  * 서비스 시작
  ```
  $ sudo systemctl start mariadb
  ```
  

