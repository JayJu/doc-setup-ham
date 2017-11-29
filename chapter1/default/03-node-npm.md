# Node.js 와 npm 설치 및 환경설정
* 참고자료
    * [패키지 매니저로 Node.js 설치하기](https://nodejs.org/ko/download/package-manager/#enterprise-linux-fedora)

1. node 설치 (root로 진행)
    ```
    # curl --silent --location https://rpm.nodesource.com/setup_6.x | bash -
    $ yum -y install nodejs
    $ node -v
    ```