## 현재 구축된 인프라

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d1db6b7-e5cf-4aeb-99ad-f487644d7bc4/Untitled.png)

- EC2
    - Docker container
        - Reverse proxy server (nginx, https)
    - Docker container
        - web server (nginx)
    - Docker container
        - WAS
    - Docker container
        - DB (postgres)

네트워크 구조

```tsx
EC2's IP -> proxy -> web server -> get pages!
page -> EC2's IP -> proxy -> WAS -> DB
```

.conf

```tsx
EC2's IP:80 접속(443 포트로 치환)
경로에 따라 web 혹은 was로 연결
db는 was와 내부 통신만 가능

하나의 EC2 IP
```

## 새로 설계된 인프라

과금, DDOS, 주의

- CloudFront (https)
    - S3 (web server)

- ALB (https)
    - EC2
        - Nginx Proxy (docker container)
        - WAS (docker container)

- AWS-RDS (postgres)

네트워크 구조

```tsx
CloudFront(S3) -> ALB(Proxy) -> EC2(was with nginx) -> DB
```

### 인프라 변경하므로써 얻는 이점

1. 역할에 따른 물리적 분리, 효율적 방식의 부하 분산
    
    1차적으로 웹서버와 was, 데이터베이스가 물리적으로 분리되고 웹서버는 정적파일 요청, was는 데이터베이스에 대한 요청만 처리하게 된다.
    
2. 캐싱(Cloudfront)을 이용해 웹서버 트래픽 분산, UX 개선
    
    2차적으로 웹 서버는 cloudfront 서비스를 이용해 전세계 여러 서버에 정적 파일을 캐싱해둔다. 물리적으로 여러대의 서버를 사용하므로 웹서버는 WAS와 분리된 것 외에도 한번 더 트래픽이 분산되는 구조가 생긴다.
    

### 인프라가 변경되면 수정되는 부분과 도커의 장점

도커를 써서 애초에 서로 다른 ip로 통신하는 것처럼 설계되어있음

1. 도메인 설정 변경
2. 프론트 코드에서 api 주소 변경 (처음부터 전역 변수로 설정하면 간단하게 수정 가능)
3. was의 config에서 DB ip 변경

### 궁금했던 점

1. cloudfront는 각각 다른 서버인데 cors는 자동으로 설정되나?
    1. cors에 웹사이트 도메인네임을 등록하면, 도메인네임으로 검색해서 ip를 판단하나?
2. cloudfront의 도메인 설정은? ip가 필요한데..
    1. route5를 네임서버로 사용하고, 가비아에서 연결했던 ec2의 ip는 삭제해준다.
    2. ip대신 도메인네임(별칭)을 사용한다.
    3. route5와 cloudefront를 연결하면 나는 ip에 대해 신경 쓸 필요가 없어지는 건가?
3. ALB 도메인네임이 있으면 IP는 변경되어도 상관 없는가?
    1. 1번과 동일
4. ALB → WAS 사이의 cors 설정만 해주면 되나? ip대신 도메인네임으로 설정할수있는건가?
    1. 1번과 동일, WAS의 CORS설정에 ALB 도메인네임을 등록하면 된다.

```tsx
www.baeum.net(클라우드프론트)에서 aws.alb.com으로 api 요청
aws에서 설정한 alb ip로 연결됨. alb는 www.baeum.net의 요청을 허락함
이때 www.baeum.net은 클라우드프론트 서비스이므로 여러 ip중 하나가 될텐데
이 부분을 route5와 cloudfront 설정함으로써 해결함
was는 aws.alb.com의 요청을 허락함
```

### 도메인 설정 과정 (순서 아님)

도메인네임을 aws 서비스에 사용하려면 Route53이 있어야함 (aws서비스의 동적인 IP와 도메인을 연결)

- 클라우드프론트
    - 엔드포인트를 S3로 설정
    - 도메인 설정
- 구매한 도메인네임을 이용해서 AWS route53에 레코드를 생성한다. (도메인 네임 서버 구축)
    - 이 과정에서 route53은 도메인네임에 엔드포인트를 연결한다.
        - AWS의 DNS로 이전한 것이다.
        - 도메인으로 원하는 서비스에 접속 가능 → ACM (SSL) 발급 신청 가능
- ACM 발급