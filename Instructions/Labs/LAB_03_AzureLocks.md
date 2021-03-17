---
lab:
    title: '03 - Resource Manager 잠금'
    module: '모듈 01 - ID 및 액세스 관리'
---

# 랩 03: Resource Manager 잠금
# 학생 랩 매뉴얼

## 랩 시나리오 

실수로 삭제되거나 변경되는 것을 방지하기 위해 리소스 잠금을 사용하는 방법을 보여주는 개념 증명을 만들어야 합니다. 특히 다음이 필요합니다.

- 읽기 전용 잠금 만들기

- 삭제 잠금 만들기

> 이 랩에 있는 모든 리소스의 경우 **미국 동부** 지역을 사용합니다. 강사에게 이 지역이 수업에 사용할 지역인지 확인합니다. 
 
## 랩 목표

이 랩에서는 다음과 같은 연습을 완료합니다:

- 연습 1: Resource Manager 잠금

### 연습 1: Resource Manager 잠금

#### 예상 시간: 20분

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: 스토리지 계정이 있는 리소스 그룹을 만듭니다.
- 작업 2: 스토리지 계정에 읽기 전용 잠금을 추가합니다. 
- 작업 3: 읽기 전용 잠금을 테스트합니다. 
- 작업 4: ReadOnly 잠금을 제거하고 삭제 잠금을 만듭니다.
- 작업 5: 삭제 잠금을 테스트합니다.

#### 작업 1: 스토리지 계정이 있는 리소스 그룹을 만듭니다.

이 작업에서는 랩에 대한 리소스 그룹 및 스토리지 계정을 만듭니다. 

1. Azure Portal **`https://portal.azure.com/`** 에 로그인합니다.

    >**참고**: 이 랩에 사용 중인 Azure 구독에 Owner 또는 Contributor 역할이 있는 계정을 사용하여 Azure Portal에 로그인합니다.

1. Azure Portal 오른쪽 상단에 있는 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 메시지가 표시되면 **PowerShell** 및 **스토리지 만들기**를 선택합니다.

1. Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **PowerShell**이 선택되었는지 확인합니다.

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 리소스 그룹을 만듭니다(위치 매개 변수 값에 대해 강사와 함께 확인).

    ```powershell
    New-AzResourceGroup -Name AZ500LAB03 -Location 'EastUS'
    ```

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 새로 만든 리소스 그룹에 스토리지 계정을 만듭니다.
    
    ```powershell
    New-AzStorageAccount -ResourceGroupName AZ500LAB03 -Name (Get-Random -Maximum 999999999999999) -Location  EastUS -SkuName Standard_LRS -Kind StorageV2 
    ```

   >**참고**:  스토리지 계정이 생성될 때까지 기다립니다. 이 과정은 몇 분 정도 걸릴 수 있습니다. 

1. Cloud Shell 창을 닫습니다.

#### 작업 2: 스토리지 계정에 읽기 전용 잠금을 추가합니다. 

이 작업에서는 스토리지 계정에 읽기 전용 잠금만 추가합니다. 이를 통해 실수로 삭제하거나 수정하지 않도록 리소스를 보호합니다. 

1. Azure Portal 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **리소스 그룹**을 입력하고 **Enter**키를 누릅니다.

1. **리소스 그룹** 블레이드에서 **AZ500LAB03** 리소스 그룹 항목을 선택합니다.

1. **AZ500LAB03** 리소스 그룹 블레이드의 리소스 목록에서 새 스토리지 계정을 선택합니다. 

1. **설정** 섹션에서 "잠금" 아이콘을 클릭합니다.

1. **+ 추가**를 클릭하고 다음 설정을 지정합니다.

   |설정|값|
   |---|---|
   |잠금 이름|**읽기 전용 잠금**|
   |잠금 유형|**읽기 전용**|

1. **확인**을 클릭합니다. 

   >**참고**:  이제 스토리지 계정은 실수로 삭제 및 수정되지 않도록 보호됩니다.

#### 작업 3: 읽기 전용 잠금 테스트 

1. 스토리지 계정 블레이드의 **설정** 섹션에서 **구성**을 클릭합니다.

1. **보안 전송 필수** 옵션을 **비활성화**로 설정한 다음 **저장**을 클릭합니다.

1. **스토리지 계정을 업데이트하지 못했습니다**라는 알림이 표시되어야 합니다.

1. Azure Portal 상단의 도구 모음에서 **알림** 아이콘을 클릭하고 다음 텍스트와 유사한 알림을 검토합니다. 

	> **"'xxxxxxxx' 스토리지 계정을 업데이트하지 못했습니다. 오류: 'xxxxxxxx' 범위는 다음 범위가 잠겨 있기 때문에 쓰기 작업을 수행할 수 없습니다. '/subscriptions/xxxxx-xxx-xxxx-xxxx-xxxxxxxx/resourceGroups/AZ500LAB03/providers/Microsoft.Storage/storageAccounts/xxxxxxx'. 잠금을 제거하고 다시 시도하세요."**

1. 스토리지 계정의 **구성** 블레이드를 반환하고 **삭제**를 클릭합니다. 

1. 스토리지 계정 블레이드에서 **개요**를 선택하고 **개요** 블레이드에서 **삭제**를 클릭합니다.

1. **스토리지 계정 삭제** 블레이드에서 스토리지 계정 이름을 입력하여 진행할지 확인한 다음 **삭제**를 클릭합니다.

1. 다음 텍스트와 유사한 새로 생성된 알림을 검토합니다. 

	> **"'xxxxxxx' 스토리지 계정을 삭제하지 못했습니다. 오류: 다음 범위가 잠겨 있기 때문에, 'xxxxxxx' 범위는 삭제 작업을 수행할 수 없습니다. '/subscriptions/xxxx-xxxx-xxxx-xxxx-xxxxxx/resourceGroups/AZ500LAB03/providers/Microsoft.Storage/storageAccounts/xxxxxxx'. 잠금을 제거하고 다시 시도하세요."**

   >**참고**:  이제 ReadOnly 잠금으로 인해 실수로 리소스가 삭제 및 수정되지 않음을 확인했습니다.

#### 작업 4: ReadOnly 잠금을 제거하고 삭제 잠금을 만듭니다.

이 작업에서는 스토리지 계정에서 읽기 전용 잠금을 제거하고 잠금 삭제를 만듭니다. 

1. Azure Portal에서, 새로 만들어진 스토리지 계정의 속성을 나타내는 블레이드로 다시 이동합니다.

1. **설정** 섹션에서 **잠금**을 선택합니다.  

1. **잠금** 블레이드에서 **읽기 전용 잠금** 항목의 맨 오른쪽에 있는 **삭제** 아이콘을 클릭합니다.

1. **+ 추가**를 클릭하고 다음 설정을 지정합니다.

   |설정|값|
   |---|---|
   |잠금 이름|**잠금 삭제**|
   |잠금 유형|**삭제**|

1. **확인**을 클릭합니다. 

#### 작업 5: 삭제 잠금을 테스트합니다.

이 작업에서는 삭제 잠금을 테스트합니다. 스토리지 계정을 수정할 수는 있지만 삭제하지는 않아야 합니다. 

1. 스토리지 계정 블레이드의 **설정** 섹션에서 **구성**을 클릭합니다.

1. **보안 전송 필수** 옵션을 **비활성화**로 설정한 다음 **저장**을 클릭합니다.

   >**참고**:  이번에는 변경을 완료해야 합니다.

1. 스토리지 계정 블레이드에서 **개요**를 선택하고 **개요** 블레이드에서 **삭제**를 클릭합니다.

1. **스토리지 계정 삭제** 블레이드에서 스토리지 계정 이름을 입력하여 진행할지 확인한 다음 **삭제**를 클릭합니다.

1. 다음 텍스트와 비슷한 내용의 알림을 검토합니다. 

	> **리소스 또는 부모에 삭제 잠금이 있기 때문에 'xxxxxx'를 삭제할 수 없습니다. 이 리소스를 삭제하려면 먼저 잠금을 제거해야 합니다."**

   >**참고**:  이제 **삭제** 잠금이 구성 변경을 허용하지만 실수로 삭제되지 않도록 하는 것을 확인했습니다.

   >**참고**:  리소스 잠금을 사용하면 실수로 인한/악의적인 가장 중요한 리소스 변경 및/또는 삭제를 추가로 방어할 수 있습니다. **Owner** 역할을 하는 모든 사용자가 리소스 잠금을 제거할 수 있지만 이를 위해서는 의식적인 노력이 필요합니다. 잠금은 역할 기반 액세스 제어를 보완합니다. 

> 결과: 이 연습에서는 Resource Manager 잠금을 사용하여 수정 및 실수로 삭제되지 않도록 리소스를 보호하는 방법을 배웠습니다.

**리소스 정리**

> 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

1. Azure Portal 오른쪽 위의 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 메시지가 표시되면 **다시 연결**을 클릭합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 삭제 잠금을 제거합니다.

    ```powershell
    $storageaccountname = (Get-AzStorageAccount -ResourceGroupName AZ500LAB03).StorageAccountName

    $lockName = (Get-AzResourceLock -ResourceGroupName AZ500LAB03 -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts).Name

    Remove-AzResourceLock -LockName $lockName -ResourceName $storageAccountName  -ResourceGroupName AZ500LAB03 -ResourceType Microsoft.Storage/storageAccounts -Force
    ```

1.  Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 리소스 그룹을 제거합니다.

    ```powershell
    Remove-AzResourceGroup -Name "AZ500LAB03" -Force -AsJob
    ```

1.  **Cloud Shell** 창을 닫습니다. 
