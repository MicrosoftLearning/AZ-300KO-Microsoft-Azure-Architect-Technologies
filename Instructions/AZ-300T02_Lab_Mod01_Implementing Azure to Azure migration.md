# Azure로의 서버 마이그레이션 평가 및 수행

# 랩: Azure 간 마이그레이션 구현

### 시나리오

Adatum Corporation에서 기존 Azure VM을 다른 지역으로 마이그레이션하려고 합니다.

### 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure Site Recovery Vault 구현

- Azure 지역 간에 Azure VM 마이그레이션

### 랩 설정

예상 시간: 45분

사용자 이름: **학생**

암호: **Pa55w.rd**

## 연습 1: Azure Site Recovery를 사용하여 Azure VM 마이그레이션을 위한 필수 구성 요소 구현

이 연습의 주요 작업은 다음과 같습니다.

1. 마이그레이션할 Azure VM 배포

1. Azure Recovery Services 자격 증명 모음 만들기

#### 작업 1: 마이그레이션할 Azure VM 배포

1. 랩 가상 머신에서 Microsoft Edge를 시작하여 Azure Portal [**http://portal.azure.com**](http://portal.azure.com) 로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell**. 내에서 **PowerShell** 세션을 시작합니다.

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

   - 구독: 대상 Azure 구독의 이름

   - Cloud Shell 지역: 구독에서 사용할 수 있고 랩 위치에 가장 가까운 Azure 지역의 이름입니다.

   - 리소스 그룹: 새 리소스 그룹 **az3000600-LabRG**의 이름

   - 스토리지 계정: 새 스토리지 계정의 이름

   - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다.

   ```pwsh
   New-AzResourceGroup -Name az3000601-LabRG -Location <Azure region>
   ```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 매개 변수 파일 **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.parameters.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 홈 디렉터리로 전환하세요.
   ```pwsh
   cd $home
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 Windows Server 2016 Datacenter 를 호스트하는 Azure VM을 배포합니다.

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000601-LabRG -TemplateFile azuredeploy06.json -TemplateParameterFile azuredeploy06.parameters.json
   ```

   > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요.


#### 작업 2: Azure Site Recovery Services 자격 증명 모음 구현

1. Azure Portal에서 다음 설정을 사용하여 **Recovery Services 자격 증명 모음**의 인스턴스를 만듭니다.

   - 이름: **vaultaz3000602**

   - 구독: 대상 Azure 구독의 이름

   - 리소스 그룹: 새 리소스 그룹 **az3000602-LabRG**의 이름

   - 위치: 구독에서 사용 가능하며 이전 태스크에서 Azure VM을 배포한 지역과는 **다른** Azure 지역의 이름

1. 자격 증명 모음이 프로비전될 때까지 기다립니다. 약 1분이 소요됩니다.

> **결과**: 마이그레이션할 Azure VM과, 해당 Azure VM 의 마이그레이션된 디스크 파일을 호스트할 Azure Site Recovery 자격 증명 모음을 만들어 이 연습을 완료해야 합니다.

## 연습 2: Azure Site Recovery를 사용하여 Azure 지역 간 Azure VM 마이그레이션

이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM 복제 구성

1. Azure VM 복제 설정 검토

1. Azure VM 의 복제를 사용하지 않도록 설정하고 Azure Recovery Services 자격 증명 모음 삭제

#### 작업 1: Azure VM 복제 구성

1. Azure Portal에서 새로 프로비전된 Azure Recovery Services 자격 증명 모음의 블레이드로 이동합니다.

1. Recovery Services 자격 증명 모음 블레이드에서 **+ 복제** 단추를 클릭합니다.

1. 다음 설정을 지정하여 복제를 사용하도록 설정합니다.

   - 원본: **Azure**

   - 원본 위치: 이 랩의 이전 연습에서 Azure VM 을 배포한 것과 동일한 Azure 지역

   - Azure 가상 머신 배포 모델: **Resource Manager**

   - 원본 리소스 그룹: **az3000601-LabRG**

   - 가상 머신: **az3000601-vm**

   - 대상 위치: 구독에서 사용 가능하며 이전 태스크에서 Azure VM을 배포한 지역과는 다른 Azure 지역의 이름

   - 대상 리소스 그룹: **(신규) az3000601-LabRG-asr**

   - 대상 가상 네트워크: **(신규) az3000601-vnet-asr**

   - 캐시 스토리지 계정: 기본 설정 적용

   - 복제본 관리 디스크: **(신규) 프리미엄 디스크 1개, 표준 디스크 0개**

   - 대상 가용성 집합: **해당 없음**

   - 복제 정책: **새로 만들기**

   - 이름: **12-hour-retention-policy**

   - 복구 지점 보존 기간: **12시간**

   - 앱 일치 스냅샷 빈도: **6시간**

   - 다중 VM 일관성: **아니요**

1. 대상 리소스 만들기를 시작합니다.

1. 복제를 사용하도록 설정합니다.

   > **참고**: 복제를 사용하도록 설정하는 작업이 완료될 때까지 기다립니다. 그런 후에 다음 태스크를 진행합니다.

#### 작업 2: Azure VM 복제 설정 검토

1. Azure Portal의 Azure Site Recovery 자격 증명 모음 블레이드에서 Azure VM **az3000601-vm**을 나타내는 복제된 항목 블레이드로 이동합니다.

2. 복제된 항목 블레이드에서 **상태**, **사용 가능한 최신 복구 지점** 및 **장애 조치 (Failover) 준비** 섹션을 검토합니다. 도구 모음에서 **장애 조치(Failover)** 및 **테스트 장애 조치(Failover)** 항목을 적어 둡니다. 아래쪽의 **인프라 보기**로 스크롤합니다.

3. 시간이 되면 Azure VM 의 상태가 **보호**로 변경될 때까지 기다립니다. 이 경우 15-20 분이 더 걸릴 수 있습니다. 이 시점에서 **충돌 일치** 및 **앱 일치** 복구 지점의 값을 점검합니다. **RPO**를 확인하려면 테스트 장애 조치(failover)를 수행해야 합니다.

#### 작업 3: Azure VM 의 복제를 사용하지 않도록 설정하고 Azure Recovery Services 자격 증명 모음 삭제

1. Azure Portal에서 Azure VM **az3000601-vm**의 복제를 사용하지 않도록 설정합니다.

2. 복제가 사용하지 않도록 설정될 때까지 기다립니다.

3. Azure Portal에서 Recovery Services 자격 증명 모음을 삭제합니다.

   > **참고**: 자격 증명 모음을 삭제하려면 먼저 복제된 항목이 제거되었는지 확인해야 합니다.

> **결과**: Azure VM의 자동 복제를 구현하여 이 연습을 완료해야 합니다.

## 연습 3: 랩 리소스 제거

#### 작업 1: Cloud Shell 열기

1. 포털 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. 필요한 경우 Cloud Shell 창의 왼쪽 상단에 있는 드롭다운 목록을 사용하여 Bash 셸 세션으로 전환하십시오.

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹을 나열합니다.

   ```
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv
   ```

1. 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 다음 태스크에서 이러한 그룹을 삭제합니다.

#### 작업 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 포털 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
