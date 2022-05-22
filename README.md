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



## [실습] AWS Elastic Load Balancing 을 이용한 서버 트래픽 분산 관리
ELB: 로드밸런서의 역할을 하는 AWS 서비스
AWS 같은 클라우드 서비스를 사용하지않으면 L4 스위치같은 장비를 직점구매후 관리해야함.

#### ELB 사용 요금 : https://aws.amazon.com/ko/elasticloadbalancing/pricing/?nc2=type_a 

대상그룹: 로드밸런서가 요청을 전달할 서버들을 묶어둔 개념적그룹

Load Balancer 의 상태검사: 로드밸런서는 관리하는 서버 중 정상적으로 동작하고있는 서버에만 요청을 전달해줌

[EXAMPLE] 서버에서 GET/HEALTH 로 등록을 해두면 로드밸런서에서는 주기적으로 서버들에게 GET/health 요청을 보내봄--? 만약 상태코드 200이아닌 다른 상태코드로 응답하거나 응답을 제시간에 주지못하면 비정상상태로 판단 정상상태로 응답하기전까지 클라이언트에 요청 전달 x


[실습] Auto Scaling 그룹,대상그룹,로드밸런서 구성 
1. 로드밸런싱-로드밸런서-로드밸런서생성 클릭
2. HTTP 클릭
3.  이름:exercise-lb , 로드밸런서 프로토콜 HTTP port:80  가용영역 2a-2c 클릭 
4.  target-group 생성해서 Autoscaling 그룹으로 지정한다  상태검사 경로는 /health
5.  로드밸런싱-로드밸런서 exercise-lb 로드밸런서 클릭 DNS 주소로 주소창로그인


## 장애조치 아키텍쳐 구성

장애조치란 장애극복기능으로 시스템의 일부 서버에 장애가 발생했을때 전체시스템이 죽는것을 방지 
예비 시스템이 즉시 요청을 대신 처리해서 다운타임을 최소화하고 문제없이 서비스가 돌아가도록 하는것
서버장애는 필연적이므로 운영서버라면 꼭 서버를 2대이상 띄워서 장애조치가 가능하도록 처리해야함

#### AutoScaling 을 이용한 장애조치 


[실습] Auto Scaling 그룹과 로드밸런서를 통한 장애조치

EXAMPLE: 두대의 서버를 이용하다가 1개의 서버가 장애가 나는경우 로드밸런서가 자동으로 정상적인 서버에만 요청을 보내보는지 검사 

1. EC2-로드밸런싱-대상그룹- exercise-target-group 을 선택하고 [상태검사] 탭을 클릭
2. HEALTH threshold :2, UNHEALTHY threshold:2, TIMEOUT:2, INTERVAL:5, SUCCESS codes:200
4. AUTO SCALING -> AUTO SCALING 그룹 메뉴 - EXERCISE 그룹 선택후 세부정보 편집 목표용량 최소 최대 모두 2로변경
5. TARGET GROUP 생성후 등록
6. 두 인스턴스 서버에 ssh 접속 -> NEW SESSION
7. cd/opt/nginx/logs 
8. tail -f access.log
9. -------- 두 서버에 나눠서 log 저장되는것을 볼수있다
10. [장애상황재현] sudo service nginx stop
11. 당분간은 홈페이지 접속시 오류메세지가 뜬다
12. 몇초 후면 로드밸런서가 정상서버에만 요청하기떄문에 정상적으로 로그인이 된다.


14. 
15. EC2 로드밸런싱-대상그룹 클릭후 앞서 생성한 대상그룹인 exercise-target-group 을 선택후 상태검사 클릭


## 운영서버의 외부환경 구성

클라이언트가 요청을 보내는 서버마다 고유 IP를 가지고있으나 이는 사람이 알아보기어려움 
www.naver.com www.daum.net 같은 사람이 알아보기쉬운 도메인 주소가필요함


도메인 주소 작동방식(DOMAIN NAME SYSTEM)
1.클라이언트가 DNS 서버에 www.naver.com 도메인 IP주소 질문 
2. DNS 서버에서 상위 DNS 서버로 계속 질문 그리고 알고있는 서버에서 IP 주소를 하위 서버에 응답
3. 클라이언트는 IP주소를 받아 페이지 조회 요청 

[AWS 에서 ROUTE53 을 이용한 도메인 등록가능]
##### 비용이 나가기떄문에 생략
1. AWS route53 검색 후 클릭 



## SSL/TLS, HTTPS 

HTTPS-> HTTP 프로토콜의 보안이 강화된 버젼 ,HTTP 프로토콜에 SSL/TLS 암호화 프로토콜을 이용해 전송되는 데이터를 암호화하는 과정이 추가된것

HTTPS 의 3원칙 : 기밀성,무결성,인증
1. 기밀성: 중간에 내용을 가로채가더라도 내용을 읽지못하게 하는게 암호화
2. 무결성: 내용을 변조해서 중간자 공격을 못하게하는것이 무결성 
3. 클라이엍느가 통신하고있는 서버의 신원을 확인할수있는것이 인증



HTTPS:동작방식
1. 서버 관리자가 인증서 발급기관을 통해 서비스하는 도메인에대한 인증서 발급
2. 발급받은 인증서를 클라이언트와 통신하는 서버 인스턴스의 웹서버에 설치
3. 이용자가 웹부라우저 같은 클라이언트에서 https:// 로 시작하는 주소로 접속을 시도한다
4. 클라이언트는 사용자가 입력한 주소에 해당하는 서버에 SSL/TLS 로 암호화한 통신을 요청하면서 사용할수있는 SSL/TLS 버젼목록과 암호화알고리즘 선택
5. 서버는 전달받은 목록과 알고리즘중 선호하는 것을 고르고 서버에 설치된 인증서를 함꼐 응답.(인증서에 공개암호화키포함)
6. 클라이언트는 인증서가 신뢰할수있는곳으로부터 서명된 것인지 확인후 대칭암호화 키로 사용할 키 (pre-master key생성) 서버에서 받은 공개암호회 키로 암호화해 전달
7. 서버와 클라이언트는 이 대칭암호화 키를 이용해 암복호화가 잘되는지 통신
8. 그다음부터 서버와 클라이언트는 서로 데이터를 주고받을때 이 대칭암호화키를 이용해 암호화해서 전달

## 무중단/중단 배포

#### 핵심은 배포할때 서비스를 중단할지 하지않을지에 대한 차이

<무중단배포>

현재위치 배포
서버:sn (n=1,2,3,4)
s1,s2,s3,s4 

s1,s2를 로드밸런서에서 제외하고 s3,s4로 요청을 나눠서 처리함 
s1,s2에 새로운 버젼의 코드를 배포함  그리고 다시 로드밸런서에 s1,s2 도 등록
이번에는 s3,s4를 로드밸런서에서 제외. s3,s4에 새로운버젼의 코드를 배포후 로드밸런서에등록

장점: 새로운 인스턴스를 생성할필요가없음 
단점:
1. 배포중에 클라이언트의 요청을 처리할 서버의 수가 줄어든다. 대처법: 여유 인스턴스추가
2. Batch사이즈만큼 묶어서 처리한다면 배포시간은 B(1)//B(n) 시간만큼 줄어든다. 하지만 요청량 또한 그만큼 줄어들것이다
3. 배포한버젼에 문제가생기면 이전버젼으로 롤백을 해야하는데 대응하는 시간이 오래걸린다


[서버단위의 블루/그린배포]

Blue(기존), Green(최신코드) 두그룹으로 나눈다. 

1. 아무서버도 가지고있지않은 Green 그룹에 Blue 그룹과 똑같은 인스턴스를 생성한다.
2. 그리고 g1,g2,g3, . . . gn 에 최신버젼의 코드를 배포한다
3. 로드밸런서에 Green 그룹도 등록해서 나눠서 처리하게만든다
4. 로드밸런서에서 Blue 그룹을 제외하고 Green 그룹에서 처리하게만든다.
5. Blue 그룹에 존재하는 인스턴스를 모두 종료하고 배포완료한다.
6. 다음 버젼을 배포할때에는 블루그룹에 새로운 버젼을 배포하고 그린그룹의 인스턴스를 종료하면된다.

장점:
1. 구,신버전이 동시간에 떠있는 시간을 매우 짧게 처리할수있다는점
2. 기본원칙은 구,신버젼이 함께 떠있어도 아무런문제가없어야함
3. 롤백을 굉장히 빨리할수있음. 테스트환경에서 발견이 안되었지만 운영환경에서만 나타나는 버그들. 로드밸런서에서 등록 해제만 하면되기때문에 굉장히 쉬움
4. 실제로 배포후 Blue group 의 인스턴스들을 바로 줄이지않는다. ( 몇시간정도 모니터링을 진행하다가 문제가 확실히없다고 판단될때 종료)
5. 배포과정에서 서비스되는 인스턴스의 수가 줄지않으므로 요청량을 처리하는데서 오는 장애의부담이 없다

단점:
1. 인스턴스의 수를 두배로 늘려야하므로 배포준비시간이 길다

운영 테스트용 로드밸런서를 만들어서 운영테스트용으로도 써볼수있다.

## CODE DEPLOY 

1. 우리가 개발한 애플리케이션 프로젝트의 최상단 경로에 AppSpec.yml이라는 파일을 추가한다. 배포에 필요한 모든절차들을 적어놓은 명세서역할 
2. CodeDeploy 에 프로젝트의 특정한 버전을 배포해달라고 요청
3. 배포진행할 Ec2 인스턴스들에 설치되어있는 CodeDeploy Agent들과 통신하여 Agent 들에게 요청받은 버전을 배포해달라고 요청
4. 요청을 받은 CodeDeploy Agent 들은 코드 저장소에서 프로젝트 전체를 서버에 내려받는다 그리고 내려받은 프로젝트에있는 AppSpect.yml 파일을 읽고 해당파일에 적힌 절차대로 배포진행
5. CodeDeploy Agent 배포를 진행한후 성공/실패 등 결과를 CodeDeploy 에 알려줌

#### https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/CodeDeploy/NCodeDeploy.html  Code Deployment 공식문서 
