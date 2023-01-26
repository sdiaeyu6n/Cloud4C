# Public Cloud Team Project

## 📌 개요

🔹 **주제**

• 다음의 시나리오를 기반으로 Amazon Web Service(AWS) console에서 3 Tier Architecture 전체 플랜을 구현

> 당신은 A기업의 솔루션 아키텍트로 일하고 있다.
A기업은 온라인쇼핑몰을 운영하기 위해 웹서비스를 제공하려고 한다.
이 온라인쇼핑몰은 한국 고객 위주로 운영이 될 것이며 추후에는 해외 고객들도 쇼핑할 수 있도록 운영할 계획이 있다.
신규 비즈니스 및 웹사이트로서 예상되는 고객층 및 고객수에 대한 정보가 없으며, 향후 급속도로 올라가는 접속자 수에 대한 상황도 미리 고려해야한다.
시스템은 24시간 풀가동이 가능해야 하며 추가적으로 사내에서 클라우드에 접속하여 개발을 하기 원하므로 SSH로 접속할 수 있는 환경과 암호 키에 대한 관리가 필요하다.
고객개인정보를 준수해야 하기 때문에 보안조치 또한 신경 써야한다.
솔루션 아키텍처로서, AWS 퍼블릭 클라우드를 기반으로 아키텍트를 구성하고 기획해야 한다.

🔹 **기본 정보**

• 팀원: 정보통신공학과 학부생 4명

• 참여 기간: 2021. 05 - 2021. 06

• 글로벌 클라우드 솔루션 MSP 기업 ‘Cloud4C’와의 협업 프로젝트

🔹 **역할**

• 클라우드 아키텍처 설계 및 구현 - AWS의 여러 컴포넌트를 사용한 서비스 구축

• 발표 자료 제작 및 결과 발표, 보고서 총괄

📍 ********개발 도구********
<div>
  <img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white"> 
  <img src="https://img.shields.io/badge/Amazon Web Services-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white"> 
  <img src="https://img.shields.io/badge/Putty-000000?style=for-the-badge"> 
</div>

## ☁️ 아키텍처 구조
![Untitled](https://user-images.githubusercontent.com/89725142/214896098-2bf7e65c-f308-455b-af64-691e8040732a.png)

<details>
<summary>아키텍처 설명</summary>  
   
<div>       
  <details>   
     
<summary>1. VPC 구성</summary>
  <div>
ㅤ

🔹 vpc는 클라우드 환경을 생성하는 데 가장 기본이 되는 리소스이며 하나의 리전에 종속된 vpc는 독립적이다.  
🔹 vpc를 구성하기 위해서 필요한 요소로는 서브넷, 라우팅 테이블, 인터넷 게이트웨이, 엔드포인트, CIDR 블록 등이 있다. VPC를 구성하기 위해 다음의 과정들을 거쳤다.  
        
- Subnet
        
서브넷은 vpc보다 작은 단위로 물리적인 리소스가 생성되는 가용 영역(available zone)과 연결되며 vpc 내부의 리소스가 생성될 수 있는 네트워크이다.
IPv4 CIDR은 사설 IP 대역 10.0.0.0을 사용하였고 할당 가능한 ip의 개수는 masking에 16을 사용하여 64K개만큼 할당해주었다. webserver-subnetpublic1, 2와 webserver-subnetprivate1, 2 총 4개의 서브넷을 생성하였다.
        
- Routing Table
        
라우팅 테이블은 네트워크에서 destination address를 네트워크 노선으로 적용시키는데에 이용된다. 라우팅 테이블의 각 노드에는 패킷을 목적지로 보내는 최적의 경로에 대한 정보가 포함되어 있으며, 각 노드에서는 패킷의 데이터를 확인하고 최적의 경로를 설정해준다. 라우팅 테이블 생성 과정에서는 앞에서 생성한 SeoulRegion vpc를 공간으로 선택하여 private용 라우팅 테이블과 public용 라우팅 테이블을 생성하였다. priavte 라우팅 테이블에는 private subnet 1, 2를 연결하고 public 라우팅 테이블에는 public subnet 1, 2를 연결하였다.
     
- Internet Gateway
        
IGW는 네트워크 간에 통신을 가능하게 해주고 다른 네트워크에 접근하기 위해서는 반드시 거쳐야 한다. vpc와 subnet을 만들고 subnet 내부에 인스턴스를 생성하여도 인스턴스에 접근하기 위해서는 vpc에 IGW를 연결하고 subnet의 라우팅 테이블이 IGW를 가리키도록 설정해야 한다.  
vpc에 IGW를 연결하기 위해 먼저 IGW를 생성하였다. 생성한 IGW를 앞에서 생성한 SeoulRegion vpc와 연결하고 IGW의 이름을 SeoulRegion-IGW로 하였다.  
         
두 번째로, subnet의 라우팅 테이블이 IGW를 가리키게 하기 위해 앞에서 생성했던 라우팅 테이블 WebServer-RouteTable-public의 라우팅 편집을 하였다. 목적지 대상은 0.0.0.0/0으로 설정하고 타겟 대상에는 SeoulRegion-IGW의 id 값을 넣었다.   
0.0.0.0/0은 모든 IP를 의미하므로 두 번째 추가한 규칙은 VPC CIDR 접근 범위 이외의 모든 IP를 IGW로 라우트해주는 규칙이다.   
        
</div>
</details>        
 <details>
<summary>2. RDS 구축 및 연결</summary>
  <div>
  ㅤ  

🔹 Amazon에서 제공하는 데이터베이스 중 RDS를 선택하여 사용했다. Amazon RDS는 보안성과 확장성이 뛰어나 인스턴스와 스토리지에 대해 유연한 대응이 가능하고 다양한 CPU/Memory 옵션을 제공하여 스토리지를 필요에 따라 확장시킬 수 있다.  
🔹 Amazon RDS는 자동 백업 기능을 가지고 있어 손실된 데이터를 쉽게 복구할 수 있으며 백업된 snapshot을 통해 데이터베이스를 생성할 수도 있다. 또한 멀티 AZ 기능을 가지고 있어 다른 AZ에 데이터를 저장하여 안정성을 높여준다. 이렇게 AZ간 동기화 구성이 가능하기 때문에 장애나 재난 대처에도 효과적이며 데이터 이전이 비교적 쉽다.  
🔹 또한, Amazon RDS 엔진의 저장 및 전송 중 암호화 기능을 활용하면 네트워크에 대한 접근을 쉽게 제어할 수 있어 보안성이 뛰어나다.  
        
- 보안 그룹 생성
 
RDS를 EC2에 연결하기 위해 먼저 보안그룹을 생성했다. 보안그룹은 SeoulRegion-DB를 생성했다. 보안 그룹의 인바운드 규칙에는 지정된 EC2의 인스턴스에서만 접근할 수 있도록 규칙을 추가했다.
        
- 서브넷 그룹 생성
        
EC2와 동일한 VPC ID를 선택하고 관련된 모든 서브넷을 추가해 서브넷 그룹을 생성하였다.
        
- 파라미터 생성 및 설정
        
MySQL 8.0버전의 파라미터 그룹을 생성하고 캐릭터 인코딩 값들을 모두 UTF8로 변경했다.
또, collation의 값들을 UTF8_general_ci로 변경했다.
        
- 데이터베이스 생성
        
데이터베이스 생성 탭에서 MySQL community 엔진을 선택하고 free tier에서 사용 가능한 t2.micro 클래스를 사용하였다. 서브넷 그룹은 SeoulRegion을 선택하고 보안 그룹은 이전에 생성한 SeoulRegion-DB를 선택했다. 파라미터는 앞에서 생성한 파라미터 그룹을 선택하여 데이터베이스를 생성했다.
</div>
</details>      
 <details>
<summary>3. IAM 사용자 설정</summary>
  <div>
ㅤ

🔹 dgunigrp4(솔루션 아키텍처) 계정에게는 Root 계정에 준하는 AWS FULL ACCESS 권한을 부여함으로서 AWS의 모든 리소스에 접근 가능하도록 설정하였다.  
🔹 Company(회사) 계정의 경우 IAM 그룹(Group4) 안에 계정을 생성하게 되었다.  
🔹 IAM 그룹으로 IAM 사용자들을 모아 관리를 하게 될 경우 IAM 그룹에 접근제어 및 권한설정을 할 수 있으며 IAM 그룹에 설정된 내용은 IAM 그룹 안에 포함된 모든 사용자들에게 적용되기에, 관리가 편하기 때문에 설정하게 되었다.  
🔹 Group4의 권한을 AmazonEC2FullAccess를 부여함으로써, Group4에 속해있는 IAM사용자들은 이 권한을 따르게 된다. Group4에 IAM 사용자(dguunigrp4)를 추가하였다.  
🔹 사용자(IAM) 이름 설정 및 액세스 키 ID 및 비밀 액세스 키를 활성화해야 하는데, 액세스 키의 경우 분실 시 사용자를 다시 생성해야 하므로 보관에 유의하여야 한다.  

</div>
</details>      
 <details>
<summary>4. Web Server Tier</summary>
  <div markdown="5">     
  ㅤ

🔹 web server 의 경우 한 서버가 다운되는 경우 대비하기 위하여 두개의 서버를 구성하기로 하였다. 그 중 가용영역 C에 있는 서버는 가용영역 A에 생성할 서버의 Auto Scaling 을 통해 생성하였다.     
🔹 보통 서버의 경우 크기를 크게해야 여러 트래픽들을 정리하는데 용이하다. 하지만, 프로젝트 기획안에서 프리티어 내에서만 사용하도록 제한하고 있으므로 프리티어의 최대한도(30GiB)보다 낮은 8GiB 의 서버를 생성하기로 하였다.    
🔹 이렇게 생성된 서버에 접속하기 위해서는 SSH(Secure Shell)를 사용하게 된다. Windows의 경우 putty를 사용하게 된다.    
🔹  Security Group을 활용해 EC2 인스턴스 즉 웹서버에게 방화벽 설정을 하였다.      
🔹  보안그룹을 생성하고 Security Group Inbound 규칙란에서 HTTP의 경우 위치와 무관하게 모든 IP가 접근할 수 있게끔 하였다. 그 결과 2개의 Security Group이 형성되는데, 이 부분은 쇼핑몰 사용자들이 Web server에 접근할 경우 HTTP 즉 , 홈페이지만 접근할 수 있게끔 구현한 것이다.     
🔹  반면 SSH , 즉 서버에 직접 접근하여 서버의 속성을 변경 할 수 있는 IP는 솔루션 아키텍쳐의 IP만 부여함으로서 다른 IP는 접근이 불가 하도록 하였다.          
🔹  또한,  ICMP - IPV4 의 주소의 경우 같은 가용영역 내의 WAS Security Group과의 상호통신을 확인하기 위해 같은 가용영역 내의 WAS security group의 주소를 설정하였다. 따라서 security group의 경우 auto scaling을 활용하여 생성된 가용영역 C에 위치한 web server와 본래 생성된 가용영역 A의 web server와 한쌍으로 SeoulRegion-WEB_Security, WAS Server 한 쌍 씩 SeoulRegion-WAS_Security 그룹으로 지정하였다.
 </div>
</details>      
 <details>
<summary>5. Web Application Tier</summary>
  <div>   
  ㅤ  


🔹  Web Application Server(WAS)의 경우 web server와 마찬가지로 가용영역 C에 속해 있는 PRIVATE SUBNET에 가용영역A에 서버를 먼저 생성한 후 Auto Scaling을 통해 한개 더 생성하게 되었다.  
🔹  WAS server도 마찬가지로 security group을 형성할때 인바운드 규칙의 경우 SSH 설정은 web server와 동일하고, ICMP-IPV4의 경우 web security group의 주소를 설정하게 되어 마찬가지로, 통신을 위해 설정하게 되었다.   
🔹  WAS 서버의 경우 USER의 경우 접근 할 이유가 없다. WAS에서 제공하는 application을 WEB server로 옮긴 후 사용자의 접근이 발생하기 때문에  HTTP에 대한 접근 권한은 부여할 필요가 없다.  
 </div>
</details>      
 <details>
<summary>6. Load Balancer</summary>
  <div>           
  ㅤ  


🔹  서버의 역할중 하나는 서버를 이용하는 고객들의 요청을 응답해주는 것이다.  
🔹  고객의 수가 적을 때는 서버가 모든 고객의 요청을 들어줄 수 있지만, 고객의 수가 급격히 많아지면 서버는 과부하가 걸려 동작을 멈추게 된다. 이를 해결하기 위해서는 하드웨어의 성능을 향상시키거나(scale-up) 서버를 여러 개 생성하는 방법(scale-out)이 있다.   
🔹  하드웨어를 향상시키는 것보다는 서버를 여러 개 생성하는 것이 비용적인 측면에서 더 효과적이고 자연재해에 의한 피해 예방에도 효율적이기 때문에 확장성이 좋은 scale-out 방식을 선호한다. 이때 생성한 여러 개의 서버에 트래픽을 분산시키는 역할을 하는 것이 로드밸런서이다. 즉, 로드밸런싱은 여러 서버에서 트래픽을 분산 처리하여 서버의 부하, 속도 등을 고려해 처리해주는 것이다.
        
• NLB로드밸런서를 생성하기 위해 이름은 Web-NLB로 설정하고 VPC영역은 생성했던 SeoulRegion VPC를 사용, Subnet은 SubnetPublic1, 2를 선택했다.
        
• 가용영역으로는 ap-northeast-2a와 2c를 사용했다. 사용할 VPC, Subnet, 가용 영역을 선택함으로써 로드밸런서에서 트래픽을 라우팅할 VPC와 Subnet, 가용영역을 지정해주었다.
        
• Target Group으로는 NLB인스턴스를 생성하여 사용했다.
        
• 생성한 네트워크 로드밸런서를 확인해보면 target group NLB로 연결 하고 있는 것을 확인할 수 있다.
 </div>
</details>      
 <details>
<summary>7. Auto Scaling</summary>
  <div>            
    ㅤ  


🔹  Auto Scaling은 사용자 정책에 따라 서버를 자동으로 생성하고 삭제해주는 서비스이다.   
🔹  사용자가 많을 때는 원활한 서비스를 위해 서버를 늘리고 사용자가 적을 때는 서버를 줄여 불필요한 서버 관리로 발생하는 비용을 줄여준다. 이러한 Auto Scaling의 특성을 제대로 활용하기 위해서는 여러 개의 서버에 트래픽을 자동 분산 시켜주는 Load Balancer에 연결하여 사용해야 한다. 따라서 Auto Scaling을 구현할 때 Load Balancer와 연결하여 구현하였다.  
        
- Auto Scaling 그룹 생성
        
먼저 Web1 인스턴스를 Auto Scaling 해주었다.
        
Auto Scaling 그룹 이름을 AS_Web1으로 설정하고 시작 템플릿은 web1의 시작 템플릿인 ST_web1을 선택했다.
        
- 네트워크 설정

VPC는 SeoulRegion VPC를 선택하고 웹서버 서브넷인 SubnetPrivate1을 사용했다.
        
Auto Scaling의 효율을 위해 기존에 생성했던 Load Balancer와 연결해주었다.
Network LoadBalancer를 생성하였으므로 로드 밸런서 대상 그룹에서 선택을 확인하고 Web-NLB Load Balancer와 연결해주었다.
        
- 로드 밸런서에 연결
        
Auto Scaling의 효율을 위해 기존에 생성했던 Load Balancer와 연결해주었다.
Network LoadBalancer를 생성하였으므로 로드 밸런서 대상 그룹에서 선택을 확인하고 Web-NLB Load Balancer와 연결해주었다.
        
같은 과정을 Web2, WAS1, WAS2에 반복하여 총 4개의 Auto Scaling 그룹을 생성해 서버의 생성 및 삭제를 하면서 트래픽을 다중 분산 처리할 수 있도록 해주었다.
        
최종적으로 로드밸런서와 오토스케일링을 사용하여 Web과 WAS의 설계가 위 사진과 같이 되도록 하였으며 AWS의 인스턴스 목록에서 확인 결과 아래 사진과 같이 오토스케일링으로 인해 규모가 자동 확장된 모습을 확인할 수 있었다.
 </div>
</details>      
 <details>
<summary>8. PuTTY를 이용한 SSH 접근</summary>
  <div>
ㅤ
  
🔹  SSH는 네트워크 프로토콜의 하나로 public network를 이용해 통신할 때 데이터 전송, 원격 접속 등을 안전하게 사용하기 위해 사용한다.   
🔹  AWS의 클라우드 인스턴스 서비스를 이용하기 위해서 SSH가 사용되는데, SSH를 통해 원격 접속을 하거나 암호화된 통신을 가능하게 해 정보의 유출을 방지해준다.  
🔹  EC2의 네트워크 보안 탭에서 키 페어 생성 서비스를 이용해 키 페어를 생성하였다. 키 페어 생성을 누른 뒤 자동으로 다운로드 되는 MasterKey.pem을 받아 ppk 형식으로 전환하기 위해 PuTTYgen을 사용하였다.        
🔹  PuTTYgen에서 다운 받은 MasterKey.pem의 경로를 지정하고 Save Private Key를 눌러 ppk형식으로 저장한다. 하지만 MasterKey.ppk를 이용해 Web과 WAS에 PuTTY를 이용해 접속 시도를 했지만 실패했다. 그 이유는 Web과 WAS에 접근하기 위해서는 Bastion host를 거쳐야 하는데 Bastion host는 ppk가 아닌 pem 형식을 지원하기 때문에 ppk에서 다시 pem으로 전환하는 과정이 필요했다.    
🔹  ppk 형식으로 변환할 때는 conversion 탭의 import key 메뉴를 사용했지만 pem 형식으로 변환할 때는 Export OpenSSH key 메뉴를 사용해 전환하였다. 이렇게 최종적으로 MasterKey.pem을 SSH 접속의 키 페어로써 사용하였다.   
🔹  Bastion host 인스턴스의 인바운드, 아웃바운드 규칙 수정 후 PuTTY를 통해 Private IP주소로 접속해 PrivateSubnet 내의 Webserver에 접속 가능한 것을 확인했다. 이 때 bastion host와의 연결을 위해 master.pem 키를 생성하여 사용하였다.  
 <details>
<summary>9. Private Key 관리</summary>
  <div>        
ㅤ
  
🔹  IAM 사용자 접속 및 SSH 접근에 이용되는 Private Key의 관리를 위해 AWS Management 콘솔을 이용해 정기적으로 키를 교체하는 방법을 채택했다.   
🔹  이는 키가 손상돼도 어플리케이션 작동에 영향이 미미하다는 장점이 있어 키 관리 방법으로 채택하게 되었다.   

</div>
</details>
