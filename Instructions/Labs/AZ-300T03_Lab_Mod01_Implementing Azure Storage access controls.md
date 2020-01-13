---
lab:
    title: 'Azure Storage 액세스 제어 구현'
    module: '모듈 1: 컴퓨팅 및 스토리지 솔루션 선택'
---

# 스토리지 구현 및 관리
# 랩: Azure Storage 액세스 제어 구현
  
### 시나리오
  
Adatum Corporation은 Azure Storage에 있는 콘텐츠를 보호하려고 합니다.


### 목표
  
이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-  Azure Storage 계정 만들기

-  Azure Storage에 데이터 업로드

-  Azure Storage 액세스 제어 구현

### 랩 설정
  
예상 소요 시간: 30분

사용자 이름: **Student**

암호: **Pa55w.rd**


## 연습 1: Azure Storage 계정 만들기 및 구성
  
이 연습의 주요 태스크는 다음과 같습니다.

1. Azure에서 스토리지 계정 만들기

1. 스토리지 계정의 속성 확인 


#### 태스크 1: Azure에서 스토리지 계정 만들기

1. 랩 가상 머신에서 Microsoft Edge를 시작하여 Azure Portal([**http://portal.azure.com**](http://portal.azure.com))로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.
  
1. Azure Portal에서 다음 설정을 사용하여 새 스토리지 계정을 만듭니다. 

    - 구독: 대상 Azure 구독의 이름

    - 리소스 그룹: 새 리소스 그룹 **az3000201-LabRG**

    - 스토리지 계정 이름: 소문자와 숫자로 구성된 3~24자 사이의 유효하고 고유한 이름

    - 위치: 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름

    - 성능: **Standard**

    - 계정 종류: **스토리지(범용 v1)**

    - 복제: **LRS(로컬 중복 스토리지)**
    
    - 보안 전송 필요: **사용 안 함**

    - 가상 네트워크: **모든 네트워크**
    
    - Blob 일시 삭제: **사용 안 함**

    - 계층 구조 네임스페이스: **사용 안 함**

1. 스토리지 계정이 프로비전될 때까지 기다립니다. 프로비전이 완료되려면 1분 정도 걸립니다.


#### 태스크 2: 스토리지 계정의 속성 확인
  
1. Azure Portal에서 스토리지 계정 블레이드를 열고 **개요** 섹션에서 위치, 복제, 성능 설정 등을 검토합니다.

1. **액세스 키** 블레이드를 표시합니다. 액세스 키 블레이드에서는 key1 및 key2를 비롯한 스토리지 계정 이름 값을 복사할 수 있으며 두 키를 모두 다시 생성할 수 있습니다.

1. **구성** 블레이드를 표시합니다.

1. **구성** 블레이드에서는 **범용 v2** 계정으로 업그레이드를 수행하고 복제 설정을 변경할 수 있습니다. 그러나 성능 설정은 변경할 수 없습니다. 성능 설정은 스토리지 계정을 만들 때만 할당할 수 있습니다.

> **결과**: Azure Storage를 만들고 해당 속성을 검사하여 이 연습을 완료해야 합니다.


## 연습 2: blob 만들기 및 관리
  
이 연습의 주요 태스크는 다음과 같습니다.

1. 컨테이너 만들기

1. Azure Portal을 사용하여 컨테이너에 데이터 업로드

1. SAS 토큰을 사용하여 Azure Storage 계정의 콘텐츠에 액세스


#### 태스크 1: 컨테이너 만들기
  
1. Azure Portal에서 이전 태스크에서 만든 스토리지 계정의 속성이 표시되는 블레이드로 이동합니다.

1. 스토리지 계정 블레이드에서 다음 설정을 사용하여 새 blob 컨테이너를 만듭니다.

    - 이름: **labcontainer**

    - 액세스 유형: **개인**


#### 태스크 2: Azure Portal을 사용하여 컨테이너에 데이터 업로드
  
1. Azure Portal에서 **labcontainer** 블레이드로 이동합니다.

1. **labcontainer** 블레이드에서 다음 파일을 업로드합니다. **C:\\Windows\\ImmersiveControlPanel\\images\\splashscreen.contrast-white_scale-400.png**.  


#### 태스크 3: SAS 토큰을 사용하여 Azure Storage 계정의 콘텐츠에 액세스
  
1. **labcontainer** 블레이드에서 새로 업로드한 blob의 URL을 확인합니다. 

1. Microsoft Edge를 시작하여 해당 URL로 이동합니다.

1. **ResourceNotFound** 오류 메시지가 표시되는지 확인합니다. blob은 개인 컨테이너에 있어서 인증 후에 액세스해야 하므로 이 오류 메시지가 표시되는 것은 정상적인 현상입니다. 

1. Azure Portal이 표시된 Microsoft Edge 창으로 전환한 후 **splashscreen.contrast-white_scale-400.png** 블레이드에서 **SAS 생성** 탭으로 전환합니다. 

1. **SAS 생성** 탭에서 **HTTP** 옵션을 사용하도록 설정하고 blob SAS 토큰 및 해당 URL을 생성합니다.

1. 새 Microsoft Edge 창을 열고 이전 단계에서 생성한 URL로 이동합니다.

1. 이미지가 표시되는지 확인합니다. URL에 포함된 SAS 토큰을 기반으로 blob 액세스 권한이 부여되었으므로 이 메시지가 표시되는것은 정상적인 현상입니다. 

1. 이미지가 표시된 Microsoft Edge 창을 닫습니다.


#### 태스크 4: SAS 토큰 및 저장된 액세스 정책을 사용하여 Azure Storage 계정의 콘텐츠에 액세스

1. Azure Portal에서 **labcontainer** 블레이드로 이동합니다.

1. **labcontainer** 블레이드에서 **labcontainer - 액세스 정책** 블레이드로 이동합니다.

1. 다음 설정을 사용하여 새 정책을 추가합니다.

    - 식별자: **labcontainer-read**

    - 권한: **읽기**

    - 시작 시간: 현재 날짜와 시간

    - 만료 시간: 현재 날짜와 시간 + 24시간

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell**. 내에서 **PowerShell** 세션을 시작합니다. 

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

    - 구독: 대상 Azure 구독의 이름

    - Cloud Shell 지역: 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름

    - 리소스 그룹: **az3000201-LabRG**

    - 스토리지 계정: 새 스토리지 계정의 이름

    - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩의 첫 번째 연습에서 만든 스토리지 계정 리소스를 확인한 다음 변수에 저장합니다.

```pwsh
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName az3000201-LabRG)[0]
```

1. Cloud Shell 창에서 다음 명령을 실행하여 스토리지 계정에 대한 모든 권한을 부여하는 보안 컨텍스트를 설정합니다.

```pwsh
   $keyContext = $storageAccount.Context
```

1. Cloud Shell 창에서 다음 명령을 실행하여 이전 태스크에서 만든 액세스 정책을 기반으로 blob용 SAS 토큰을 만듭니다.

```pwsh
   $sasToken = New-AzStorageBlobSASToken -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Policy labcontainer-read -Context $keyContext
```

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 만든 SAS 토큰을 기반으로 보안 컨텍스트를 설정합니다. 

```pwsh
   $sasContext = New-AzStorageContext $storageAccount.StorageAccountName -SasToken $sasToken
```

1. Cloud Shell 창에서 다음 명령을 실행하여 blob의 속성을 검색합니다. 

```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
```

1. blob에 정상적으로 액세스했는지 확인합니다.

1. Cloud Shell 창을 최소화합니다.


#### 태스크 5: 액세스 정책을 수정하여 SAS 토큰 무효화

1. Azure Portal에서 **labcontainer - 액세스 정책** 블레이드로 이동합니다.

1. 시작 및 만료 시간을 어제 날짜로 설정하여 기존 정책 **labcontainer-read** 를 편집합니다. 

1. Cloud Shell 창을 다시 엽니다. 

1. Cloud Shell 창에서 다음 명령을 다시 실행하여 blob의 속성 검색을 시도합니다. 

```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
```

1. 더 이상 blob에 액세스할 수 없는지 확인합니다.


> **결과**: blob 컨테이너를 만들고, 해당 컨테이너에 파일을 업로드하고, SAS 토큰 및 저장된 액세스 정책을 사용해 액세스 제어를 테스트하여 이 연습을 완료해야 합니다.

## 연습 3: 랩 리소스 제거

#### 작업 1: Cloud Shell 열기

1. 포털 상단에서 **Cloud Shell** 아이콘을 클릭하여 Clould Shell 창을 엽니다.

1. 필요한 경우 Cloud Shell 창의 왼쪽 상단에 있는 드롭다운 목록을 사용하여 Bash 셸 세션으로 전환하십시오.

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹을 나열합니다.

```
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv
```

1. 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인합니다. 이러한 그룹은 다음 작업에서 삭제됩니다.

#### 작업 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

```sh
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. 포털 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
