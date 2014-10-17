# linux-nginx-flask-uwsgi-mysql skeleton

본 repo는 [원본 뼈대](https://github.com/Hogoonn/AWS-nginx-flask-uwsgi-mysql-skeleton)를 토대로 작성되었습니다.


## AWS EC2 setting

AWS flask application 세팅하기
(아래의 모든 'your_app_folder'는 자신이 사용하는 프로젝트 폴더의 이름으로 변경합니다.)

1. 패키지 설치
    ```
    sudo yum install python26-devel nginx gcc gcc-c++ python-setuptools
    sudo easy_install pip
    sudo pip install uwsgi virtualenv
    ```

2. 작업폴더 생성, 세팅
    ```
    mkdir your_app_folder
    cd your_app_folder
    virtualenv venv
    . venv/bin/activate
    pip install -flask flask-sqlalchemy flask-migrate python-gflags flask-oauth flask-wtf
    ```

3. Nginx 세팅

sudo vi /etc/nginx/nginx.conf

    ```
    user root;
    server {
        listen 80
        server_name localhost;
        root /home/ec2-user;
        location / {
            try_files $uri @path;
        }
        location @path {
            include uwsgi_params;
            uwsgi_pass unix:/home/ec2-user/your_app_folder/uwsgi.sock;
        }
    }
    ```

4. uwsgi 세팅, .ini 파일

(vi uwsgi.ini로 화일 수정하고 저장)

    ```
    [uwsgi]
    base = /home/ec2-user/your_app_folder
    chdir = %(base)
    module = app:app
    venv = %(base)/venv
    socket = %(base)/uwsgi.sock
    master = True
    process = 2
    chmod-socket = 644
    daemonize = %(base)/logs/log.log
    pidfile = %(base)/mypid.pid
    home = %(base)/venv
    ```

5. Nginx 재시작

    ```
    sudo service nginx stop
    sudo service nginx start
    ```

6. 로그화일 생성 

 - 어플리케이션 다시 시작하기

    ```
    uwsgi --ini uwsgi.ini
    ```
 - 로그 보기

    ```
    vi logs/log.log
    ```

 - 마지막 10줄 보기

    ```
    tail logs/log.log
    ```
 - 프로세스 id 생성 확인

    ```
    vi mypid.pid
    ```

7. 빠른 어플리케이션 다시 시작

 - 쉘 스크립트 화일 실행

    ```
    sh restart.sh 
    ```

8. ls로 생성된 화일 확인

생성된 파일은 다음과 같아야 합니다.

    ```
    logs mypid.pid restart.sh uwsgi.ini uwsgi.sock venv
    ```

9. mysql 설치

    ```
    sudo yum install mysql
    sudo yum install mysql-server
    sudo yum install mysql-devel
    sudo chgrp -R mysql /var/lib/mysql
    sudo chmod -R 770 /var/lib/mysql
    sudo service mysqld start
    ```

 - 설치시 입력한 비밀번호를 'rootpassword' 대신 입력합니다.

    ```
    /usr/bin/mysqladmin -u root password rootpassword
    ```
 - 앞으로 mysql에 접속할 유저를 생성합니다.

 생성할 유저명을 'myuser' 대신 입력하고, 사용할 비밀번호를 'yourpasswordhere' 대신 입력합니다.

    ```
    mysql> CREATE USER 'myuser'@'localhost' IDENTIFIED BY 
        -> 'yourpasswordhere';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'localhost'
        -> WITH GRANT OPTION;
    mysql> CREATE USER 'myuser'@'%' IDENTIFIED BY 'yourpasswordhere';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%'
        -> WITH GRANT OPTION;
    ```

 - app/settings.py 파일 수정

 `SQLALCHEMY_DATABASE_URI` 항목의 username과 password를 생성한 계정의 유저명과 비밀번호로 수정합니다.

 또, 'DBname'을 생성하고자 하는 DB의 이름으로 변경합니다.

```
class Config(object):
    SECRET_KEY = "aaaaaaa"
    debug = False


class Production(Config):
    debug = True
    CSRF_ENABLED = False
    SQLALCHEMY_DATABASE_URI = "mysql://username:password@127.0.0.1/DBname"
    migration_directory = "migrations"
```

 - db 생성

    ```
    mysql -u root -p
    show databases;
    create database DBname;
    ```

 - MySQL-python 설치

    ```
    sudo yum install MySQL-python
    ```

이후, virtualenv 상태에서,

    ```
    pip install MySQL-python
    ```
     

