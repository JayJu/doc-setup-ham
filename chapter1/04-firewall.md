# 방화벽 정책

1. 기본명령
  * 정책 확인
  ```
  # iptables -L --line-numbers
  ```
  * Zone 정보 확인
  ```
  #  firewall-cmd --list-all-zones 
  ```
  * Zone 별 목록보기
  ```
  # firewall-cmd --zone=public --list-all
  ```
  * 등록서비스 확인
  ```
  # firewall-cmd --list-services
  ```
  * 등록 포트확인
  ```
  # firewall-cmd --list-ports
  ```
  * 서비스 적용
  ```
  # firewall-cmd --reload
  ```

2. 미디어 서버
  * Kurento Media Server Port : 8888
  ```
  # firewall-cmd --zone=public --permanent --add-port=8888/tcp
  ```
  * Streaming Port: udp all
  ```
  # firewall-cmd --zone=public --permanent --add-port=49152-65535/udp
  ```
     
3. 어플리케이션 서버
  * Jenkins
  ```
  # firewall-cmd --zone=public --permanent --add-port=7000/tcp
  ```
  * SonarQube
  ```
  # firewall-cmd --zone=public --permanent --add-port=8000/tcp
  ```
  * Webpack
  ```
  # firewall-cmd --zone=public --permanent --add-port=9000/tcp
  ```
  * HVCS - Https
  ```
  # firewall-cmd --zone=public --permanent --add-port=8443/tcp
  ```
  * Samba
  ```
  # firewall-cmd --add-service=samba --permanent
  ```
  
4. DB 서버
  * Gitlab
  ```
  # firewall-cmd --zone=public --permanent --add-port=80/tcp
  ```
  * Mariadb
  ```
  # firewall-cmd --zone=public --add-service=mysql --permanent
  # firewall-cmd --zone=public --permanent --add-port=3306/tcp
  # firewall-cmd --zone=public --add-port=4567/tcp --permanent
  # firewall-cmd --zone=public --add-port=4567/udp --permanent
  ```



