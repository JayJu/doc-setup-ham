# Jenkins 설치 및 환경구성

  * 설치환경
    * RHEL 7.4
    * 참고자료: [How To Install Jenkins on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-16-04)
   
---
1. JDK 설치
  * [JDK설치 및 환경설정](/chapter1/default/01-jdk.md)
  
2. Gradle 설치
  * [Gradle 설치 및 환경설정](/chapter1/default/02-gradle.md)

3. 사용자 추가 및 권한 설정
  * 사용자 추가
  ```
  # useradd jenkins -m -d /home/jenkins
  # passwd jenkins
  ```
  * sudoer 등록
  ```
  # visudo
  ```
  * gradle profile 추가
  ```
  $ su - jenkins
  $ vi ./.bashrc
  ## gradle
  PATH="/opt/gradle/gradle-4.0.2/bin:$PATH"
  $ source ./.bashrc
  ```

4. Jenkins 설치\(jenkins로 로그인 후 진행\)
  * 레파지토리 등록
  ```
  $ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
  ```
  * Key Import
  ```
  $ sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
  ```
  * Jenkins 설치
  ```
  $ sudo yum install -y jenkins
  ```

5. Jenkins 시작
   * 포트변경
  ```
  $ sudo vi /etc/sysconfig/jenkins
  // port 부분을 변경.
  HTTP_PORT=8080 -> HTTP_PORT=7000
  ```
  * systemctl을 사용하여 Jenkins 시작
  ```
  $ sudo systemctl start jenkins
  ```
  * status 확인
  ```
  $ sudo systemctl status jenkins
  jenkins.service - LSB: Start Jenkins at boot time
  Loaded: loaded (/etc/init.d/jenkins; bad; vendor preset: enabled)
  Active: active (exited) since Fri 2017-07-07 09:47:29 KST; 2min 1s ago
  Docs: man:systemd-sysv-generator(8)
  ```

6. 방화벽 오픈
  * [방화벽포트오픈](04-firewall.md) - Jenkins 참조
  * 방화벽 재시작
  ```
  $ sudo firewall-cmd --reload
  ```
  * 방화벽 확인
  ```
  $ sudo iptables -L --line-numbers
  ```

7. Jenkins 기본 설정
  * Jenkins 접속
  ```
  http://ip_address_or_domain_name:7000
  ```
  * 관리자 비번 입력 후 Unlock Jenkins
  ```
  $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
  ![](/img/ch1/sub1/1-1-1.jpg)

  * "Install suggested plusins" 선택  
  ![](/img/ch1/sub1/1-1-2.jpg)

  * 관리자 계정 생성 - 정보 입력 후 "Save and Finish"  
  ![](/img/ch1/sub1/1-1-4.jpg)

  * "Start using Jenkins"  
  ![](/img/ch1/sub1/1-1-5.jpg)![](/img/ch1/sub1/1-1-6.jpg)
---

# Jenkins - Gitlab 연동
* 참고문서
  * [Jenkins CI service](https://docs.gitlab.com/ee/integration/jenkins.html#configure-gitlab-users)
  * [gitlab-plusin intruction](https://github.com/jenkinsci/gitlab-plugin#using-it-with-a-job)
  * [Gitlab Continuous Integration on Jenkins](http://blog.ljdelight.com/gitlab-continuous-integration-on-jenkins/)
  * [Introduction to Continuous Integration with JHipster](http://blog.ippon.tech/continuous-integration-with-jhipster/)
  
1. GitLab 설정
  * 사용자 계정과 access token 생성
  ![](/img/ch1/sub1/1-1-7.png)
  
2. Jenkins 설정
  * Global Tool Configuration 설정
  ![](/img/ch1/sub1/1-1-14.png)

  * 플러그인 설치
    * [Gitlab](https://wiki.jenkins.io/display/JENKINS/GitLab+Plugin)
    * [Javadoc](https://wiki.jenkins.io/display/JENKINS/Javadoc+Plugin)
    * [Pipeline-jenkins2.x에 포함됨](https://wiki.jenkins.io/display/JENKINS/Pipeline+Plugin)
  ![](/img/ch1/sub1/1-1-8.png)

  * GitLab 접근을 위한 credential 생성
    Jenkins CI 가 build status를 GitLab API로 보내기 위한 설정
    GibLab에서 생성한 사용자 access token을 등록
    ![](/img/ch1/sub1/1-1-9.png)

  * GitLab connection 설정
  ![](/img/ch1/sub1/1-1-10.png)
  
3. Jenkins Build Job 생성
  GitLab의 mgerge request 를 처리할 job 생성: "ham-build"
  * 폴더 생성
  ![](/img/ch1/sub1/1-1-11.png)

  * 폴더 하위에 pipeline job 생성
  ![](/img/ch1/sub1/1-1-12.png)
  
  * build triggers 섹션
  * "Build when a change is pushed to GitLab CI.." 체크
  * "Advenced.." 에 key generate..
  * 위 URL과 generated 된 Secret token으로  GitLab에서 Jenkins Webhook 생성해야함
  ![](/img/ch1/sub1/1-1-13.png)

  * Pipeline 섹션
  ![](/img/ch1/sub1/1-1-17.png)
  * Definition 에 "Pipeline script fro SCM" 선택
  * SCM에 git 선택
  * Repository URL에 git project 주소 입력
  > Credential 에 GitLab에 API Token 생성 하여 입력 해야 되는데 add 가 안되는 
  > 현상이 있음. 임시로 credential을 gitlab user/password form으로 생성하여 임시조치
  > ![](/img/ch1/sub1/1-1-15.png)
  * Branch에 master -> release로 변경
  * Script Path: Jenkinsfile -> 소스에 Jenkinsfile 을 생성해여 소스레벨에서 배포스크립트를 관리하는 방식으로 진행
  ```
    #!/usr/bin/env groovy

    node {
        stage('checkout') {
            checkout scm
        }

        gitlabCommitStatus('build') {
            stage('check java') {
                sh "java -version"
            }

            stage('clean') {
                sh "chmod +x gradlew"
                sh "./gradlew clean --no-daemon"
            }

            stage('npm install') {
                sh "./gradlew npmInstall -PnodeInstall --no-daemon"
            }

            stage('backend tests') {
                try {
                //    sh "./gradlew test -PnodeInstall --no-daemon"
                } catch(err) {
                    throw err
                } finally {
                //    junit '**/build/**/TEST-*.xml'
                }
            }

            stage('frontend tests') {
                try {
                //    sh "./gradlew npm_test -PnodeInstall --no-daemon"
                } catch(err) {
                    throw err
                } finally {
                //    junit '**/build/test-results/karma/TESTS-*.xml'
                }
            }

            stage('packaging') {
            //    sh "./gradlew bootRepackage -x test -Pprod -PnodeInstall --no-daemon"
                sh "./gradlew bootRepackage -Pdev -PnodeInstall --no-daemon"
                archiveArtifacts artifacts: '**/build/libs/*.war', fingerprint: true
            }
        }
    }
  ```
  
4. GibLab Webhook 생성 및 테스트
  * GitLab Project > Setting > Integration
  * Pipeline Job 생성 단계의 URL과 generated Secret token 복사/붙여넣기
  * 트리거에 Merge Request events 체크 (Merge Request시에만 Jenkins build)
  ![](/img/ch1/sub1/1-1-16.png)
  
5. 연동테스트
  * Release용 브랜치 생성
  ![](/img/ch1/sub1/1-1-18.png)
  
  * Master 브랜치에 변경배용 push 후 master -> release Merge Request 생성
  ![](/img/ch1/sub1/1-1-19.png)
  ![](/img/ch1/sub1/1-1-20.png)
  
  * Jenkins에서 Pipeline Job 수행 되는지 확인
  ![](/img/ch1/sub1/1-1-21.png)

---

# Jenkins Slave 구성
* 설치환경
* RHEL 7.4
* 참고자료: [Add linux slave node in the Jenkins](https://mohitgoyal.co/2017/02/14/add-linux-slave-node-in-the-jenkins/)
---
1. JDK 설치
  * [JDK설치 및 환경설정](/chapter1/default/01-jdk.md)
2. Gradle 설치
  * [Gradle 설치 및 환경설정](/chapter1/default/02-gradle.md)

3. 사용자 추가 및 권한 설정
  * 사용자 추가
  ```
  # useradd jenkins -m -d /home/jenkins
  # passwd jenkins
  ```
  * sudoer 등록
  ```
  # visudo
  ```
  * gradle profile 추가
  ```
  $ su - jenkins
  $ vi ./.bashrc
  ## gradle
  PATH="/opt/gradle/gradle-4.0.2/bin:$PATH"
  $ source ./.bashrc
  ```
4. SSH Key 생성(마스터서버에 jenkins 계정으로 수행)
  * SSH Key 생성
  ```
  $ cd
  $ ssh-keygen -t rsa
  $ (passphrase는 입력 없이 엔터)
  $ ls .ssh
  ```
5. Public Key를 Slave 노드에 복사
  * ssh key copy
  ```
  $ ssh-copy-id jenkins@app서버ip
  ```
6. Jenkins Master 관리화면에서 Slave 노드 추가
  * Jenkins Master 노드에 관리자로 로그인
  * Jenkins > Manage Jenkins > Manage Nodes 클릭
  
    ![](/img/ch1/sub1/1-1-23.png)
  * New Node 클릭
    
    ![](/img/ch1/sub1/1-1-24.png)


  