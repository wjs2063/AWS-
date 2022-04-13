# DEVOPS-AWS-
AWS 서버 개설 및 DEVOPS 공부


# 1.운영서버 아키텍처의 이해 
 클라이언트 -> [ APP + DATABASE ] 구조
 
 ### 장점: 단순한 구성이기 떄문에 테스트 서버나 간단한 애플리케이션 구축할떄 용이
 ### 단점: APP과 DATABSE 가 같은 자원을 공유하기때문에 둘중하나라도 자원을 모두 사용하게된다면 전체서비스가 죽게된다. 서버자원효율성과 보안성이 떨어짐
 ### 로드밸런서 -> 클라이언트와 서버가 직접 통신하는 대신 로드밸런서라는 서버와 통신하고 그 뒤에 여러대의 애플리케이션 서버를 두는 방식
클라이언트는 로드밸런서하고만 통신을 하게되며 로드밸런서는 클라이언트에게 받은 요청사항을 서버들에게 나눠주게 된다  뒤에있는 서버들이 요청을 처리하면 로드밸런서를통해 클라이언트에게 전달 ( OSI 7계층에서 L4 SWITCH라고 얘기하는것이 로드밸런서의 역할을 하게됨)


## AWS EC2를 이용한 서버 인스턴스 생성과 관리

EC2 를 생성하려면 꼭 알아야하는개념

1.AMI( AMAZON MACHINE IMAGE) + SECURITY GROUP(보안그룹) + KEY PAIR(키페어)

AMI -> EC2 인스턴스의 기반이 되는 이미지

보안그룹 -> IP 와 포트번호를 이용해 정의해두는 서버접속규칙  특정 IP 와 포트에 대해서만 서버접속허용

KEY PAIR -> 서버에 접속하기위한 열쇠, 공개 키 암호화 기법으로 서버에는 공개키 사용자는 개인키를 들고 접속 


## [실습]  AWS EC2 인스턴스 생성해보기

1. AWS 로그인 
2. 지역설정 [아시아 태평양(서울)] 선택
3. EC2 검색후 선택
4. 인스턴스 시작 버튼 클릭
5. AMAZON LINUX 2 SSD VOLUME TYPE 선택 (64bit)
6. 인스턴스 유형중 프리티어인 t2.micro 선택 ( 돈이 꽤나 여유롭다면 다른것을 선택해도됨..)
7. 인스턴스 세부정보 구성 ( 별다른게없으니 그대로 선택)
8. 스토리지 추가 ( 별다른게없으니 그대로 선택)
9. 태그지정 필수  KEY-VALUE 형식 (dictionary 구조)  이므로 NAME - 하고싶은이름 으로 해주자
10. 보안그룹 구성 ( 보안그룹이름 -설명 ) ( ssh - security for ssh access ) 
11. 인스턴스 시작 검토 ( 시작버튼누르기)
12. 기존 키페어 선택 및 새 키페어 생성(  키페어 생성시 잃어버리거나 삭제되지않도록 주의해야함 )

## 생성된 EC2 인스턴스로의 접속

SSH 프로토콜을 이용해 접속가능  ( 윈도우에서는 putty 앱을 이용해 접속  macOS 는 ssh 명령어를통해 접속)
HTTP,HTTPS 도 접근가능하도록 보안그룹 추가하기 (이름:web, 설명:security group for web)  HTTP - 80 , HTTPS - 443 )
인스턴스- 인스턴스메뉴 선택 ->보안그룹변경 -> web 보안그룹 체크 ssh 체크 

서버접속방법  putty 실행후

Session 에  HOSTNAME 부분에 ec2-user@ [내 AWS 인스턴스의 퍼블릭 IPV4 DNS  부분 복사] 한다

그리고 왼쪽에 CONNECTION 에  SSH  AUTH 클릭후 PRIVATE KEY 를 통해 로그인하면된다 ( 이전에 받았던 KEY PAIR파일인 pem 파일을 ppk 파일로 변환해두어야함, putty gen 을이용해서 생성)


[실습 코드]
1.curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
2. . ~/.nvm/nvm.sh
3. nvm install 10.13.0
4. node -e "console.log('Running Node.js' + process.version)"

[GIT 으로 EC2 인스턴스에 코드 배포]

1.sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
2.cd/var
3. sudo mkdir www
4. sudo chown ec2-user www
5. cd /var/www
6. git clone https://github.com/deopard/aws-exercise-a.git
7. cd aws-exercise-a
8. npm install

[실습코드] nginx,phusion passenger 설치 및 서비스
1. 서버 로그인
2. cd /var/www
3. wget http://s3.amazonaws.com/phusion-passenger/releases/passenger-5.3.7.tar.gz  ( 책에는 5.3.6 인데  책대로하면 버젼오류가뜨므로 5.3.7로변경... 1시간잡아먹음) 
4. sudo mkdir /var/passenger
5. sudo chown ec2-user /var/passenger
6. tar -xzvf passenger-5.3.7.tar.gz -C /var/passenger
7. gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB  ( http://rvm.io/rvm/install 사이트참조) --- 간혹 http 와 fetch 할수없다는 에러창이 뜨면 curl -sSL https://rvm.io/pkuczynski.asc | gpg --import - 로 해보자
8. curl -sSL https://get.rvm.io | bash -s stable
9. source /home/ec2-user/.rvm/scripts/rvm
10. rvm reload
11. rvm requirements run
12. rvm install 2.4.3
13. echo export PATH=/var/passenger/passenger-5.3.7/bin:$PATH >> ~/.bash_profile
14. source ~/.bash_profile
15. passenger-install-nginx-module ( SPACE 로 체크 및 체크해제 가능, NODE.JS 만 설치  가상메모리부족 경고창 뜬다. 경고메세지대로 가상메모리 늘려주기)
16. sudo dd if=/dev/zero of=/swap bs=1M count=1024
  sudo mkswap /swap
  sudo swapon /swap
 16. passenger-install-nginx-module ( 1번 누르고 엔터)
 17.  
  export ORIG_PATH="$PATH"
  rvmsudo -E /bin/bash
  export PATH="$ORIG_PATH"
  export rvmsudo_secure_path=1
  /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby /var/passenger/passenger-5.3.7/bin/passenger-install-nginx-module
 18. exit
 19. sudo vi /opt/nginx/conf/nginx.conf ( nginx 설정변경을위한  파일열기)
 20. 
worker_process 1;
events{
   worker_connections 1024;
}

http{
server_names_hash_bucket_size 256;
passenger_root /var/passenger/passenger-5.3.7;
passenger_ruby /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby;
include mime.types;
default_type application/octet-stream;
sendfile on;
keepalive_timeout 65;

server{
listen 80;
server_name <EC2 public address 입력>
root /var/www/aws-exercise-a/public;

passenger_enabled on;
passenger_app_type node;
passenger_startup_file /var/www/aws-exercise-a/app.js;

}
}



21. sudo /opt/nginx/sbin/nginx
22. 브라우저로 ec2 public 주소로 접속하면 해당문구가 뜬다
23. cd /etc/init.d
24. sudo vi nginx
25.  https://gist.githubusercontent.com/deopard/fe2b37c499f3e3225a99f8bc45d5be07/raw/339c10687700e8bc5695297af548ae1356eb2592/nginx 해당 파일을 복사붙혀넣기  
26.  sudo chmod 755 nginx  
27.  서비스 조작명령어   
28.  sudo service nginx stop  
29.  sudo service nginx start  
30.  sudo service nginx restart  
31.  sudo service nginx status  
32.  시스템 시작시 자동시작 서비스 등록  
33.  sudo chkconfig --add nginx  
34.  sudo ntsysv  
35.  nginx 체크 ( space 와 버튼이동(tab))  
 ## [실습] 하나의 서버에서 두개의 애플리케이션 서비스하기
 
 1. cd /var/www
 2. git clone https://github.com/deopard/aws-exercise-b.git
 3. cd aws-exercise-b
 4. npm install
 5. sudo vi /opt/nginx/conf/nginx.conf
 6. 기존 서버파일 
worker_process 1;
events{
   worker_connections 1024;
}

http{
server_names_hash_bucket_size 256;  
passenger_root /var/passenger/passenger-5.3.7;  
passenger_ruby /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby;  
include mime.types;  
default_type application/octet-stream;  
sendfile on;   
keepalive_timeout 65;  

server{  
listen 80;  
server_name <EC2 public address 입력>    ## 중요!!
root /var/www/aws-exercise-a/public;  

passenger_enabled on;  
passenger_app_type node;  
passenger_startup_file /var/www/aws-exercise-a/app.js;  

}
server{  
listen 80;  
server_name <EC2 public DNS 입력>    ## 중요!!
root /var/www/aws-exercise-b/public;  

passenger_enabled on;  
passenger_app_type node;  
passenger_startup_file /var/www/aws-exercise-b/app.js;  

}


}
7. sudo service nginx restart
8. 접속하기!

## [실습] Auto Scaling
1. EC2 서비스의 instance 가 Stopped 상태로 만들기(스냅숏을 생성하기위함)
2. 인스턴스 이미지생성 ( 우클릭후 이미지생성)
3. 이미지 이름 지정후 이미지생성
4. EC2메뉴의 이미지 ->AMI 를 클릭해서들어가면 스냅숏한 이미지가 저장되어있음 
5. 생성 이미지의 상태가 Available 인지 확인후
6. 생성된 AMI ID 복사
7. 인스턴스 -> Launch Templates 메뉴 선택
8. 시작템플릿 생성 클릭 ( 서버 사양 및 보안그룹설정)
9. 시작템플릿 이름, 버전설명 , 스냅숏한 AMI, 인스턴스유형,키페어이름,네트워크,보안그룹설정하기 
10. 생성후 왼쪽의 AUto Scaling 그룹생ㅅ엉
11. Auto Scaling 그룹이름-EXERCISE-GROUP, VPC -subnet 은 northeast-2a, 2c 두개 추가
12. Auto SCaling 그룹 생성 , 그룹용량 최소 1 최대 2로 설정, 평균 CPU 사용률 80%로 설정

--결과 ![스크린샷(25)](https://user-images.githubusercontent.com/76778082/163183770-2ece104a-06b5-4207-9eb6-943f0c212575.png)


## [실습] Auto Scaling을 통한 인스턴스 자동 추가,제거
### 앞선 실습에서 CPU 사용률이 80% 넘으면 인스턴스 하나 추가하고 이하로 떨어지면 인스턴스를 줄이는 조정정책을 추가했다. 제대로 동작하는지 확인
1. AutoScaling Group에서 생성한 그룹 클릭후 인스턴스 관리 클릭 , 인스턴스가 In Service 인지 체크
2. 해당 인스턴스의 CPU 사용량을 늘려주기위해 해당 인스턴스의 Public IP 주소를 확인후 로그인
3. sudo amazon-linux-extras install epel -y ( 아마존 리눅스에서 stress 설치오류 )
4. sudo yum install stress -y
5. stress --cpu 1 --timeout 600  
6. 5~10분 기다리면 인스턴스가 자동으로 한대 더 추가되는것을 볼수있다
7. 실습을 모두 끝내고 Auto Scaling 인스턴스들을 모두 종료하고싶다면 그룹의 목표용량을 0으로설정하면 된다!
