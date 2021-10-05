### DevOps
- IaC?
    - 인프라를 이루는 구성 요소들을 코드로 관리함
    - 스테이트 파일과 현재 인프라의 상태를 똑같이 유지하는 것이 핵심
        - AWS 인스턴스를 생성하고 여러가지 옵션을 설정하는 것을 코드로 작성해서 할 수 있음.
    
 ssh 접속이 안될 때 확인해야 할 사항

- 시큐리티 그룹에서 자신의 IP 등록
- ec2의 public IP
- public subnet
- nacl

최초 유저 ID
- ubuntu
- ec2-user

aws를 공부한다는 것은 vpc를 빠짐없이 공부한다는 것

실제서비스에서는 well-known port는 지양

현재 AWS 리눅스에 설치된 것들

```tsx
sudo apt-get install zip unzip
```
