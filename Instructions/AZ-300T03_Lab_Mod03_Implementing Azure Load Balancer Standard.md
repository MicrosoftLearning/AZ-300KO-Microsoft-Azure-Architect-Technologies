# 고급 가상 네트워킹 구현
# 랩: 표준 Azure Load Balancer 구현
  
### 시나리오
  
Adatum Corporation 은 표준 Azure Load Balancer 를 구현하여 Azure VM의 인바운드 및 아웃바운드 트래픽을 전송하려고 합니다. 


### 목표
  
이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-  표준 Azure Load Balancer 를 사용하여 인바운드 부하 분산 구현 

-  표준 Azure Load Balancer 를 사용하여 아웃바운드 SNAT 트래픽 구성 

### 랩 설정
  
예상 시간: 45분

사용자 이름: **학생**

암호: **Pa55w.rd**


## 연습 1: 표준 Azure Load Balancer 를 사용하여 인바운드 부하 분산 및 NAT 구현 
  
이 연습의 주요 태스크는 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 가용성 집합에서 Azure VM 배포

1. 표준 Azure Load Balancer 의 인스턴스 만들기

1. 표준 Azure Load Balancer 의 부하 분산 규칙 만들기

1. 표준 Azure Load Balancer 의 NAT 규칙 만들기

1. 표준 Azure Load Balancer 의 기능 테스트


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 가용성 집합에서 Azure VM 배포

1. 랩 가상 머신에서 Microsoft Edge 를 시작하여 Azure Portal [**http://portal.azure.com**](http://portal.azure.com) 로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell** 내에서 **Bash** 세션을 시작합니다. 

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

    - 구독: 대상 Azure 구독의 이름

    - Cloud Shell 지역: 구독에서 사용할 수 있고 랩 위치에 가장 가까운 Azure 지역의 이름입니다.

    - 리소스 그룹: 새 리소스 그룹 **az3000800-LabRG**의 이름

    - 스토리지 계정: 새 스토리지 계정의 이름

    - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다.

   ```
   az group create --name az3000801-LabRG --location <Azure region>
   ```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 매개 변수 파일 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.parameters.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 Windows Server 2016 Datacenter를 호스트하는 Azure VM 쌍을 배포합니다.

   ```
   az deployment group create --resource-group az3000801-LabRG --template-file azuredeploy0801.json --parameters @azuredeploy0801.parameters.json
   ```

    > **참고**: 다음 태스크를 진행하기 전에 배포가 완료될 때까지 기다립니다. 배포가 완료되려면 10분 정도 걸립니다.

1. Azure Portal 에서 Cloud Shell 창을 닫습니다.


#### 작업 2: 표준 Azure Load Balancer의 인스턴스 만들기

1. Azure Portal에서 다음 설정을 사용하여 새 Azure Load Balancer 를 만듭니다.

    - 구독: 대상 Azure 구독의 이름

    - 리소스 그룹: **az3000801-LabRG**
    
    - 이름: **az3000801-lb**

    - 지역: 이 연습의 이전 태스크에서 Azure VM을 배포한 Azure 지역의 이름
    
    - 유형: **공용**

    - SKU: **표준**

    - 공용 IP 주소: **새로 만들기** - **az3000801-lb-pip01**로 이름 지정

    - 가용성 영역: **영역 중복**


#### 작업 3: 표준 Azure Load Balancer 의 부하 분산 규칙 만들기

1. Azure Portal 에서 새로 배포한 Azure Load Balancer **az3000801-lb**의 속성이 표시되는 블레이드로 이동합니다.

1. **az3000801-lb** 블레이드에서 **백 엔드 풀**을 클릭합니다.

1. **az3000801-lb - 백 엔드 풀** 블레이드에서 **+ 추가**를 클릭합니다.

1. **백 엔드 풀 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 클릭합니다.

    - 이름: **az3000801-bepool**

    - 가상 네트워크: **az3000801-vnet(VM 2개)**

    - 가상 머신: **az3000801-vm0**  IP 주소: **ipconfig1(10.0.0.4)** 또는 **ipconfig1(10.0.0.5)**

    - 가상 머신: **az3000801-vm1**  IP 주소: **ipconfig1(10.0.0.5)** 또는 **ipconfig1(10.0.0.4)**

    > **참고**: 가상 머신의 IP 주소는 역순으로 할당될 수도 있습니다. 

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.

1. **az3000801-lb - 백 엔드 풀** 블레이드로 돌아와서 **상태 프로브**를 클릭합니다.

1. **az3000801-lb - 상태 프로브** 블레이드에서 **+ 추가**를 클릭합니다.

1. **상태 프로브 추가** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 이름: **az3000801-healthprobe**

    - 프로토콜: **TCP**

    - 포트: **80**

    - 간격: **5**

    - 비정상 임계값: **2**

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.

1. **az3000801-lb - 상태 프로브** 블레이드로 돌아와서 **부하 분산 규칙**을 클릭합니다.

1. **az3000801-lb - 부하 분산 규칙** 블레이드에서 **+ 추가**를 클릭합니다.

1. **부하 분산 규칙 추가** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 이름: **az3000801-lbrule01**

    - IP 버전: **IPv4**

    - 프런트 엔드 IP 주소: 드롭다운 목록에서 **LoadBalancedFrontEnd**에 할당된 공용 IP 주소를 선택합니다.

    - 프로토콜: **TCP**

    - 포트: **80**

    - 백 엔드 포트: **80**

    - 백 엔드 풀: **az3000801-bepool(가상 머신 2개)**

    - 상태 프로브: **az3000801-healthprobe(TCP:80)**

    - 세션 지속성: **없음**

    - 유휴 시간 제한 (분): **4**

    - 부동 IP(Direct Server Return): **사용 안 함**

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.


#### 작업 4: 표준 Azure Load Balancer의 NAT 규칙 만들기

1. Azure Portal 의 **az3000801-lb** 블레이드에서 **인바운드 NAT 규칙**을 클릭합니다.

1. **az3000801-lb - 인바운드 NAT 규칙** 블레이드에서 **+ 추가**를 클릭합니다.

1. **인바운드 NAT 규칙 추가** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 이름: **az3000801-vm0-RDP**

    - 프런트 엔드 IP 주소: 드롭다운 목록에서 **LoadBalancedFrontEnd**에 할당된 공용 IP 주소를 선택합니다.

    - IP 버전: **IPv4**

    - 서비스: **RDP**

    - 프로토콜: **TCP**

    - 포트: **33890**

    - 대상 가상 머신: **az3000801-vm0**

    - 네트워크 IP 구성: **ipconfig1(10.0.0.4)** 또는 **ipconfig1(10.0.0.5)**

    - 포트 매핑: **사용자 지정**

    - 부동 IP(Direct Server Return): **사용 안 함**

    - 대상 포트: **3389**

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.

1. **az3000801-lb - 인바운드 NAT 규칙** 블레이드로 돌아와서 **+ 추가**를 클릭합니다.

1. **인바운드 NAT 규칙 추가** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 이름: **az3000801-vm1-RDP**

    - 프런트 엔드 IP 주소: 드롭다운 목록에서 **LoadBalancedFrontEnd**에 할당된 공용 IP 주소를 선택합니다.

    - IP 버전: **IPv4**

    - 서비스: **RDP**

    - 프로토콜: **TCP**

    - 포트: **33891**

    - 대상 가상 머신: **az3000801-vm1**

    - 네트워크 IP 구성: **ipconfig1(10.0.0.5)** 또는 **ipconfig1(10.0.0.4)**

    - 포트 매핑: **사용자 지정**

    - 부동 IP(Direct Server Return): **사용 안 함**

    - 대상 포트: **3389**

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.


#### 작업 5: 표준 Azure Load Balancer 의 기능 테스트

1. Azure Portal 에서 **az3000801-lb** 블레이드로 이동하여 **공용 IP 주소** 항목의 값을 적어 둡니다.

1. 랩 컴퓨터에서 Microsoft Edge 를 시작하고 이전 단계에서 확인한 IP 주소로 이동합니다.

1. 기본 **인터넷 정보 서비스 시작** 페이지가표시되는지 확인합니다. 

1. 랩 컴퓨터에서 **시작** , **실행**을 차례로 클릭하고 **열기** 텍스트 상자에서 다음 명령을 실행합니다. 여기서 `<IP address>` 자리 표시자는 이 태스크의 이전 단계에서 확인한 공용 IP 주소로 바꿉니다.

   ```
   mstsc /v:<IP address>:33890
   ```

1. 메시지가 표시되면 다음 값을 지정하여 인증합니다.

    - 사용자 이름: **학생**

    - 암호: **Pa55w.rd1234**

1. 원격 데스크톱 세션 내의 서버 관리자 창에서 **로컬 서버 **보기로 전환한 다음 **az3000801-vm0**Azure VM 에 연결되어 있는지 확인합니다.

1. 랩 컴퓨터로 전환하여 **시작** , **실행**을 차례로 클릭하고 **열기** 텍스트 상자에서 다음 명령을 실행합니다. 여기서 `<IP address>` 자리 표시자는 이 태스크의 이전 단계에서 확인한 IP 주소로 바꿉니다.

   ```
   mstsc /v:<IP address>:33891
   ```

1. 메시지가 표시되면 다음 값을 지정하여 인증합니다.

    - 사용자 이름: **학생**

    - 암호: **Pa55w.rd1234**

1. 원격 데스크톱 세션 내의 서버 관리자 창에서 **로컬 서버 **보기로 전환한 다음 **az3000801-vm1**Azure VM 에 연결되어 있는지 확인합니다.

1. 원격 데스크톱 세션 내에서 Windows PowerShell 세션을 시작하고 다음 명령을 실행하여 현재 공용 IP 주소를 확인합니다.

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. cmdlet의 출력을 검토하여 IP 주소 항목이 이 태스크의 이전 단계에서 확인한 공용 IP 주소와 일치하는지 확인합니다.

1. 원격 데스크톱 세션은 다음 연습에서 사용할 것이므로 열어 둡니다.



> **결과**: 표준 Azure Load Balancer 인바운드 부하 분산 및 NAT 규칙을 구현하고 테스트하여 이 연습을 완료해야 합니다.


## 연습 2: 표준 Azure Load Balancer 를 사용하여 아웃바운드 SNAT 트래픽 구성
  
이 연습의 주요 태스크는 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 기존 가상 네트워크에 Azure VM 배포

1. 표준 Azure Load Balancer 를 만들고 아웃바운드 SNAT 규칙 구성

1. 표준 Azure Load Balancer 의 아웃바운드 규칙 테스트


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 기존 가상 네트워크에 Azure VM 배포

1. 랩 가상 머신에서 Microsoft Edge 를 시작하여 Azure Portal [**http://portal.azure.com**](http://portal.azure.com) 로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell** 내에서 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 매개 변수 파일 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.parameters.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 Windows Server 2016 Datacenter 를 호스트하는 Azure VM 쌍을 배포합니다.

   ```
   az deployment group create --resource-group az3000801-LabRG --template-file azuredeploy0802.json --parameters @azuredeploy0802.parameters.json
   ```

    > **참고**: 다음 태스크를 진행하기 전에 배포가 완료될 때까지 기다립니다. 배포가 완료되려면 5분 정도 걸립니다.

1. Azure Portal 에서 Cloud Shell 창을 닫습니다.


#### 작업 2: 표준 Azure Load Balancer 를 만들고 아웃바운드 SNAT 규칙 구성

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell** 내에서 **Bash** 세션을 시작합니다. 

1. Azure Portal 내의 Cloud Shell 창에서 다음 명령을 실행하여 부하 분산 장치의 아웃바운드 공용 IP 주소를 만듭니다.

   ```
   az network public-ip create --resource-group az3000801-LabRG --name az3000802-lb-pip01 --sku standard
   ```

1. Azure Portal 내의 Cloud Shell 창에서 다음 명령을 실행하여 표준 Azure Load Balancer를 만듭니다.

   ```sh
   LOCATION=$(az group show --name az3000801-LabRG --query location --out tsv)
   az network lb create --resource-group az3000801-LabRG --name az3000802-lb --sku standard --backend-pool-name az3000802-bepool --frontend-ip-name loadBalancedFrontEndOutbound --location $LOCATION --public-ip-address az3000802-lb-pip01
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 아웃바운드 규칙을 만듭니다.

   ```
   az network lb outbound-rule create --resource-group az3000801-LabRG --lb-name az3000802-lb --name outboundRuleaz30000802 --frontend-ip-configs loadBalancedFrontEndOutbound --protocol All --idle-timeout 15 --outbound-ports 10000 --address-pool az3000802-bepool
   ```

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.

1. Cloud Shell 창을 닫습니다.

1. Azure Portal 에서 Azure Load Balancer **az3000802-lb**의 속성이 표시되는 블레이드로 이동합니다.

1. **az3000802-lb** 블레이드에서 **백 엔드 풀**을 클릭합니다.

1. **az3000802-lb - 백 엔드 풀** 블레이드에서 **az3000802-bepool**을 클릭합니다.

1. **az3000802-bepool** 블레이드에서 다음 설정을 지정하고 **저장**을 클릭합니다:

    - 가상 네트워크: **az3000801-vnet(VM 4개)**

    - 가상 머신: **az3000802-vm0**  IP 주소: **ipconfig1(10.0.1.4)** 또는 **ipconfig1(10.0.1.5)**

    - 가상 머신: **az3000802-vm1**  IP 주소: **ipconfig1(10.0.1.5)** 또는 **ipconfig1(10.0.1.4)**

    > **참고**: 작업이 완료될 때까지 기다립니다. 작업은 1분 이내에 완료됩니다.


#### 작업 3: 아웃바운드 규칙이 적용되었는지 확인
 
1. Azure Portal에서 **az3000802-lb** 블레이드로 이동하여 **공용 IP 주소** 항목의 값을 적어 둡니다.

1. 랩 컴퓨터에서** **다음 명령을 실행하여 **az3000801-vm0**으로의 원격 데스크톱 세션을 시작합니다.

   ```
   mstsc /v:az3000802-vm0
   ```

1. 메시지가 표시되면 다음 값을 지정하여 인증합니다.

    - 사용자 이름: **학생**

    - 암호: **Pa55w.rd1234**

1. **az3000802-vm0**으로의 원격 데스크톱 세션 내에서 Windows PowerShell 세션을 시작하고 다음 명령을 실행하여 현재 공용 IP 주소를 확인합니다.

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. cmdlet의 출력을 검토하여 IP 주소 항목이 이 태스크의 이전 단계에서 확인한 공용 IP 주소와 일치하는지 확인합니다.


> **결과**: 표준 Azure Load Balancer 아웃바운드 NAT 규칙을 구성하고 테스트하여 이 연습을 완료해야 합니다. 


## 연습 3: 랩 리소스 제거

#### 작업 1: Cloud Shell 열기

1. 포털 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. 필요한 경우 Cloud Shell 창의 왼쪽 상단에 있는 드롭다운 목록을 사용하여 Bash 셸 세션으로 전환하십시오.

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹을 나열합니다.

   ```
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv
   ```

1. 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 다음 태스크에서 이러한 그룹을 삭제합니다.

#### 작업 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 포털 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
