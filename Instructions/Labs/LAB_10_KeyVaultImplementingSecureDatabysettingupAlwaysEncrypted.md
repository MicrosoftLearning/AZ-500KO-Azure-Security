---
lab:
    title: '10 - Key Vault (Always Encrypted를 설정하여 보안 데이터 구현)'
    module: '모듈 03 – 보안 데이터 및 애플리케이션'
---

# 랩 10: Key Vault(Always Encrypted를 설정하여 보안 데이터 구현)
# 학생 랩 매뉴얼

## 랩 시나리오

Always Encrypted 기능에 대한 Azure SQL Database 지원을 사용하는 개념 증명 애플리케이션을 만들어야 합니다. 이 시나리오에 사용된 모든 비밀과 키는 키 자격 증명 모음에 저장되어야 합니다. 애플리케이션은 보안 태세를 향상시키기 위해 Azure AD(Azure Active Directory)에 등록해야 합니다. 이러한 목표를 달성하기 위해 개념 증명에는 다음이 포함되어야 합니다.

- Azure Key Vault를 만들어 자격 증명 모음에 키 및 비밀을 저장합니다.
- 항상 암호화를 사용하여 SQL 데이터베이스를 만들고 데이터베이스 테이블의 열 콘텐츠를 암호화합니다.

> 이 랩의 모든 리소스에 대해, **미국 동부** 지역을 사용하고 있습니다. 강사에게 이 영역이 수업에 사용할 영역인지 확인합니다. 

## 랩 목표

이 랩에서는 다음의 연습을 완료합니다.

- 연습 1: 키와 비밀로 키 자격 증명 모음 구성
- 연습 2: 암호화를 위한 키 자격 증명 모음을 사용하여 시연할 애플리케이션 만들기

## 랩 파일:

- **\\Allfiles\\Labs\\10\\az-500-10_azuredeploy.json**
- **\\Allfiles\\Labs\\10\\az-500-10_azuredeploy.parameters.json**
- **\\Allfiles\\Labs\\10\\program.cs**
 
### 연습 1: 키와 비밀로 키 자격 증명 모음 구성

### 예상 시간: 60분

> 이 랩의 모든 리소스는 **미국 동부** 지역을 사용하고 있습니다. 강사에게 이 지역을 수업에서 사용하는지 확인합니다. 

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: Visual Studio 2019 및 SQL Server Management Studio를 통해 Azure VM 배포 및 구성
- 작업 2: 키 자격 증명 모음 만들기 및 구성
- 작업 3: 키 자격 증명 모음에 키를 추가합니다.
- 작업 4: 키 자격 증명 모음에 암호 추가

#### 작업 1: (선택적 필수 구성 요소) Visual Studio 2019 및 SQL Server Management Studio를 통해 Azure VM 배포 및 구성

>**참고**: Visual Studio 2019와 SQL Server Management 스튜디오가 모두 설치된 랩 공급자를 사용하는 경우 이 단계를 건너뛰고 작업 2로 바로 이동합니다.

이 작업에서는 Azure VM을 배포하고, 연결하고, Visual Studio 2019 및 SSMS(SQL Server Management Studio)를 다운로드 및 설치합니다.

1. Azure Portal(`https://portal.azure.com/`)에 로그인합니다.

    >**참고**: 이 랩에 사용 중인 Azure 구독에 Owner 또는 Contributor 역할이 있는 계정을 사용하여 Azure Portal에 로그인합니다.

1. Azure Portal에서 Azure Portal 페이지 상단의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **사용자 지정 템플릿 배포**를 입력하고 **Enter**키를 누릅니다.

1. **사용자 지정 배포** 블레이드에서 **편집기에서 사용자 고유의 탬플릿 빌드** 옵션을 클릭합니다.

1. **템플릿 편집** 블레이드에서 **파일 로드**를 클릭하고 **\\Allfiles\\Lab\\\10\\az-500-10_azuredeploy.json** 파일을 찾아 **열기**를 클릭합니다.

1. **템플릿 편집** 블레이드에서 **저장**을 클릭합니다.

1. **사용자 지정 배포** 블레이드로 돌아가서 **매개 변수 편집**을 클릭합니다.

1. **매개 변수 편집** 블레이드에서 **로드 파일**을 클릭하고 **\\Allfiles\\Lab\\10\\az-500-10_azuredeploy.parameters.json** 파일을 찾아 **열기**를 클릭합니다.

1. **매개 변수 편집** 블레이드에서 **저장**을 클릭합니다.

1. **사용자 지정 배포** 블레이드에서 다음 설정이 구성되었는지 확인합니다 (다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용할 Azure 구독의 이름|
   |리소스 그룹| **새로 만들기**를 클릭하고 이름에 **AZ500LAB10**를 입력합니다.|
   |위치| **(미국) 미국 동부** |
   |VM 크기| **Standard_D2s_v3** |
   |VM 이름| **az500-10-vm1** |
   |관리자 사용자 이름| **학생** |
   |관리자 암호| **Pa55w.rd1234** |
   |가상 네트워크 이름| **az500-10-vnet1** |

    >**참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/ko-kr/regions/offers/)을 참고하세요.

1. **검토 + 만들기** 그리고 **만들기**를 차례로 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 

1. Azure Portal에서 Azure Portal 페이지 상단의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **가상 머신**을 입력하고 **Enter** 키를 누릅니다.

1. **가상 머신** 블레이드에서 **az500-10-vm1** 항목을 클릭합니다. 

1. **az500-10-vm1** 블레이드에서 **연결**을 클릭하고 드롭 다운 메뉴에서 **RDP**를 클릭합니다. 

1. **RDP 파일 다운로드**를 클릭하고 원격 데스크톱을 통해 **az500-10-vm1** Azure VM에 연결하는 데 사용합니다. 인증하라는 메시지가 표시되면 다음 자격 증명을 입력합니다.

   |설정|값|
   |---|---|
   |사용자 이름| **학생** |
   |암호| **Pa55w.rd1234** |

    >**참고**: 원격 데스크톱 세션과 **서버 관리자**가 로드될 때까지 기다립니다.  

    >**참고**: 이 랩의 나머지 단계는 원격 데스크톱 세션 내에서 **az500-10-vm1** Azure VM에 대해 수행됩니다. 

1. **az500-10-vm1** Azure VM에 대한 원격 데스크톱 세션 내의 **서버 관리자**에서 **로컬 서버**를 클릭한 다음 **IE 강화된 보안 구성**을 클릭합니다.

1. **Internet Explorer 강화된 보안 구성** 대화 상자에서 두 옵션을 **해제**로 설정하고 **확인**을 클릭합니다.

1. **Internet Explorer**를 시작하고 [SQL Server Management Studio 다운로드](https://docs.microsoft.com/ko-kr/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) 페이지를 검색합니다.

1. SQL Server Management Studio 설치 프로그램을 다운로드하고 시작합니다.  

    >**참고**: SQL Server Management Studio 설치가 완료되기까지 기다리지 말고 다음 단계로 진행합니다. 

1. **Internet Explorer**를 시작하고 [Visual Studio 다운로드](https://visualstudio.microsoft.com/downloads/) 페이지를 찾아봅니다.

1. Visual Studio 2019 Community Edition의 다운로드 및 설치를 시작합니다. 메시지가 표시되면 **Visual Studio 설치 프로그램** 창에서 **계속**을 클릭합니다.

1. 메시지가 표시되면 **워크로드** 창의 **데스크톱 및 모바일** 섹션에서 **NET 데스크톱 개발** 확인란을 선택하고 **설치를 클릭합니다*.

    >**참고**: Visual Studio 2019의 설치가 완료될 때까지 기다리지 말고 다음 작업으로 진행합니다. 

> 결과: 이 랩의 두 번째 연습에서 사용할 Azure VM **az500-vm1**의 템플릿 배포를 시작했습니다.

#### 작업 2: Key Vault 만들기 및 구성

이 작업에서는 랩 리소스 그룹과 키 자격 증명 모음을 만듭니다. 또한 키 자격 증명 모음 사용 권한을 구성합니다.

1. **az500-10-vm1** Azure VM에 대한 원격 데스크톱 세션 내에서 Azure Portal(**`https://portal.azure.com/`**)에 로그인합니다.

    >**참고**: 이 랩에 사용 중인 Azure 구독에 Owner 또는 Contributor 역할이 있는 계정을 사용하여 Azure Portal에 로그인합니다.

1. Azure Portal 오른쪽 상단에 있는 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 메시지가 표시되면 **PowerShell**을 선택하고 **스토리지를 만듭니다**.

1. Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **PowerShell**이 선택되었는지 확인합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 리소스 그룹 **AZ500LAB10**에 키 자격 증명 모음을 만듭니다. 키 자격 증명 모음 이름은 고유해야 합니다. 선택한 이름을 기억하세요. 이 랩에서 필요합니다.  

1. 작업 1을 건너뛰었다면 다음 명령을 실행하여 리소스 그룹을 만들고, 그렇지 않을 경우 다음 작업으로 이동합니다.

    ```powershell
    New-AzResourceGroup -Name 'AZ500LAB10' -Location eastus
    ```

1. Key Vault 만들기

    ```powershell
    $kvName = 'az500kv' + $(Get-Random)
    New-AzKeyVault -VaultName $kvName -ResourceGroupName 'AZ500LAB10' -Location 'eastus'
    ```

    >**참고**: 이 출력에는 자격 증명 이름과 자격 증명 URI가 표시됩니다. 자격 증명 URI는 `https://<vault_name>.vault.azure.net/` 형식입니다.

1. Cloud Shell 창을 닫습니다. 

1. Azure Portal에서 Azure Portal 페이지 상단의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **리소스 그룹**을 입력하고 **Enter**키를 누릅니다.

1. **리소스 그룹** 블레이드의 리소스 그룹 목록에서 **AZ500LAB10** 항목을 클릭합니다.

1. **AZ500LAB10-RG** 블레이드에서 새로 만든 키 자격 증명 모음을 나타내는 항목을 클릭합니다. 

1. 키 자격 증명 모음 블레이드의 **설정** 섹션에서 **액세스 정책**을 클릭한 다음 **액세스 정책 추가**를 클릭합니다.

1. **액세스 정책 추가** 블레이드에서 다음 설정을 지정합니다(다른 모든 설정을 기본값으로 남겨둠). 

    |설정|값|
    |----|----|
    |템플릿에서 구성(선택 사항)| **키, 비밀 및 인증서 관리**|
    |주요 권한| **모두 선택**을 클릭하면 권한이 **16개 선택**됩니다.|
    |비밀 권한| **모두 선택**을 클릭하면 총 **8개의 선택된** 권한이 나타납니다.|
    |인증 권한| **모두 선택**을 클릭하면 총 **16개의 선택된** 권한이 나타납니다.|
    |보안 주체 선택| **선택된 항목 없음**을 클릭하고, **보안 주체** 블레이드에서 사용자 계정을 선택하고 **선택**을 클릭합니다.|

1. **액세스 정책 추가** 블레이드로 돌아가서, **추가**를 클릭하여 액세스 정책을 추가하고, 키 자격 증명 모음의 액세스 정책 블레이드로 다시 이동하여 변경 내용을 저장하기 위해 **저장**을 클릭합니다. 

#### 작업 3: Key Vault에 키 추가

이 작업에서는 키 자격 증명 모음에 키를 추가하고 키에 대한 정보를 확인합니다. 

1. Azure Portal의 Cloud Shell 창에서 PowerShell 세션을 엽니다.

1. Cloud Shell 창의 왼쪽 위 드롭다운 메뉴에서 **PowerShell**이 선택되어 있는지 확인합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 Key Vault에 소프트웨어로 보호된 키를 추가합니다. 

    ```powershell
    $kv = Get-AzKeyVault -ResourceGroupName 'AZ500LAB10'

    $key = Add-AZKeyVaultKey -VaultName $kv.VaultName -Name 'MyLabKey' -Destination 'Software'
    ```

    >**참고**: 키 이름은 MyLabKey입니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 키가 만들어졌는지 확인합니다.

    ```powershell
    Get-AZKeyVaultKey -VaultName $kv.VaultName
    ```

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 키 식별자를 표시합니다.

    ```powershell
    $key.key.kid
    ```

1. Cloud Shell 창을 최소화합니다. 

1. Azure Portal로 돌아가서 키 자격 증명 모음 블레이드의 **설정** 섹션에서 **키**를 클릭합니다.

1. 키 목록에서 **MyLabKey** 항목을 클릭한 다음 **MyLabKey** 블레이드에서 키의 현재 버전을 나타내는 항목을 클릭합니다.   

    >**참고**: 생성한 키 관련 정보를 점검합니다.

    >**참고**: 키 식별자를 사용하여 모든 키를 참조할 수 있습니다. 최신 버전을 얻으려면 `https://<key_vault_name>.vault.azure.net/keys/MyLabKey`을 참조하거나 다음과 같은 특정 버전을 가져옵니다. `https://<key_vault_name>.vault.azure.net/keys/MyLabKey/<key_version>`


#### 작업 4: Key Vault에 비밀 추가

1. Cloud Shell 창을 복원합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 보안 문자열 값이 있는 변수를 만듭니다.

    ```powershell
    $secretvalue = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    ```

1.  Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 자격 증명 모음에 암호를 추가합니다.

    ```powershell
    $secret = Set-AZKeyVaultSecret -VaultName $kv.VaultName -Name 'SQLPassword' -SecretValue $secretvalue
    ```

    >**참고**: 암호의 이름은 SQLPassword입니다. 

1.  Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 비밀이 만들어졌는지 확인합니다.

    ```powershell
    Get-AZKeyVaultSecret -VaultName $kv.VaultName
    ```

1. Cloud Shell 창을 최소화합니다. 

1. Azure Portal에서 키 자격 증명 모음 블레이드로 다시 이동하여, **설정** 섹션에서 **암호**를 클릭합니다.

1. 비밀 목록에서 **SQLPassword** 항목을 클릭한 다음 **SQLPassword** 블레이드에서 현재 버전의 비밀을 나타내는 항목을 클릭합니다.

    >**참고**: 생성한 비밀 관련 정보를 점검합니다.

    >**참고**: 비밀의 최신 버전을 얻으려면,`https://<key_vault_name>.vault.azure.net/secrets/<secret_name>`를 참조하거나 특정 버전을 얻거나 `https://<key_vault_name>.vault.azure.net/secrets/<secret_name>/<secret_version>`를 참조합니다.


### 연습 2: 암호화를 위한 키 자격 증명 모음을 사용하여 시연할 애플리케이션 만들기

### 예상 시간: 60분

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: 클라이언트 애플리케이션이 Azure SQL Database 서비스에 액세스할 수 있도록 합니다. 
- 작업 2: Key Vault에 애플리케이션이 액세스할 수 있는 정책 만들기
- 작업 3: Azure SQL Database 만들기.
- 작업 4: SQL Database에 테이블을 만들고 암호화를 위한 데이터 열을 선택합니다.
- 작업 5: 암호화된 열을 사용하는 콘솔 애플리케이션 빌드

#### 작업 1: 클라이언트 애플리케이션이 Azure SQL Database 서비스에 액세스할 수 있도록 합니다. 

이 작업에서는 클라이언트 애플리케이션이 Azure SQL Database 서비스에 액세스할 수 있도록 설정합니다. 이렇게 하면 필수 인증을 설정하고 애플리케이션을 인증하는 데 필요한 애플리케이션 ID 및 암호를 획득하여 수행됩니다.

1. Azure Portal에서 Azure Portal 페이지 상단의 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **앱 등록**을 입력하고 **Enter**키를 누릅니다.

1. **앱 등록** 블레이드에서 **새 등록**을 클릭합니다. 

1. **애플리케이션 등록** 블레이드에서 다음 설정을 지정합니다(다른 모든 설정을 기본값으로 남겨둡니다).

    |설정|값|
    |----|----|
    |이름| **sqlApp** |
    |리디렉션 URI(선택 사항)| **웹** 및 **https://sqlapp** |

1. **애플리케이션 등록** 블레이드에서 **등록**을 클릭합니다.    

    >**참고**: 등록이 완료되면 브라우저가 자동으로 **sqlApp**블레이드로 리디렉션합니다. 

1. **sqlApp** 블레이드에서 **애플리케이션(클라이언트) ID**의 값을 식별합니다. 

    >**참고**: 값을 기록합니다. 다음 작업에 필요합니다.

1.  **sqlApp** 블레이드의 **관리** 섹션에서 **인증서 및 비밀**을 클릭합니다.

1.  **SQLApp**에서** | 인증서 및 암호** 블레이드에서 **+ 새 클라이언트 암호**를 클릭합니다.

1. **클라이언트 암호 추가** 창에서 다음 설정을 지정합니다.

    |설정|값|
    |----|----|
    |설명| **Key1** |
    |만료| **1년 내에** |
	
1. 애플리케이션 자격 증명을 업데이트하려면 **추가**를 클릭합니다.

1.  **SQLApp**에서 **\| 인증서 및 비밀** 블레이드에서 **Key1**의 값을 식별합니다.

    >**참고**: 값을 기록합니다. 다음 작업에 필요합니다. 

    >**참고**: 블레이드에서 멀리 *이동*하기 전에 값을 복사해야 합니다. 이 시점에서 더 이상 명확한 텍스트 값을 검색할 수 없습니다.


#### 작업 2: Key Vault에 애플리케이션이 액세스할 수 있는 정책 만들기

이 작업에서는 새로 등록된 앱에 키 자격 증명 모음에 저장된 비밀에 액세스할 수 있는 권한을 부여합니다.

1. Azure Portal의 Cloud Shell 창에서 PowerShell 세션을 엽니다.

1. Cloud Shell 창의 왼쪽 위 드롭다운 메뉴에서 **PowerShell**이 선택되어 있는지 확인합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 이전 작업에 기록된 **애플리케이션(클라이언트) ID**를 저장하는 변수를 만듭니다(`<Azure_AD_Application_ID>` 자리 표시자를 **애플리케이션(클라이언트) ID**의 값으로 대체).
   
    ```powershell
    $applicationId = '<Azure_AD_Application_ID>'
    ```
1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 키 자격 증명 모음 이름을 저장하는 변수를 만듭니다.
	```
    $kvName = (Get-AzKeyVault -ResourceGroupName 'AZ500LAB10').VaultName
    ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이전 작업에 등록한 애플리케이션에 키 자격 증명 모음에 대한 권한을 부여합니다.

    ```powershell
    Set-AZKeyVaultAccessPolicy -VaultName $kvName -ResourceGroupName AZ500LAB10 -ServicePrincipalName $applicationId -PermissionsToKeys get,wrapKey,unwrapKey,sign,verify,list
    ```

1. Cloud Shell 창을 닫습니다. 

#### 작업 3: Azure SQL Database 만들기

이 작업에서는 Azure SQL 데이터베이스를 만들고 해당 ADO.NET 연결 문자열을 식별합니다. 

1. Azure Portal의 페이지 상단에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **SQL 데이터베이스**를 입력하고 **Enter** 키를 누릅니다.

1. **SQL 데이터베이스** 블레이드에서 **추가**를 클릭합니다.

1. **SQL 데이터베이스 만들기** 블레이드의 **기본** 탭에서 설정을 지정합니다(다른 설정은 기본값으로 남겨둠).

    |설정|값|
    |---|--- |
    |구독|이 랩에서 사용할 Azure 구독의 이름|
    |리소스 그룹| **AZ500LAB10** |
    |데이터베이스 이름| **medical** |

1. **서버** 드롭다운 리스트 바로 아래에서 **새로 만들기**를 클릭하고 **새 서버** 블레이드에서 다음 설정을 지정한 후 **확인**을 클릭합니다(다른 설정은 기본값으로 남겨둠).

    |설정|값|
    |---|---|
    |서버 이름|유효하고 전역적으로 고유한 이름|
    |서버 관리자 로그인| **학생** |
    |암호| **Pa55w.rd1234** |
    |위치 |**(미국) 미국 동부** |

1. **검토 + 만들기**와 **만들기**를 차례로 클릭합니다. 

    >**참고**: SQL Database가 만들어질 때까지 기다립니다. 프로비전은 약 2분이 소요됩니다. 

1. Azure Portal에서 **SQL 데이터베이스** 블레이드로 다시 이동한 다음, SQL 데이터베이스 목록에서 **의료** 항목을 클릭합니다.

1. SQL Database 블레이드의 **설정** 섹션에서 **연결 문자열**을 클릭합니다. 

    >**참고**: 인터페이스에는 ADO.NET, JDBC, ODBC, PHP 및 Go용 연결 문자열이 포함됩니다. 
   
1. **ADO.NET 연결 문자열**을 기록합니다.

    >**참고**: 연결 문자열을 사용하는 경우 `{your_password}` 자리 표시자를 **Pa55w.rd1234**로 교체해야 합니다.


#### 작업 4: SQL Database에서 테이블을 만들고 암호화를 위한 데이터 열을 선택합니다.

>**참고**: 이 작업을 진행하기 전에 이 랩의 시작 부분에서 시작한 SQL Server Management Studio의 설치가 완료되었는지 확인합니다.

이 작업에서는 SQL Server Management Studio를 사용하여 SQL Database에 연결하고 테이블을 생성합니다. 그런 다음 Azure Key Vault에서 자동 생성된 키를 사용하여 두 개의 데이터 열을 암호화합니다. 

1. Azure Portal 내 **의료** SQL 데이터베이스의 블레이드에서 **개요**를 클릭하고 **서버 이름**을 식별한 다음 **서버 방화벽 설정**을 클릭합니다.  

    >**참고**: 서버 이름을 기록합니다. 이 작업의 후반에서 필요할 것입니다.

1. **방화벽 설정** 블레이드에서 + **클라이언트 IP + 추가**를 클릭하고 **저장**을 클릭한 다음 **확인**을 클릭합니다.    

    >**참고**: 이렇게 하면 서버 방화벽 설정을 수정하여 현재 사용 중인 컴퓨터와 연결된 공용 IP 주소에서 의료 데이터베이스에 연결할 수 있습니다. 

1. **시작**을 클릭하고, **시작** 메뉴에서**Microsoft SQL Server Tools 18** 폴더를 확장하고  **Microsoft SQL Server Management Studio** 메뉴 항목을 클릭합니다.

1. **서버에 연결** 대화 상자에서, 다음 설정을 지정합니다. 

    |설정|값|
    |---|---|
    |서버 유형| **데이터베이스 엔진**|
    |서버 이름|이 작업의 앞부분에서 식별한 서버 이름|
    |인증: **SQL Server 인증**
    |로그인| **학생** |
    |암호| **Pa55w.rd1234** |

1. **서버에 연결** 대화 상자에서 **연결**을 클릭합니다.

1. **SQL Server Management Studio** 콘솔 내, **개체 탐색기** 창에서 **데이터베이스** 폴더를 확장합니다.

1. **개체 탐색기** 창에서 **의료** 데이터베이스를 마우스 오른쪽 단추로  클릭하고 **새 쿼리**를 클릭합니다.

1. 다음 코드를 쿼리 창에 붙여넣고 **실행**을 클릭합니다. 그러면 **환자** 테이블이 생성됩니다.

     ```sql
     CREATE TABLE [dbo].[Patients](
		[PatientId] [int] IDENTITY(1,1),
		[SSN] [char](11) NOT NULL,
		[FirstName] [nvarchar](50) NULL,
		[LastName] [nvarchar](50) NULL,
		[MiddleName] [nvarchar](50) NULL,
		[StreetAddress] [nvarchar](50) NULL,
		[City] [nvarchar](50) NULL,
		[ZipCode] [char](5) NULL,
		[State] [char](2) NULL,
		[BirthDate] [date] NOT NULL 
     PRIMARY KEY CLUSTERED ([PatientId] ASC) ON [PRIMARY] );
     ```
1. 테이블을 성공적으로 만든 후 **개체 탐색기**에서 **의료** 데이터베이스 노드, **테이블** 노드를 확장하여 **dbo.Patients** 노드를 마우스 오른쪽 단추로 클릭한 후 **열 암호화**를 클릭합니다. 

    >**참고**: 그러면 **항상 암호화** 마법사 표시가 시작됩니다. 

1. **소개** 페이지에서 **다음**을 클릭합니다.

1. **열 선택** 페이지에서 **SSN** 및 **생년월일** 열을 선택하고 **SSN** 열의 **암호화 유형**을 **명확함**으로 **생년월일** 열을 **무작위**로 설정한 후 **다음**을 클릭합니다.

1. **마스터 키 구성** 페이지에서 **Azure Key Vault**를 선택하고 **로그인**을 클릭한 후 메시지가 표시되면 이 랩의 앞부분에서 Azure Key Vault 인스턴스를 프로비전하는 데 사용한 동일한 사용자 계정을 사용하여 인증하고, 키 자격 증명 모음이 **Azure Key Vault 선택** 드롭다운 목록에 표시되는지 확인한 후 **다음**을 클릭합니다.

1. **설정 실행** 페이지에서 **다음**을 클릭합니다.
	
1. **요약** 페이지에서 **완료**를 클릭하여 암호화를 진행합니다. 메시지가 표시되면 이 랩의 앞부분에서 Azure Key Vault 인스턴스 프로비전에 사용한 것과 동일한 사용자 계정을 사용하여 다시 로그인합니다.

1. 암호화 프로세스가 완료되면 **결과** 페이지에서 **닫기**를 클릭합니다.

1. **SQL Server Management Studio** 콘솔의 **의료** 노드 아래에 있는 **개체 탐색기** 창에서 **보안** 및 **항상 암호화 키** 하위 노드를 확장합니다. 

    >**참고**: **항상 암호화 키** 하위 노드에는 **열 마스터 키** 및 **열 암호화 키** 하위 폴더가 포함되어 있습니다.


#### 작업 5: 암호화된 열 사용을 위한 콘솔 애플리케이션 빌드

>**참고**: 이 작업을 진행하기 전에 이 랩의 앞부분에서 시작한 Visual Studio 2019의 설치가 완료되었는지 확인합니다.

그런 다음 Visual Studio를 통해 콘솔 애플리케이션을 만들어서 암호화된 열에 데이터를 로드하고, Key Vault의 키에 액세스하는 연결 문자열을 사용하여 해당 데이터에 안전하게 액세스합니다.

1. Visual Studio 2019 시작 메시지를 표시하는 창으로 전환하고, **나중에** 링크를 클릭한 다음 **Visual Studio 시작**을 클릭합니다.

1. **시작** 페이지에서 **새 프로젝트 만들기**를 클릭합니다. 

1. 프로젝트 템플릿 목록에서 **콘솔 앱(.NET 프레임워크)**을 검색하고, 결과 목록에서 **C#**에 대해 **콘솔 앱(.NET 프레임워크)**을 클릭한 후 **다음**을 클릭합니다.

1. **새 프로젝트 구성** 페이지에서 다음 설정을 지정합니다(다른 설정은 기본값으로 남겨둠).

    |설정|값|
    |---|---|
    |프로젝트 이름| **OpsEncrypt** |
    |솔루션 이름| **OpsEncrypt** |
    |프레임워크| **NET 프레임워크 4.7.2.** |

1. Visual Studio 콘솔에서 **도구** 메뉴를 클릭하고, 드롭다운 메뉴에서 **NuGet 패키지 관리자**를 클릭한 후 계단식 메뉴에서 **패키지 관리자 콘솔**을 클릭합니다.

1. **패키지 관리자 콘솔** 창에서 다음을 실행하여 필요한 **NuGet** 패키지를 설치합니다.

    ```powershell
    Install-Package Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

1. **\\Allfiles\\10\\program.cs**로 이동하여 메모장에서 열고 해당 콘텐츠를 클립보드에 복사합니다.

1. Visual Studio 콘솔로 전환하여 **솔루션 탐색기** 창에서 **Program.cs**를 클릭하고 해당 콘텐츠를 클립보드에 복사한 코드로 바꿉니다.

1. Visual Studio 창의 **Program.cs** 창 15줄에서 `<connection string noted earlier>` 자리 표시자를 랩의 앞부분에서 기록한 Azure SQL Database **ADO.NET** 연결 문자열로 바꿉니다.

1. Visual Studio 창의 **Program.cs** 창 16줄에서 `<client id noted earlier>` 자리 표시자를 랩의 앞부분에서 기록한 등록된 앱의 **애플리케이션(클라이언트) ID** 값으로 바꿉니다. 

1. Visual Studio 창의 **Program.cs** 창 17줄에서 `<key value noted earlier>` 자리 표시자를 랩의 앞부분에서 기록한 등록된 앱의 **Key1** 값으로 바꿉니다. 

1. Visual Studio 콘솔에서 **시작** 단추를 클릭하여 콘솔 애플리케이션의 빌드를 시작합니다.

1. 애플리케이션은 명령 프롬프트 창을 시작합니다. 암호에 대한 메시지가 표시되면 Azure SQL 데이터베이스에 연결하기 위해 **Pa55w.rd1234를**입력합니다. 

1. 콘솔 앱은 실행 중인 상태로 유지하고 **SQL Management Studio** 콘솔로 전환합니다. 

1. **개체 탐색기** 창에서 의료 데이터베이스를 마우스 오른쪽 단추로 클릭하고 마우스 오른쪽 클릭 메뉴에서 **새 쿼리**를 클릭합니다.

1. 쿼리 창에서 다음 쿼리를 실행하여 콘솔 앱의 데이터베이스에 로드된 데이터가 암호화되었는지 확인합니다.

    ```sql
    SELECT FirstName, LastName, SSN, BirthDate FROM Patients;
    ```

1. 이제 콘솔 애플리케이션으로 다시 전환하면 유효한 SSN을 입력하라는 메시지가 표시됩니다. 이 SSN을 입력하면 암호화된 열에서 데이터를 쿼리합니다. 명령 프롬프트에서 다음을 입력하고 Enter 키를 누릅니다.

    ```cmd
    999-99-0003
    ```

    >**참고**: 쿼리에서 반환된 데이터가 암호화되지 않았는지 확인합니다.

1. 콘솔 앱을 종료하려면 Enter 키를 누릅니다.

**리소스 정리**

> 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

1. Azure Portal에서 Azure Portal의 오른쪽 상단의 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 

1. Cloud Shell 창의 왼쪽 위 드롭다운 메뉴에서 **PowerShell**을 선택하고 메시지가 표시되면 **확인**을 클릭합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 제거합니다.
  
    ```powershell
    Remove-AzResourceGroup -Name "AZ500LAB10" -Force -AsJob
    ```

1.  **Cloud Shell** 창을 닫습니다. 
