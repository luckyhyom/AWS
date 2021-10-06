### DevOps
- IaC?
    - 코드 자체가 문서가 된다.
        - 깃허브를 통해 모든 코드의 변경사항은 지속적으로 공유되고 리뷰를 받아야한다.
    - 인프라를 이루는 구성 요소들을 코드로 관리함
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

**vpc.tf**

```tsx
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "terraform-101"
  }
}

resource "aws_subnet" "public_subnet" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  availability_zone = "ap-northeast-2a"

  tags = {
    Name = "101subnet-1"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table_association" "route_table_association_public" {
  subnet_id = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route" "public_route" {
  route_table_id = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.igw.id
}

resource "aws_subnet" "private_subnet" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"

  tags = {
    Name = "101subnet-2"
  }
}

resource "aws_eip" "nat" {
  vpc = true

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_nat_gateway" "nat_gateway" {
  allocation_id = aws_eip.nat.id

  subnet_id = aws_subnet.public_subnet.id
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table_association" "route_table_association_private" {
  subnet_id = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route" "private_nat" {
  route_table_id = aws_route_table.private.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id = aws_nat_gateway.nat_gateway.id
}
```

**Public Subnet**

1. vpc 생성
2. subnet 생성
3. internet gateway 생성
4. route table 생성
5. route table association 설정
6. route rule 설정

**Private Subnet**

1. 동일한 vpc에 subnet 생성
2. 고정 ip 생성 (nat gateway는 고정ip를 필요로 한다.)
3. nat gateway 생성
    - internet gateway 대신 nat gateway가 public subnet에 기생하며 private subnet을 인터넷에 연결시켜준다.
4. private subnet과 nat gateway의 라우팅 설정

### S3

- web server
- versioning

### IAM

콘솔에서 access key를 통해 권한을 가지고 작업할수있지만,
실제 서버에서 put, get 등을 하려고 s3에 접근하려하면? 서버 인스턴스에 iam role을 할당한다.
즉, 암호나 액세스키를 공유하지 않고 aws 계정의 리소스를 관리하고 사용할수 있는 권한을 부여할수있다.

```tsx
aws s3 ls // 현재 인스턴스의 configure에 모든 권한을 가진 키를 저장한 상태여서 s3에 접근할수있다.
cat ~/.aws/credentials // 권한을 가지고 있는 키
aws configure // credentials 파일을 삭제하면 configure 정보도 초기화됨

// aws 웹콘솔에서 iam role을 설정하면 다시 s3 접근가능해짐
// 서버 configure에 access key를 등록하면 유실될수있으니 role을 생성해 부여하는건가
```
