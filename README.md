# vault-transit-app

## Vault

### Policy

1. policy 작성

   * `transit_admin` : transit secret에 모든 권한 

   ```
   vi transit_admin.hcl
   ```

   ```go
   path "sys/mounts/transit" {
     capabilities = [ "create", "read", "update", "delete", "list" ]
   }
   
   path "sys/mounts" {
     capabilities = [ "read" ]
   }
   
   path "transit/*" {
      capabilities = [ "create", "read", "update", "delete", "list"  ]
   }
   ```

   

   * `transit_encrypt` : 암호화 권한만 부여 (복호화 불가)

   ```
   vi transit_encrypt.hcl
   ```

   ```go
   path "sys/mounts/transit" {
     capabilities = [ "create", "read", "update", "delete", "list" ]
   }
   
   path "sys/mounts" {
     capabilities = [ "read" ]
   }
   
   path "transit/encrypt/ds-poc" {
      capabilities = [ "update" ]
   }
   ```

   

   * `transit_decrypt` : 복호화 권한만 부여 (암호화 불가)

   ```
   vi transit_decrpyt.hcl
   ```

   ```go
   path "sys/mounts/transit" {
     capabilities = [ "create", "read", "update", "delete", "list" ]
   }
   
   path "sys/mounts" {
     capabilities = [ "read" ]
   }
   
   path "transit/decrypt/ds-poc" {
      capabilities = [ "update" ]
   }
   ```

   

2. policy 등록

```
vault policy write transit_admin transit_admin.hcl
vault policy write transit_enc transit_encrypt.hcl
vault policy write transit_dec transit_decrypt.hcl
```



3. token 생성

```
vault token create -policy=transit_admin
```

```
vault token create -policy=transit_enc
```

```
vault token create -policy=transit_dec
```



**admin은 transit 관련 명령이 모두 가능하지만 transit_enc는 decrypt 불가, transt_dec는 encrypt가 불가능하다 ** 





## java 17

```
yum install jdk-17_linux-x64_bin.rpm
```

```
java -version
java version "17.0.8" 2023-07-18 LTS
Java(TM) SE Runtime Environment (build 17.0.8+9-LTS-211)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.8+9-LTS-211, mixed mode, sharing)
```



## 리소스 사용량 모니터링 설치 (visualvm)

```
brew install --cask visualvm
```



## gradle

```
sudo mkdir -p /opt/gradle
sudo unzip -d /opt/gradle gradle-8.3-bin.zip
echo 'export PATH=$PATH:/opt/gradle/gradle-8.3/bin' >> ~/.zshrc
source ~/.zshrc

gradle -version
```





## mysql

* 설치후 서비스 재시작

```
sudo systemctl status mysqld 
sudo systemctl start mysqld
```



* 접속

```
mysql -u root -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

&rightarrow; **초기에 접속 시 root 계정 패스워드에 기본설정이 되어있어 변경해야한다.**



CentOS 7



1. root 패스워드 확인

```
sudo cat /var/log/mysqld.log | grep 'temporary password'
```

2. 확인한 패스워드로 접속 

```
mysql -u root -p
```

3. root 패스워드 변경

```
alter user 'root'@'localhost' identified by '<변경할 패스워드>';
```

* 패스워드는 반드시 특수문자, 대문자, 소문자, 숫자 포함 8자 이상이어야 한다.





### mysql 설정

```
mysql -h <MySQL Host> -u <user명> -p<password>
```

```mysql
CREATE DATABASE VaultData;
```

```mysql
CREATE USER app@'%' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES
    ON VaultData.*
    TO app@'%';

FLUSH PRIVILEGES;
```

```mysql
USE VaultData;

Database changed

create table vault_data (
  id int unsigned auto_increment not null,
  data varchar(200) not null,
  date_created timestamp default now(),
  primary key (id)
);
```





## App 

* App Build 전 설정

```
export MYSQL_HOST=<MySQL 주소>
export MYSQL_PORT=<MySQL port , default=3306>
export MYSQL_DB_NAME=VaultData
export MYSQL_USERNAME=<user명>
export MYSQL_USERPW=<password 입력>
export VAULT_HOST=<Vault LB 주소>
export VAULT_PORT=8200
export VAULT_SCHEME=http
export VAULT_TOKEN=<VAULT TOKEN>
export VAULT_TRANSIT_KEY_NAME=<key-name>

export S3_ENABLE=<true/false>
export S3_BUCKET_NAME=<S3 Bucket 이름>
export AWS_ACCESS_KEY_ID=<AWS_ACCESS_key>
export AWS_SECRET_ACCESS_KEY=<AWS_SECRET_ACCESS_key>
export AWS_DEFAULT_REGION=ap-northeast-2
```



* App Build

```
cd java-app
gradle build
```



* App 실행

```
cd build/libs
```

```
java -jar demo-0.2.1.jar
```



> VisualVM
>
> ```
> java -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.port=<visualvm에게 열어줄 port> -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Djava.rmi.server.hostname=<app서버 ip> -Dcom.sun.management.jmxremote.rmi.port=8800 -Xms1024m -Xmx1024m -jar ./build/libs/demo-0.2.1.jar
> ```
>
> * `-Xms1024m -Xmx1024m` : Heap memory 설정
> * `-Dcom.sun.management.jmxremote.port=8800` : visualvm 이 모니터링할 port 
>





## k6 ( java run 모니터링 )

### 설치

```
sudo yum install dnf

sudo dnf install https://dl.k6.io/rpm/repo.rpm
sudo dnf install k6 
```

* 필요시 `--nogpgcheck` 옵션추가



### k6 실행

```
k6 run <js파일>
```



### k6 옵션

링크: https://k6.io/docs/using-k6/k6-options/reference/



* `vus ` : 병렬로 처리할 유저 수
* `duration`: 처리할 시간 
* `iterations` : 총 요청수  









