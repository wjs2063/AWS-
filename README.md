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
1. curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install | bash
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

