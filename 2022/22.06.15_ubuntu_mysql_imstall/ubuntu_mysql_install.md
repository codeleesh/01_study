# Ubuntu Mysql Install



## Ubuntu 환경



```bash
lovethefeel@lovethefeel-H310H5-M7:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.3 LTS
Release:	20.04
Codename:	focal
```





## Mysql Install



```bash
sudo apt install mysql-server
```



```bash
...
...

done!
update-alternatives: using /var/lib/mecab/dic/ipadic-utf8 to provide /var/lib/mecab/dic/debian (mecab-dictionary) in auto mode
mysql-server-8.0 (8.0.29-0ubuntu0.20.04.3) 설정하는 중입니다 ...
update-alternatives: using /etc/mysql/mysql.cnf to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Renaming removed key_buffer and myisam-recover options (if present)
mysqld will log errors to /var/log/mysql/error.log
mysqld is running as pid 79519
Created symlink /etc/systemd/system/multi-user.target.wants/mysql.service → /lib/systemd/system/mysql.service.
mysql-server (8.0.29-0ubuntu0.20.04.3) 설정하는 중입니다 ...
Processing triggers for systemd (245.4-4ubuntu3.15) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.7) ...
```



```bash
sudo service mysql status
```



```bash
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-06-15 14:17:45 KST; 1min 10s ago
   Main PID: 79730 (mysqld)
     Status: "Server is operational"
      Tasks: 37 (limit: 9287)
     Memory: 358.9M
     CGroup: /system.slice/mysql.service
             └─79730 /usr/sbin/mysqld

 6월 15 14:17:44 lovethefeel-H310H5-M7 systemd[1]: Starting MySQL Community Server...
 6월 15 14:17:45 lovethefeel-H310H5-M7 systemd[1]: Started MySQL Community Server.
```



```bash
sudo ss -tap | grep mysql
```



```bash
LISTEN   0        70             127.0.0.1:33060                0.0.0.0:*        users:(("mysqld",pid=79730,fd=22))
LISTEN   0        151            127.0.0.1:mysql                0.0.0.0:*        users:(("mysqld",pid=79730,fd=24))
```



### MySQL 서버의 시작과 종료



```
sudo ufw allow mysql
```



```
sudo ufw reload
```



```
sudo systemctl restart ufw
```





#### 서버 시작

```
sudo systemctl start mysql
```



#### Ubuntu 서버 재시작시 MySQL 자동 재시작

```
sudo systemctl enable mysql
```



```
sudo /usr/bin/mysql -u root -p
```



```
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.29-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```



### 데이터베이스 생성



```
mysql> create database TESTDB
```





### 사용자 계정 생성

#### 사용 중인 사용자 계정 확인하기

```sql
-- root 계정의 데이터베이스 중 'mysql' 이라는 데이터베이스 선택하기
mysql> use mysql;

-- 'user' 이라는 테이블의 정보에서 사용자 계정 확인하기
mysql> select host, user from user;
```



#### 사용자 계정 생성하기

##### 아이디만 생성

```sql
mysql> create user 계정ID; 
```



##### 아이디 + 비밀번호 + host 생성

```sql
mysql> create user '계정'@localhost identified by '비밀번호';
```



##### localhost만 추가된 계정에 외부 host 접근 권한 추가

```sql
mysql> create user '계정'@'%' identified by '비밀번호';
```



## 참고



https://ubuntu.com/server/docs/databases-mysql