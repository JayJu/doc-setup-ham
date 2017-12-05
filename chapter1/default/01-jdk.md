# JDK 설치 및 환경설정
1. JDK 설치(version 8)
  ```
  $ sudo yum install java-1.8.0-openjdk-devel
  ```
2. JAVA_HOME 환경변수 설정
  * 환경파일에 JAVA_HOME 추가 (JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64")
  
    ```
    $ sudo vi ~/.bashrc
    ## Java Home
    export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-2.b15.el7_3.x86_64"
    
    $ source ~/.bashrc

    $ echo $JAVA_HOME
    /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-2.b15.el7_3.x86_64
    ```
3. Default JDK 설정 (2개 이상 설치 된 경우)
  * 설치 목록 확인 및 변경
  ```
  $ sudo yum install alternatives
  ```
  * default 확인
  ```
  $ sudo /usr/sbin/alternatives --config java
      Selection Path Priority Status
    ------------------------------------------------------------
    0 /usr/lib/jvm/java-8-oracle/jre/bin/java 1081 auto mode
    1 /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 1081 manual mode
    * 2 /usr/lib/jvm/java-8-oracle/jre/bin/java 1081 manual mode
  ```
