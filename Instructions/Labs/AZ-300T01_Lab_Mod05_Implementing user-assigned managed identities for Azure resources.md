---
lab:
    title: 'Azure 리소스용으로 사용자가 할당한 관리 ID 구현'
    module: '모듈 5: ID 관리'
---

# ID 관리

# 랩: Azure 리소스용으로 사용자가 할당한 관리 ID 구현

### 시나리오

Adatum Corporation에서는 관리 ID를 사용하여 Azure VM에서 실행 중인 애플리케이션을 인증하려고 합니다.

### 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- 사용자가 할당한 관리 ID 만들기 및 구성

- 사용자가 할당한 관리 ID의 기능 유효성 검사

### 랩 설정

예상 소요 시간: 30분

사용자 이름: **Student**

암호: **Pa55w.rd**

## 연습 1: 사용자가 할당한 관리 ID 만들기 및 구성

이 연습의 주요 태스크는 다음과 같습니다.

1. Windows Server 2016 Datacenter를 실행하는 Azure VM 배포

1. 사용자가 할당한 관리 ID 만들기

1. Azure VM에 사용자가 할당한 관리 ID 할당

1. 사용자가 할당한 관리 ID에 RBAC 기반 권한 부여

#### 태스크 1: Windows Server 2016 Datacenter를 실행하는 Azure VM 배포

1. 랩 가상 머신에서 Microsoft Edge를 시작하여 Azure Portal([**http://portal.azure.com**](http://portal.azure.com))로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell** 내에서 **Bash** 세션을 시작합니다.

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

   - 구독: 대상 Azure 구독의 이름

   - Cloud Shell 지역: 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름

   - 리소스 그룹: **az3000500-LabRG**

   - 스토리지 계정: 새 스토리지 계정의 이름

   - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다.

```
   az group create --resource-group az3000501-LabRG --location <Azure region>
```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T01\\Module_05\\azuredeploy05.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 매개 변수 파일 **\\allfiles\\AZ-300T01\\Module_05\\azuredeploy05.parameters.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 Windows Server 2016 Datacenter를 호스트하는 Azure VM을 첫 번째 가상 네트워크에 배포합니다.

```
   az group deployment create --resource-group az3000501-LabRG --template-file azuredeploy05.json --parameters @azuredeploy05.parameters.json
```

   > **참고**: 배포가 완료될 때까지 기다립니다. 배포가 완료되려면 5분 정도 걸립니다.

#### 태스크 2: 사용자가 할당한 관리 ID를 만들어 Azure VM에 할당

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자가 할당한 관리 ID를 만듭니다.

```
   az identity create --resource-group az3000501-LabRG --name az3000501-mi
```

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자가 할당한 관리 ID를 Azure VM에 할당합니다.

```
   az vm identity assign --resource-group az3000501-LabRG --name az3000501-vm --identities az3000501-mi
```

#### 태스크 3: 사용자가 할당한 관리 ID를 참조하는 RBAC 구성

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 이 연습에서 Azure VM을 배포한 Azure 지역의 이름으로 바꿉니다.

```
   az group create --resource-group az3000502-LabRG --location <Azure region>
```

1. Azure Portal에서 **az3000502-LabRG - 액세스 제어(IAM)** 블레이드로 이동합니다.

1. **az3000502-LabRG - 액세스 제어(IAM)** 블레이드에서 새로 만든 사용자가 할당한 관리 ID에 소유자 역할을 할당합니다.

> **결과**: 사용자가 할당한 관리 ID를 만들고 구성하여 이 연습을 완료해야 합니다.

## 연습 2: 사용자가 할당한 관리 ID의 기능 유효성 검사

이 연습의 주요 태스크는 다음과 같습니다.

1. 사용자가 할당한 관리 ID를 통해 인증을 하도록 Azure VM 구성

1. Azure VM에서 사용자가 할당한 관리 ID의 기능 유효성 검사

#### 태스크 1: 사용자가 할당한 관리 ID를 통해 인증을 하도록 Azure VM 구성

1. Azure Portal에서 **az3000501-vm** 블레이드로 이동합니다.

1. 원격 데스크톱을 사용하여 Azure VM에 연결한 후 다음 자격 증명을 입력하여 인증합니다.

   - 사용자 이름: **Student**

   - 암호: **Pa55w.rd1234**

1. 원격 데스크톱 세션이 설정되면 **Administrator: C:\\Windows\\system32\\cmd.exe** 창이 표시됩니다. PowerShell 세션을 시작하려면 명령 프롬프트에서 `PowerShell`을 입력하고 Enter 키를 누릅니다.

1. PowerShell 프롬프트에서 다음 명령을 실행하여 최신 버전의 PowerShellGet 모듈을 설치합니다. 설치를 확인하라는 메시지가 표시되면 Enter 키를 누르세요.

```pwsh
   Install-Module -Name PowerShellGet -Force
```

1. PowerShell 프롬프트에서 다음 명령을 실행하여 최신 버전의 Az 모듈을 설치합니다. 설치를 확인하라는 메시지가 표시되면 **Y** 를 입력하고 Enter 키를 누르세요.

```pwsh
   Install-Module -Name Az -AllowClobber
```

1. `exit`를 입력하고 Enter 키를 눌러 현재 PowerShell 세션을 종료합니다. 그런 다음 명령 프롬프트에 `PowerShell`을 입력하고 Enter 키를 눌러 세션을 다시 시작합니다.

1. PowerShell 프롬프트에서 다음 명령을 실행하여 AzureRM.ManagedServiceIdentity 모듈을 설치합니다.

```pwsh
   Install-Module -Name Az.ManagedServiceIdentity
```

#### 태스크 2: Azure VM에서 사용자가 할당한 관리 ID의 기능 유효성 검사

1. PowerShell 프롬프트에서 다음 명령을 실행하여 사용자가 할당한 관리 ID로 로그인합니다.

```pwsh
   Add-AzAccount -Identity
```

1. PowerShell 프롬프트에서 다음 명령을 실행하여 현재 사용 중인 관리 ID 검색을 시도합니다.

```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
```

1. 오류 메시지를 확인합니다. 해당 메시지에 나와 있는 것처럼 현재 보안 컨텍스트는 대상 리소스에 대한 권한을 충분히 부여하지 않습니다. 이 문제를 해결하려면 Azure Portal로 전환하여 **az3000501-LabRG - 액세스 제어(IAM)** 블레이드로 이동합니다.

1. **az3000501-LabRG - 액세스 제어(IAM)** 블레이드에서 사용자가 할당한 관리 ID **az3000501-mi**에 참가자 역할을 할당합니다.

1. 원격 데스크톱 세션으로 다시 전환한 후 PowerShell 프롬프트에서 다음 명령을 실행하여 현재 사용 중인 관리 ID 검색을 시도합니다.

```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
```

   > **참고**: 권한이 부족하다는 오류 메시지가 표시되면 PowerShell 프롬프트에서 다음 명령을 실행하십시오.
   
```pwsh
   Remove-AzAccount
```
   
계속해서 다음 명령을 실행하십시오.
   
```pwsh
   Add-AzAccount -Identity
```
   
1. PowerShell 프롬프트에서 다음 명령을 실행하여 변수에 위치를 저장합니다.

```pwsh
   $location = (Get-AzResourceGroup -Name az3000502-LabRG).Location
```

1. PowerShell 프롬프트에서 다음 명령을 실행하여 공용 IP 주소 리소스를 만듭니다.

```pwsh
   New-AzPublicIpAddress -Name az3000502-pip -ResourceGroupName az3000502-LabRG -AllocationMethod Dynamic -Location $location
```

1. 명령이 정상적으로 완료되었는지 확인합니다.

> **결과**: 사용자 정의 관리 ID의 기능 유효성을 검사하여 이 연습을 완료해야 합니다.

## 연습 3: 랩 리소스 제거

#### 태스크 1: Cloud Shell 열기

1. Portal 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹의 목록을 표시합니다.

```
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv
```

1. 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 다음 태스크에서 이러한 그룹을 삭제합니다.

#### 태스크 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

```sh
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. Portal 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
