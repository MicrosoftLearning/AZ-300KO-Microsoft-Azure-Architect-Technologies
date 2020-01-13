---
lab:
    title: '사용자 지정 Azure VM 이미지 구현'
    module: '모듈 3: VM(가상 머신) 배포 및 관리'
---

# VM(가상 머신) 배포 및 관리

# 랩: 사용자 지정 Azure VM 이미지 구현

### 시나리오

Adatum Corporation에서 사용자 지정 Azure VM 이미지를 만들려고 합니다.

### 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- HashiCorp Packer를 사용하여 사용자 지정 VM 이미지 만들기

- 사용자 지정 이미지를 기반으로 Azure VM 배포

### 랩 설정

예상 소요 시간: 45분

인터페이스: **BASH 모드에서 Azure Cloud Shell 사용**

## 연습 1: 사용자 지정 이미지 만들기

이 연습의 주요 태스크는 다음과 같습니다.

1. Packer 템플릿 구성

1. Packer 기반 이미지 작성

#### 태스크 1: Packer 템플릿 구성

1. 랩 가상 머신에서 Microsoft Edge를 시작하여 Azure Portal([**http://portal.azure.com**](http://portal.azure.com))로 이동한 다음 대상 Azure 구독에서 소유자 역할이 지정된 Microsoft 계정을 사용하여 로그인합니다.

1. Microsoft Edge 창에 표시된 Azure Portal을 통해 **Cloud Shell** 내에서 **Bash** 세션을 시작합니다.

1. **탑재된 스토리지가 없음** 메시지가 표시되면 다음 설정을 사용하여 스토리지를 구성합니다.

   - 구독: 대상 Azure 구독의 이름

   - Cloud Shell 지역: 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름

   - 리소스 그룹: **az3000300-LabRG**

   - 스토리지 계정: 새 스토리지 계정의 이름

   - 파일 공유: 새 파일 공유의 이름

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만들고 변수에 JSON 출력을 저장합니다. 여기서 `<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다.

```sh
   RG=$(az group create --name az3000301-LabRG --location <Azure region>)
```   
   > **참고**: Azure 지역을 나열하려면 `az account list-locations --output table`을 실행하십시오.

1. Cloud Shell 창에서 다음 명령을 실행하여 Packer에서 사용할 서비스 주체를 만들고 변수에 JSON 출력을 저장합니다.

```sh
   AAD_SP=$(az ad sp create-for-rbac)
```

1. Cloud Shell 창에서 다음 명령을 실행하여 서비스 주체 appId의 값을 검색해 변수에 저장합니다.

```sh
   CLIENT_ID=$(echo $AAD_SP | jq -r .appId)
```

1. Cloud Shell 창에서 다음 명령을 실행하여 서비스 주체 password의 값을 검색해 변수에 저장합니다.

```sh
   CLIENT_SECRET=$(echo $AAD_SP | jq -r .password)
```

1. Cloud Shell 창에서 다음 명령을 실행하여 서비스 주체 테넌트 ID의 값을 검색해 변수에 저장합니다.

```sh
   TENANT_ID=$(echo $AAD_SP | jq -r .tenant)
```

1. Cloud Shell 창에서 다음 명령을 실행하여 구독 ID의 값을 검색해 변수에 저장합니다.

```sh
   SUBSCRIPTION_ID=$(az account show --query id | tr -d '"')
```

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹 위치의 값을 검색해 변수에 저장합니다.

```sh
   LOCATION=$(echo $RG | jq -r .location)
```

1. Cloud Shell 창에서 Packer 템플릿 **\\allfiles\\AZ-300T01\\Module_03\\template03.json** 을 홈 디렉터리에 업로드합니다. 파일을 업로드하려면 Cloud Shell 창에서 위쪽 및 아래쪽 화살표가 있는 문서 아이콘을 클릭합니다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 **client_id** 매개 변수의 값에 해당하는 자리 표시자를 Packer 템플릿의 **\$CLIENT_ID** 변수 값으로 바꿉니다.

```sh
   sed -i.bak1 's/"$CLIENT_ID"/"'"$CLIENT_ID"'"/' ~/template03.json
```

1. Cloud Shell 창에서 다음 명령을 실행하여 **client_secret** 매개 변수의 값에 해당하는 자리 표시자를 Packer 템플릿의 **\$CLIENT_SECRET** 변수 값으로 바꿉니다.

```sh
   sed -i.bak2 's/"$CLIENT_SECRET"/"'"$CLIENT_SECRET"'"/' ~/template03.json
```

1. Cloud Shell 창에서 다음 명령을 실행하여 **tenant_id** 매개 변수의 값에 해당하는 자리 표시자를 Packer 템플릿의 **\$TENANT_ID** 변수 값으로 바꿉니다.

```sh
   sed -i.bak3 's/"$TENANT_ID"/"'"$TENANT_ID"'"/' ~/template03.json
```

1. Cloud Shell 창에서 다음 명령을 실행하여 **subscription_id** 매개 변수의 값에 해당하는 자리 표시자를 Packer 템플릿의 **\$SUBSCRIPTION_ID** 변수 값으로 바꿉니다.

```sh
   sed -i.bak4 's/"$SUBSCRIPTION_ID"/"'"$SUBSCRIPTION_ID"'"/' ~/template03.json
```

1. Cloud Shell 창에서 다음 명령을 실행하여 **location** 매개 변수의 값에 해당하는 자리 표시자를 Packer 템플릿의 **\$LOCATION** 변수 값으로 바꿉니다.

```sh
   sed -i.bak5 's/"$LOCATION"/"'"$LOCATION"'"/' ~/template03.json
```

#### 태스크 2: Packer 기반 이미지 작성

1. Cloud Shell 창에서 다음 명령을 실행하여 Packer 기반 이미지를 작성합니다.

```sh
   packer build template03.json
```

1. 작성이 완료될 때까지 작성 진행률을 모니터링합니다.

   > **참고**: 작성 프로세스는 10분 정도 걸릴 수 있습니다.

> **결과**: Packer 템플릿을 다운로드한 다음 해당 템플릿으로 사용자 지정 이미지를 작성하여 이 연습을 완료해야 합니다.

## 연습 2: 사용자 지정 이미지 배포

이 연습의 주요 태스크는 다음과 같습니다.

1. 사용자 지정 이미지를 기반으로 Azure VM 배포

1. Azure VM 배포 유효성 검사

#### 태스크 1: 사용자 지정 이미지를 기반으로 Azure VM 배포

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자 지정 이미지를 기반으로 Azure VM을 배포합니다.

```sh
   az vm create --resource-group az3000301-LabRG --name az3000301-vm --image az3000301-image --admin-username student --generate-ssh-keys
```

1. 배포가 완료될 때까지 기다립니다.

   > **참고**: 배포 프로세스는 3분 정도 걸릴 수 있습니다.

1. 배포가 완료되면 Cloud Shell 창에서 다음 명령을 실행하여 TCP 포트 80에서 새로 배포된 VM으로의 인바운드 트래픽을 허용합니다.

```sh
   az vm open-port --resource-group az3000301-LabRG --name az3000301-vm --port 80
```

#### 태스크 2: Azure VM 배포 유효성 검사

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 배포한 Azure VM과 연결된 IP 주소를 확인합니다.

```sh
   az network public-ip show --resource-group az3000301-LabRG --name az3000301-vmPublicIP --query ipAddress
```

1. Microsoft Edge를 시작하고 이전 단계에서 확인한 IP 주소로 이동합니다.

1. Microsoft Edge에 **nginx에 오신 것을 환영합니다!** 페이지가 표시되는지 확인합니다.

> **결과**: 사용자 지정 이미지를 기반으로 Azure VM을 배포하고 배포의 유효성을 검사하여 이 연습을 완료해야 합니다.

## 연습 3: 랩 리소스 제거

#### 태스크 1: Cloud Shell 열기

1. Portal 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 모든 리소스 그룹의 목록을 표시합니다.

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv
```

1. 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 다음 태스크에서 이러한 그룹을 삭제합니다.

#### 태스크 2: 리소스 그룹 삭제

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 이 랩에서 만든 리소스 그룹을 삭제합니다.

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. Portal 하단의 **Cloud Shell** 프롬프트를 닫습니다.

> **결과**: 이 연습에서는 이 랩에서 사용한 리소스를 제거했습니다.
