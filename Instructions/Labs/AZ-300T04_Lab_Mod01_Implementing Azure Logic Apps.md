---
lab:
    title: 'Azure 논리 앱 구현'
    module: '모듈 1: Azure App Service 웹앱 만들기'
---

# 애플리케이션 서비스 구현 및 관리
# 랩: Azure Logic Apps 구현
  
### 시나리오
  
Adatum Corporation은 리소스 그룹의 변경 사항에 대한 사용자 지정 모니터링을 구현하려고 합니다.


### 목표
  
이 랩을 완료하면 다음과 같은 역량을 갖추게 됩니다.

-  Azure 논리 앱 만들기 

-  Azure 논리 앱 및 Azure Event Grid의 통합 구성

### 랩 설정
  
예상 시간: 45분

사용자 이름: **학생**

암호: **Pa55w.rd**


## 연습 1: Azure 스토리지 계정 및 Azure 논리 앱으로 구성된 랩 환경 설정 
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure 스토리지 계정 만들기

1. Azure 논리 앱 만들기 

1. Azure AD 서비스 주체 만들기 

1. Azure AD 서비스 주체에 reader 역할 할당 

1. Microsoft.EventGrid 리소스 공급자 등록 


#### 작업 1: Azure에서 스토리지 계정 만들기

1. 실습 가상 머신에서 Microsoft Edge를 시작하고 [**http://portal.azure.com**](http://portal.azure.com)에서 Azure Portal을 찾아보고 대상 Azure 구독에서 소유자 역할이 있는 Microsoft 계정을 사용하여 로그인합니다.
  
1. Azure Portal에서 다음 설정을 사용하여 새 스토리지 계정을 만듭니다. 

    - 구독: 대상 Azure 구독의 이름

    - 리소스 그룹: **az3000701-LabRG**라는 새 리소스 그룹

    - 스토리지 계정 이름: 소문자와 숫자로 구성된 3~24자 사이의 유효하고 고유한 이름

    - 위치: 구독에서 사용할 수 있고 랩 위치에 가장 가까운 Azure 지역의 이름입니다.

    - 성능: **표준**

    - 계정 종류: **저장소(범용 v1)**

    - 복제: **LRS(로컬 중복 저장소)**

    - 필요한 보안 전송: **활성화**

    - 가상 네트워크: **모든 네트워크**

    - 계층 구조 네임스페이스: **비활성화**

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업으로 진행합니다. 


#### 작업 2: Azure 논리 앱 만들기 
 
1. Azure Portal에서 다음 설정을 사용하여 **논리 앱** 의 인스턴스를 만듭니다.

    - 이름:**logicapp3000701**

    - 구독: 대상 Azure 구독의 이름

    - 리소스 그룹: 새 리소스 그룹 **az3000702-LabRG**의 이름

    - 위치: 이전 작업에서 스토리지 계정을 배포한 것과 동일한 Azure 지역

    - 로그 분석: **꺼짐**

1. 앱이 프로비전될 때까지 기다립니다. 약 1분 정도 걸릴 것입니다. 


#### 작업 3: Azure AD 서비스 주체 만들기 

1. Azure Portal의 Microsoft Edge 창에서 **Cloud Shell** 내에서 **PowerShell** 세션을 시작합니다. 

1. **마운트된 저장소가 없습니다** 라는 메시지가 표시되면 다음 설정을 사용하여 저장소를 구성합니다.

    - 구독: 대상 Azure 구독의 이름

    - Cloud Shell 지역: 구독에서 사용할 수 있고 랩 위치에 가장 가까운 Azure 지역의 이름입니다.

    - 리소스 그룹: 새 리소스 그룹 **az3000700-LabRG** 의 이름

    - 스토리지 계정: 새 스토리지 계정의 이름

    - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음을 실행하여 이 작업의 후속 단계에서 만든 서비스 주체와 연결할 새 Azure AD 응용 프로그램을 만듭니다.

```pwsh
   $password = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString -Force -AsPlainText -String $password
   $aadApp30007 = New-AzADApplication -DisplayName 'aadApp30007' -HomePage 'http://aadApp30007' -IdentifierUris 'http://aadApp30007' -Password $securePassword
```

1. Cloud Shell 창에서 다음을 실행하여 이전 단계에서 만든 응용 프로그램과 연결된 새 Azure AD 서비스 주체를 만듭니다.

```pwsh
   New-AzADServicePrincipal -ApplicationId $aadApp30007.ApplicationId.Guid
```

1. **New-AzADServicePrincipal** 명령의 출력에서 **ApplicationId** 속성의 값을 기록합니다. 이 랩의 다음 연습에서 이 항목이 필요합니다.

1. Cloud Shell 창에서 다음을 실행하여 현재 Azure 구독의 **ID** 속성 값과  해당 구독과 연결된 Azure AD 테넌트의 **TenantId** 속성 값을 식별합니다(이 랩의 다음 연습에도 필요합니다):

```pwsh
   Get-AzSubscription
```

1. Cloud Shell 창을 닫습니다.


#### 작업 4: Azure AD 서비스 주체에 reader 역할 할당 

1. Azure Portal에서 Azure 구독의 속성을 표시하는 블레이드로 이동합니다. 

1. Azure 구독 블레이드에서 **IAM(액세스 제어)** 을 클릭합니다.

1. **aadApp30007** 서비스 주체에 Azure 구독의 범위 내에서 **Reader** 역할을 할당합니다.


#### 작업 5: Microsoft.EventGrid 리소스 공급자 등록

1. Azure Portal의 Microsoft Edge 창에서 **Cloud Shell** 내에서 **PowerShell** 세션을 다시 엽니다. 

1. Cloud Shell 창에서 다음을 실행하여 Microsoft.EventGrid 리소스 공급자를 등록합니다.

```pwsh
   Register-AzResourceProvider -ProviderNamespace Microsoft.EventGrid
```

1. Cloud Shell 창을 닫습니다.


> **결과**: 이 연습을 완료한 후 스토리지 계정, 이 랩의 다음 연습에서 구성할 논리 앱 및 해당 구성 중에 참조할 Azure AD 서비스 주체를 만들었습니다.


## 연습 2: Azure 논리 앱을 구성하여 리소스 그룹에 대한 변경 내용을 모니터링합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure 논리 앱에 트리거 추가

1. Azure 논리 앱에 작업 추가

1. Azure 논리 앱의 콜백 URL 식별

1. 이벤트 구독 구성

1. 논리 앱 테스트


#### 작업 1: Azure 논리 앱에 트리거 추가

1. Azure Portal에서 새로 프로비전된 Azure 논리 앱의 **논리 앱 디자이너** 블레이드로 이동합니다.

1. **빈 논리 앱**을 클릭합니다. 이렇게 하면 빈 디자이너 작업 영역이 생성되고 작업 영역에 추가할 커넥터 및 트리거 목록이 표시됩니다.

1. **Event Grid** 트리거를 검색하고 결과 목록에서 **리소스 이벤트가 발생하는 시기(미리 보기)** Azure Event Grid 트리거를 클릭하여 디자이너 작업 영역에 추가합니다.

1. **Azure Event Grid** 타일에서 **서비스 주체로 연결** 링크를 클릭하고 다음 값을 지정한 다음 **만들기**를 클릭합니다.

    - 연결 이름: **egc30007**

    - 클라이언트 ID: 이전 연습에서 식별한 **ApplicationId** 속성

    - 클라이언트 암호: **Pa55w.rd1234**

    - 테넌트: 이전 연습에서 식별한 **TenantId** 속성

1. **리소스 이벤트가 나타나는 시기** 타일에서 다음 값을 지정합니다.

    - 구독: 이전 연습에서 식별한 구독 **Id** 속성

    - 리소스 종류: **Microsoft.Resources.resourceGroups**

    - 리소스 이름: **/subscriptions/*subscriptionId*/resourceGroups/az3000701-LabRG** 에서 ***subscriptionId***는 이전 연습에서 식별한 구독 **ID** 속성입니다.

    - 이벤트 유형 항목 - 1: **Microsoft.Resources.ResourceWriteSuccess**

    - 이벤트 유형 항목 - 2: **Microsoft.Resources.ResourceDeleteSuccess**

1. **새 매개 변수 추가**를 클릭하고 **구독 이름**을 선택합니다.

1. **구독 이름** 텍스트 상자에서 **event-subscription-az3000701** 을 입력하고 **저장** 을 클릭합니다.

#### 작업 2: Azure 논리 앱에 작업 추가

1. Azure Portal의 새로 프로비전된 Azure 논리 앱의 논리 앱 디자이너 블레이드에서 **+ 새 단계** 를 클릭합니다. 

1. **작업 선택** 창의 **커넥터 및 작업 검색** 텍스트 상자에서 **Outlook** 을 입력합니다.

1. 결과 목록에서 **Outlook.com** 을 클릭합니다. 

1. **Outlook.com** 작업 목록에서 **Outlook.com - 이메일 보내기** 를 클릭합니다.

1. **Outlook.com** 창에서 **로그인** 을 클릭합니다. 

1. 메시지가 표시되면 이 랩에서 사용 중인 Microsoft 계정을 사용하여 인증합니다. 

1. Outlook 리소스에 액세스할 수 있는 Azure 논리 앱 사용 권한을 부여하는 데 동의하라는 메시지가 표시되면 **예** 를 클릭합니다.

1. **이메일 보내기** 창에서 다음 설정을 지정하고 **저장** 을 클릭합니다.

    - 받는 사람: Microsoft 계정의 이름

    - 제목: **리소스 업데이트:**를 입력하고 **이메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠** 행에서 **제목** 을 클릭합니다.

    - 본문: **리소스 그룹:**을 입력하고  **이메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠** 행에서 **주제** 를 클릭하고, **이벤트 유형:** 을 입력하며, **이메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠**  행에서 **이벤트 유형** 을 클릭하고, **이벤트 ID:** 를 입력합니다. **이메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠** 행에서 **ID** 를 클릭하고, **이벤트 시간:** 을 입력합니다. **이메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠** 행에서 **이벤트 시간** 을 클릭합니다.


#### 작업 3: Azure 논리 앱의 콜백 URL 식별

1. Azure Portal에서 **logicapp3000701** 블레이드로 이동한 다음 **개요** 섹션에서 **트리거 기록 보기** 를 클릭합니다. **금지된** 오류 메시지를  무시합니다.

1. **When_a_resource_event_occurs** 블레이드에서 **콜백 url [POST]** 텍스트 상자의 값을 복사합니다.


#### 작업 4: 이벤트 구독 구성

1. Azure Portal에서 **az3000701-LabRG** 리소스 그룹으로 이동한 다음 세로 메뉴에서 **이벤트** 를 클릭합니다.

1. **az3000701-LabRG - 이벤트** 블레이드에서 **시작**을 선택하고 **웹 후크**를 클릭합니다.

1. **이벤트 구독 만들기** 블레이드에서 **이벤트 유형으로 필터링** 드롭다운 목록에서 **리소스 쓰기 성공** 및 **리소스 삭제 성공** 옆에 있는 확인란만 선택되어 있는지 확인합니다.

1. **엔드포인트 유형** 드롭다운 목록에서 **웹 후크**가 선택되었는지 확인하고 **엔드포인트 선택** 링크를 클릭합니다. 

1. **웹 후크 선택** 블레이드에서 **구독자 엔드포인트** 에서 이전 작업에서 복사한 Azure 논리 앱의 **콜백 url [POST]** 의 값을 붙여넣은 다음 **선택 확인** 을 클릭합니다.

1. **이벤트 구독 세부 정보** 섹션 내의 **이름** 텍스트 상자에서 **event-subscription-az3000701** 을 입력합니다.

1. **만들기** 를 클릭합니다.


#### 작업 5: 논리 앱 테스트 

1. Azure Portal에서 **az3000701-LabRG** 리소스 그룹으로 이동한 다음 세로 메뉴에서 **개요** 를 클릭합니다. 

1. 리소스 목록에서 첫 번째 연습에서 만든 Azure 스토리지 계정을 클릭합니다.

1. 스토리지 계정 블레이드의 세로 메뉴에서 **구성** 을 클릭합니다.

1. 구성 블레이드에서 **보안 전송 필수** 설정을 **사용 안 함**으로 설정하고 **저장**을 클릭합니다.

1. **logicapp3000701** 블레이드로 이동하여 **새로 고침** 을 클릭하고 **실행 기록** 에 Azure 스토리지 계정의 구성 변경에 해당하는 항목이 포함되어 있는지 확인합니다

1. 이 연습에서 구성한 이메일 계정의 받은 편지함으로 이동하여 논리 앱으로 생성한 이메일이 포함되어 있는지 확인합니다.

> **결과**: 이 연습을 완료한 후 리소스 그룹에 대한 변경 내용을 모니터링하도록 Azure 논리 앱을 구성했습니다.


## 연습 3: 랩 리소스 제거

#### 작업 1: Cloud Shell 열기

1. 포털 상단에서 **Cloud Shell** 아이콘을 클릭하여 Clould Shell 창을 엽니다.

1. 필요한 경우 Cloud Shell 창의 왼쪽 상단에 있는 드롭다운 목록을 사용하여 Bash 셸 세션으로 전환하십시오.

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹을 나열합니다.

```
   az group list --query "[?starts_with(name,'az30007')]".name --output tsv
```

1. 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인합니다. 이러한 그룹은 다음 작업에서 삭제됩니다.

#### 작업 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

```sh
   az group list --query "[?starts_with(name,'az30007')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. 포털 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
