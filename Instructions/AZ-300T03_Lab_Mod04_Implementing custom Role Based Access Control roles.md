# ID 보호
# 랩: 사용자 지정 RBAC(역할 기반 액세스 제어) 역할 구현
  
### 시나리오
  
Adatum Corporation은 Azure VM 시작 및 중지(할당 취소) 권한을 위임하기 위해 사용자 지정 RBAC 역할을 구현하고자 합니다.


### 목표
  
이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-  사용자 지정 RBAC 역할 정의 

-  사용자 지정 RBAC 역할 할당

### 랩 설정
  
예상 시간: 30분

사용자 이름: **학생**

암호: **Pa55w.rd**


## 연습 1: 사용자 지정 RBAC 역할 정의
  
이 연습의 주요 태스크는 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. RBAC를 통해 위임할 작업 확인

1. Azure AD 테넌트에서 사용자 지정 RBAC 역할 만들기


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. 랩 가상 머신에서 Microsoft Edge를 시작하여 Azure Portal [**http://portal.azure.com**](http://portal.azure.com) 로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell**. 내에서 **PowerShell** 세션을 시작합니다. 

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

    - 구독: 대상 Azure 구독의 이름

    - Cloud Shell 지역: 구독에서 사용할 수 있고 랩 위치에 가장 가까운 Azure 지역의 이름입니다.

    - 리소스 그룹: 새 리소스 그룹 **az3000900-LabRG**의 이름

    - 스토리지 계정: 새 스토리지 계정의 이름

    - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다.

   ```pwsh
   New-AzResourceGroup -Name az3000901-LabRG -Location <Azure region>
   ```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 매개 변수 파일 **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.parameters.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 Ubuntu를 호스트하는 Azure VM을 배포합니다.

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000901-LabRG -TemplateFile $home/azuredeploy09.json -TemplateParameterFile $home/azuredeploy09.parameters.json
   ```

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요. 

1. Azure Portal에서 Cloud Shell 창을 닫습니다.


#### 작업 2: RBAC를 통해 위임할 작업 식별.

1. Azure Portal에서 **az3000901-LabRG** 블레이드로 이동합니다.

1. **az3000901-LabRG** 블레이드에서 **액세스 제어(IAM)**를 클릭합니다.

1. **az3000901-LabRG - 액세스 제어(IAM)** 블레이드에서 **역할**을 클릭합니다.

1. **역할** 블레이드에서 **소유자**를 클릭합니다.

1. **소유자** 블레이드에서 **권한**을 클릭합니다.

1. **권한(미리 보기)** 블레이드에서 **Microsoft Compute**를 클릭합니다.

1. **Microsoft Compute** 블레이드에서 **가상 머신**을 클릭합니다.

1. **가상 머신**블레이드에서 RBAC를 통해 위임할 수 있는 관리 작업 목록을 검토합니다. 여기에는 **가상 머신 할당 취소** 및 **가상 머신 시작** 작업이 포함됩니다.


#### 작업 3: Azure AD 테넌트에서 사용자 지정 RBAC 역할 만들기

1. 랩 컴퓨터에서 **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json** 파일을 열어 내용을 검토합니다.

   ```json
   {
      "Name": "Virtual Machine Operator (Custom)",
      "Id": null,
      "IsCustom": true,
      "Description": "Azure VM을 시작하고 정지(할당 취소)하도록 허용",
      "Actions": [
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/deallocate/action",
          "Microsoft.Compute/virtualMachines/start/action"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. Azure Portal의 Microsoft Edge 창에서 **Cloud Shell** 내의 **PowerShell** 세션을 시작합니다. 

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json**을 홈 디렉터리에 업로드합니다.

1. 클라우드 셸 창에서 다음 명령을 실행하여 **$SUBSCRIPTION\_ID** 자리 표시자를 Azure 구독의 ID 값으로 바꿉니다.

   ```pwsh
   $subscription_id = (Get-AzContext).Subscription.id
   (Get-Content -Path $HOME/customRoleDefinition09.json) -Replace 'SUBSCRIPTION_ID', "$subscription_id" | Set-Content -Path $HOME/customRoleDefinition09.json
   ```
 
1. Cloud Shell 창에서 다음 명령을 실행하여 사용자 지정 역할 정의를 만듭니다.

   ```pwsh
   New-AzRoleDefinition -InputFile $HOME/customRoleDefinition09.json
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 역할이 제대로 생성되었는지 확인합니다.

   ```pwsh
   Get-AzRoleDefinition -Name 'Virtual Machine Operator (Custom)'
   ```

1. Cloud Shell 창을 닫습니다.


> **결과**: 사용자 지정 RBAC 역할을 정의하여 이 연습을 완료해야 합니다.


## 연습 2: 사용자 지정 RBAC 역할 할당 및 테스트
  
이 연습의 주요 태스크는 다음과 같습니다.

1. Azure AD 사용자 만들기

1. RBAC 역할 할당 만들기

1. RBAC 역할 할당 테스트


#### 작업 1: Azure AD 사용자 만들기.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell**. 내에서 **PowerShell** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 대상 Azure AD 테넌트에 대해 명시적으로 인증합니다.

   ```pwsh
   Connect-AzureAD
   ```
   
1. Cloud Shell 창에서 다음 명령을 실행하여 Azure AD DNS 도메인 이름을 확인합니다.

   ```pwsh
   $domainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 새 Azure AD 사용자를 만듭니다.

   ```pwsh
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'lab user0901' -PasswordProfile $passwordProfile -MailNickName 'labuser0901' -UserPrincipalName "labuser0901@$domainName"
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 만든 Azure AD 사용자의 사용자 계정 이름을 확인합니다.

   ```pwsh
   (Get-AzureADUser -Filter "MailNickName eq 'labuser0901'").UserPrincipalName
   ```

1. Cloud Shell 창을 닫습니다.


#### 작업 2: RBAC 역할 할당 만들기
 
1. Azure Portal에서 **az3000901-LabRG** 블레이드로 이동합니다.

1. **az3000901-LabRG** 블레이드에서 **액세스 제어(IAM)**를 클릭합니다.

1. **az3000901-LabRG - 액세스 제어(IAM)** 블레이드에서 **+ 추가**를 클릭하고 **역할 할당 추가** 옵션을 선택합니다.

1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고 **저장**을 클릭합니다.

    - 역할: **가상 머신 운영자 (사용자 지정)**

    - 다음에 대한 액세스 할당: **Azure AD 사용자, 그룹 또는 서비스 주체**

    - 선택: **lab user0901**


#### 작업 3: RBAC 역할 할당 테스트

1. 새 InPrivate Microsoft Edge 창을 시작하고 Azure Portal [**http://portal.azure.com**](http://portal.azure.com) 로 이동한 다음 새로 만든 사용자 계정을 사용해 로그인합니다.

    - 사용자 이름: 이 연습의 첫 번째 태스크에서 확인한 사용자 계정 이름

    - 암호: **Pa55w.rd1234**

1. Azure Portal에서 **리소스 그룹** 블레이드로 이동합니다. 리소스 그룹이 표시되지 않음을 확인합니다. 

1. Azure Portal에서 **모든 리소스** 블레이드로 이동합니다. **az3000901-vm** 및 해당 관리 디스크만 표시됩니다.

1. Azure Portal에서 **az3000901-vm** 블레이드로 이동하여 가상 컴퓨터를 다시 시작해 봅니다. 알림 영역에서 오류 메시지를 검토하고, 현재 사용자에게 이 작업을 수행할 권한이 부여되지 않아 작업이 실패했음을 확인합니다.

1. 가상 머신을 중지하고 작업이 정상적으로 완료되었는지 확인합니다.

> **결과**: 사용자 지정 RBAC 역할을 할당하고 테스트하여 이 연습을 완료해야 합니다.

## 연습 3: 랩 리소스 제거

#### 작업 1: Cloud Shell 열기

1. 포털 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. 필요한 경우 Cloud Shell 창의 왼쪽 상단에 있는 드롭다운 목록을 사용하여 Bash 셸 세션으로 전환하십시오.

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹을 나열합니다.

   ```
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv
   ```

1. 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 다음 태스크에서 이러한 그룹을 삭제합니다.

#### 작업 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 포털 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
