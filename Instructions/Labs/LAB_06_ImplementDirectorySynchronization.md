---
lab:
    title: '06 - 디렉터리 동기화 구현'
    module: '모듈 01 - ID 및 액세스 관리'
---

# 랩 06: 디렉터리 동기화 구현
# 학생 랩 매뉴얼

## 랩 시나리오

Azure Active Directory(Azure AD) 테넌트와 온-프레미스 AD DS(Active Directory Domain Services) 환경을 통합하는 방법을 보여주는 개념 증명을 만들어야 합니다. 특히 다음을 수행해야 합니다.

- AD DS 도메인 컨트롤러를 호스팅하는 Azure VM을 배포하여 단일 도메인 AD DS 포리스트 구현
- Azure AD 테넌트 만들기 및 구성.
- Azure AD 테넌트와 AD DS 포리스트 동기화

> 이 랩에 있는 모든 리소스의 경우 **미국 동부** 지역을 사용합니다. 강사에게 이 지역이 수업에 사용할 지역인지 확인합니다. 

## 랩 목표

이 랩에서는 다음과 같은 연습을 완료합니다.

- 연습 1: Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM 배포
- 연습 2: Azure Active Directory 테넌트 만들기 및 구성
- 연습 3: Azure Active Directory 테넌트를 사용하여 Active Directory 포리스트를 동기화합니다

### 연습 1: Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM 배포

### 예상 시간: 10분

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: Azure VM 배포에 사용할 수 있는 DNS 이름 식별
- 작업 2: ARM 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM 배포

#### 작업 1: Azure VM 배포에 사용할 수 있는 DNS 이름 식별

이 작업에서는 Azure VM 배포용 DNS 이름을 식별합니다. 

1. Azure Portal **`https://portal.azure.com/`**에 로그인합니다.

    >**참고**: 이 랩에 사용 중인 Azure 구독에 Owner 또는 Contributor 역할이 있는 계정을 사용하여 Azure Portal에 로그인합니다.

1. Azure Portal 오른쪽 상단에 있는 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 메시지가 표시되면 **PowerShell**, 그리고 **스토리지 만들기**를 클릭합니다.

1. Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **PowerShell**이 선택되었는지 확인합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 이 연습의 다음 작업에서 Azure VM 배포에 사용할 수 있는 사용 가능한 DNS 이름을 식별합니다.

    ```powershell
    Test-AzDnsAvailability -DomainNameLabel <custom-label> -Location '<location>'
    ```

    >**참고**: '<custom-label>' 자리 표시자를 전역에서 고유할 가능성이 높은 유효한 DNS 이름으로 바꿉니다. '<location>' 자리 표시자를 이 랩에서 사용할 Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM을 배포하려는 영역의 이름으로 바꿉니다.

    >**참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/en-us/regions/offers/)를 참조하세요.

1. 이 명령이 **True**를 반환하는지 확인합니다. True가 반환되지 않으면 **True**가 반환될 때까지 '<custom-label>'의 다른 값을 사용해 같은 명령을 다시 실행합니다.

1. 성공적인 결과가 반환된 '<custom-label>'의 값을 레코드합니다. 다음 작업에 필요합니다.

1. Cloud Shell을 닫습니다.

#### 작업 2: ARM 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM 배포

이 작업에서는 Active Directory 도메인 컨트롤러를 호스팅하는 Azure VM을 배포합니다.

1. 동일한 브라우저 창에서 다른 브라우저 탭을 열고 [https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain](https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain)로 이동합니다. 

1. **새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기** 페이지에서 **Azure에 배포**를 클릭합니다. 이렇게 하면 Azure Portal의 **새 AD 포리스트로 Azure VM 만들기** 블레이드로 브라우저가 자동으로 리디렉션됩니다.

1. **새 AD 포리스트가 있는 Azure VM 만들기 **블레이드에서 **매개 변수 편집**을 클릭합니다.

1. **매개 변수 편집** 블레이드에서 **파일 로드**를 클릭하고 **열기** 대화 상자에서 **\\\\AllFiles\Labs\\06\\active-directory-new-domain\\azuredeploy.parameters.json**, **열기**, **저장**을 차례로 클릭합니다. 

1. **새 AD 포리스트를 사용하여 Azure VM 만들기** 블레이드에서 다음 설정을 지정합니다(기존 값은 유지).

   |설정|값|
   |---|---|
   |구독|Azure 구독명|
   |리소스 그룹|**새로 만들기**를 선택하고 이름 **AZ500LAB06**를 입력합니다.|
   |지역|이전 작업에서 확인한 Azure 지역|
   |관리자 사용자 이름|**Student**|
   |관리자 암호|**Pa55w.rd1234**|
   |도메인 이름|**adatum.com**|
   |DNS 접두사|이전 작업에서 만든 DNS 호스트 이름|
   |VM 크기|**Standard_D2s_v3**|

1. **새 AD 포리스트가 있는 Azure VM 만들기** 블레이드에서 **검토 + 만들기**, **만들기**를 차례로 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 배포에는 약 15분이 소요됩니다. 이 랩의 세 번째 연습에서이 작업에 배포된 가상 머신을 사용합니다.

> 결과: 이 연습을 통해, Azure Resource Manager 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스트하는 Azure VM의 배포를 시작했습니다.


### 연습 2: Azure Active Directory 테넌트 만들기 및 구성 

### 예상 시간: 20분

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: Azure Active Directory(AD) 테넌트 만들기
- 작업 2: 새 Azure AD 테넌트에 사용자 지정 DNS 이름 추가
- 작업 3: 전역 관리자 역할을 사용하여 Azure AD 사용자 만들기

#### 작업 1: Azure Active Directory(AD) 테넌트 만들기

이 작업에서는 이 랩에서 사용할 새 Azure AD 테넌트를 만듭니다. 

1. Azure Portal 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **Azure Active Directory**를 입력하고 **Enter** 키를 누릅니다.

1. 현재 Azure AD 테넌트의 **개요**를 표시하는 블레이드에서 **+ 테넌트 만들기**를 클릭합니다.

1. **디렉터리 만들기** 블레이드의 **기본** 탭에서 **Azure Active Directory** 옵션을 선택하고 **다음: 구성 >**을 클릭합니다.

1. **디렉터리 만들기** 블레이드의 **구성** 탭에서 다음 설정을 구성합니다.

   |설정|값|
   |---|---|
   |조직 이름:|**AdatumSync**|
   |초기 도메인 이름|문자와 숫자의 조합으로 구성된 고유한 이름|
   |국가 또는 지역|**미국**|

    >**참고**: 초기 도메인 이름을 기록합니다. 이 랩의 후반부에서 필요합니다.

    >**참고**: **초기 도메인 이름** 텍스트 상자의 녹색 확인 표시는 입력한 도메인 이름이 유효하고 고유한지 여부를 나타냅니다. (나중에 사용하기 위해 초기 도메인 이름 기록).

1. **검토 + 만들기**를 클릭한 다음, **만들기**를 클릭합니다.

    >**참고**: 새 테넌트가 만들어질 때까지 기다립니다. **알림** 아이콘을 사용하여 배포 상태를 모니터링합니다. 

#### 작업 2: 새 Azure AD 테넌트에 사용자 지정 DNS 이름 추가

이 작업에서는 새 Azure AD 테넌트에 사용자 지정 DNS 이름을 추가합니다. 

1. Azure Portal의 도구 모음에서 Cloud Shell 셸 아이콘 오른쪽에 있는 **디렉터리 + 구독** 아이콘을 클릭합니다. 

1. **디렉터리 + 구독** 블레이드에서 새로 만든 테넌트인 **AdatumSync**를 클릭합니다.

    >**참고**: **AdatumSync** 항목이 **디렉터리 + 구독** 필터 목록에 나타나지 않으면 브라우저 창을 새로 고침합니다.

1. **AdatumSync** 블레이드의 **관리** 섹션에서 **사용자 지정 도메인 이름**을 클릭합니다.

1. **AdatumSync \| 사용자 지정 도메인 이름** 블레이드에서 **+ 사용자 지정 도메인 추가**를 클릭합니다.

1. **사용자 지정 도메인 이름** 블레이드에서 **사용자 지정 도메인 이름** 텍스트 상자에서 **adatum.com**을 입력하고 **도메인 추가**를 클릭합니다.

1. **adatum.com** 블레이드에서 Azure AD 도메인 이름 확인을 수행하는 데 필요한 정보를 검토합니다.

    >**참고**: **adatum.com** DNS 도메인 이름이 없으므로 유효성 검사 프로세스를 완료할 수 없습니다. 그렇다고 해서 **adatum.com** AD DS 도메인을 Azure AD 테넌트와 동기화할 수 없는 것은 아닙니다. 이를 위해 이전 작업에서 식별한 Azure AD 테넌트의 초기 DNS 이름(**onmicrosoft.com** 접미사로 끝나는 이름)을 사용합니다. 그러나 결과적으로 AD DS 도메인의 DNS 도메인 이름과 Azure AD 테넌트의 DNS 이름은 다르다는 점을 기억해 두세요. 즉, Adatum 사용자는 AD DS 도메인에 로그인할 때와 Azure AD 테넌트에 로그인할 때 각기 다른 이름을 사용해야 합니다.

#### 작업 3: 전역 관리자 역할을 사용하여 Azure AD 사용자 만들기

이 작업에서는 새 Azure AD 사용자를 추가하고 전역 관리자 역할에 할당합니다. 

1. **AdatumSync** Azure AD 테넌트 블레이드의 **관리** 섹션에서 **사용자**를 클릭합니다.

1. **사용자 \| 모든 사용자** 블레이드에서 **+ 새 사용자**를 클릭합니다. 

1. **새 사용자** 블레이드에서 **사용자 만들기** 옵션이 선택되어 있는지 확인하고 다음 설정을 지정합니다. 다른 설정은 모두 기본값으로 유지하고 **만들기**를 클릭합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**syncadmin**|
   |이름|**syncadmin**|
   |암호|**암호 자동 생성** 옵션이 선택되었는지 확인하고 **암호 표시**를 클릭합니다|
   |그룹|**0개 그룹 선택됨**|
   |역할|**사용자**를 클릭하고, **전역 관리자**를 클릭한 다음 **Select**를 클릭합니다.|
   |사용 위치|**미국**|  

    >**참고**: 전체 사용자 이름을 기록합니다. 도메인 이름이 표시된 드롭다운 목록의 오른쪽에 있는 **클립보드에 복사** 단추를 클릭하여 해당 값을 복사할 수 있습니다. 

    >**참고**: 사용자의 암호를 기록합니다. 이 랩에서 나중에 필요합니다. 

    >**참고**: Azure AD Connect를 구현하려면 전역 관리자 역할을 가진 Azure AD 사용자가 필요합니다.

1. InPrivate 브라우저 창을 엽니다.

1. Azure Portal로 이동하고 **syncadmin** 사용자 계정을 사용하여 로그인합니다. 메시지가 표시되면 이 작업에서 이전에 기록한 암호를 **Pa55w.rd1234**로 변경합니다.

    >**참고**: 로그인하려면 이 작업의 앞부분에서 기록했던 AzureAD 테넌트 DNS 도메인 이름을 포함하여 **syncadmin** 사용자 계정의 정규화된 이름을 제공해야 합니다. 이 사용자 이름은 syncadmin@`<your_tenant_name>`.onmicrosoft.com 형식이며, 여기서 '<your_tenant_name>'은 고유한 Azure AD 테넌트 이름을 나타내는 자리 표시자입니다. 

1. **syncadmin**으로 로그아웃하고 InPrivate 브라우저 창을 닫습니다.

> **결과**: 이 연습을 통해 Azure AD 테넌트를 만들고, 새 Azure AD 테넌트에 사용자 지정 DNS 이름을 추가하고, 전역 관리자 역할을 가진 Azure AD 사용자를 만들었습니다.


### 연습 3: Azure Active Directory 테넌트를 사용하여 Active Directory 포리스트를 동기화합니다

### 예상 시간: 20분

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: 디렉터리 동기화를 위한 AD DS 준비
- 작업 2: Azure AD Connect 설치
- 작업 3: 디렉터리 동기화 확인

#### 작업 1: 디렉터리 동기화를 위한 AD DS 준비

이 작업에서는 AD DS 도메인 컨트롤러를 실행하는 Azure VM에 연결하고 디렉터리 동기화 계정을 만듭니다. 

   > 이 작업을 시작하기 전에 이 랩의 첫 번째 연습에서 시작한 템플릿 배포가 완료되었는지 확인합니다.

1. Azure Portal에서 **디렉터리 + 구독** 필터를 이 랩의 첫 번째 연습에서 Azure VM을 배포한 Azure 구독과 연결된 Azure AD 테넌트로 설정합니다.

1. Azure Portal 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **가상 머신**을 입력하고 **Enter** 키를 누릅니다.

1. **가상 머신** 블레이드에서 **adVM** 항목을 클릭합니다. 

1. **adVM** 블레이드에서 **연결**을 클릭하고 드롭다운 메뉴에서 **RDP**를 클릭합니다. 

1. **IP 주소** 매개 변수에서 **부하 분산 장치 공용 IP 주소**를 선택한 다음 **RDP 파일 다운로드**를 클릭합니다. 그런 후에 해당 파일을 사용하여 원격 데스크톱을 통해 **adVM** Azure VM에 연결합니다. 인증하라는 메시지가 표시되면 다음 자격 증명을 입력합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

    >**참고**: 원격 데스크톱 세션과 **서버 관리자**가 로드될 때까지 기다립니다.  

    >**참고**: 다음 단계는 **adVM** Azure VM에 대한 원격 데스크톱 세션에서 수행됩니다. 

1. **서버 관리자**에서 **로컬 서버**를 클릭한 다음 **IE 보안 강화 구성**을 클릭합니다.

1. **Internet Explorer 강화된 보안 구성** 대화 상자에서 두 옵션을 **해제**로 설정하고 **확인**을 클릭합니다.

1. **서버 관리자**에서 **도구**를 클릭하고 드롭다운 메뉴에서 **Active Directory 관리 센터**를 클릭합니다.

1. **Active Directory 관리 센터**에서 **adatum(로컬)**을 클릭하고 **작업** 창에서 **신규**를 클릭한 후 계단식 메뉴에서 **조직 구성 단위**를 클릭합니다.

1. **조직 구성 단위 만들기** 창의 **이름** 텍스트 상자에 **ToSync**를 입력하고 **확인**을 클릭합니다.

1. 새로 만든 **ToSync** 조직 구성 단위를 두 번 클릭하면 해당 콘텐츠가 Active Directory 관리 센터 콘솔의 세부 정보 창에 나타납니다. 

1. **작업** 창의 **ToSync** 섹션 내에서 **신규**를 클릭하고 계단식 메뉴에서 **사용자**를 클릭합니다.

1. **사용자 만들기** 창에서 다음 설정이 있는 새 사용자 계정을 만들고(다른 설정은 기존값으로 남겨둡니다) **확인**을 클릭합니다.

   |설정|값|
   |---|---|
   |전체 이름|**aduser1**|
   |사용자 UPN 로그온|**aduser1**|
   |사용자 SamAccountName 로그온|**aduser1**|
   |암호|**Pa55w.rd1234**|
   |기타 암호 옵션|**암호가 만료되지 않습니다**|

#### 작업 2: Azure AD Connect 설치

이 작업에서는 가상 머신에 AD Connect를 설치합니다. 

1. **adVM**에 대한 원격 데스크톱 세션에서 Internet Explorer를 시작하고, [Azure Portal](https://portal.azure.com)로 이동한 뒤 이전 연습에서 만든 **syncadmin** 사용자 계정을 사용하여 로그인합니다. 메시지가 표시되면 기록한 전체 사용자 이름과 **Pa55w.rd1234** 암호를 지정합니다.

1. Azure Portal 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **Azure Active Directory**를 입력하고 **Enter** 키를 누릅니다.

1. Azure Portal의 **AdatumSync \| 개요** 블레이드에서 **Azure AD Connect**를 클릭합니다.

1. **AdatumSync \| Azure AD Connect** 블레이드에서 **Azure AD Connect 다운로드** 링크를 클릭합니다. **Microsoft Azure Active Directory Connect** 다운로드 페이지로 리디렉션됩니다.

1. **Microsoft Azure Active Directory Connect** 다운로드 페이지에서 **다운로드**를 클릭합니다.

1. 메시지가 표시되면 **실행**을 클릭하여 **Microsoft Azure Active Directory Connect** 마법사를 시작합니다.

1. **Microsoft Azure Active Directory Connect** 마법사의 **Azure AD Connect 시작** 페이지에서 **사용 조건 및 개인정보취급방침에 동의함** 확인란을 클릭하고 **계속**을 클릭합니다.

1. **Microsoft Azure Active Directory Connect** 마법사의 **기본 설정** 페이지에서 **사용자 지정** 옵션을 클릭합니다.

1. **필수 구성 요소 설치** 페이지에서 모든 선택 구성 옵션의 선택을 취소하고 **설치**를 클릭합니다.

1. **사용자 로그인** 페이지에서 **암호 해시 동기화**만 사용하도록 설정되어 있는지 확인하고 **다음**을 클릭합니다.

1. **Azure AD에 연결** 페이지에서 이전 연습에서 만든 **syncadmin** 사용자 계정의 자격 증명을 사용하여 인증하고 **다음**을 클릭합니다. 

1. **디렉터리 연결** 페이지에서 **adatum.com** 포리스트 항목 오른쪽에 있는 **디렉터리 추가** 단추를 클릭합니다.

1. **AD 포리스트 계정** 창에서 **새 AD 계정 만들기** 옵션이 선택되어 있는지 확인한 후, 다음 자격 증명을 지정하고 **확인**을 클릭합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\\Student**|
   |암호|**Pa55w.rd1234**|

1. **디렉터리 연결** 페이지에서 **adatum.com** 항목이 구성된 디렉터리로 나타나는지 확인하고 **다음**을 클릭합니다.

1. **Azure AD 로그인 구성** 페이지에서 **UPN 접미사가 확인된 도메인 이름과 일치하지 않으면 사용자는 온-프레미스 자격 증명을 사용하여 Azure AD에 로그인할 수 없습니다**라고 표시된 경고를 확인하고, **모든 UPN 접미사를 확인된 도메인과 일치시키지 않고 계속** 확인란을 선택하고 **다음**을 클릭합니다.

    >**참고**: 앞서 설명했듯이 사용자 지정 Azure AD DNS 도메인 **adatum.com**을 확인할 수 없으므로 이러한 결과가 예상됩니다.

1. **도메인 및 OU 필터링** 페이지에서 **선택한 도메인 및 OU 동기화** 옵션을 클릭하고 모든 확인란 선택을 취소한 다음 **ToSync** OU 옆의 확인란만 클릭하고 **다음**을 클릭합니다.

1. **사용자 고유하게 식별** 페이지에서 기본 설정을 수락하고 **다음**을 클릭합니다.

1. **사용자 및 디바이스 필터링** 페이지에서 기본 설정을 수락하고 **다음**을 클릭합니다.

1. **선택적 기능** 페이지에서 기본 설정을 수락하고 **다음**을 클릭합니다.

1. **구성 준비 완료** 페이지에서 **구성이 완료되면 동기화 프로세스를 시작** 확인란이 선택되었는지 확인하고 **설치**를 클릭합니다.

    >**참고**: 설치에는 약 2분이 소요됩니다.

1. **구성 완료** 페이지에서 정보를 검토하고 **끝내기**를 클릭하여 **Microsoft Azure Active Directory Connect** 창을 닫습니다.


#### 작업 3: 디렉터리 동기화 확인

이 작업에서는 디렉터리 동기화가 작동하는지 확인합니다. 

1. Azure Portal을 표시하는 Internet Explorer 창의 **adVM**에 대한 원격 데스크톱 세션에서 Adatum Lab Azure AD 테넌트의 **사용자 - 모든 사용자** 블레이드로 이동합니다.

1. **사용자 \| 모든 사용자** 블레이드에서 사용자 개체 목록에 **aduser1** 계정이 포함되어 있음을 확인합니다. 

1. **aduser1** 계정을 선택하고 **프로필 > ID** 섹션에서 **원본** 특성이 **Windows Server AD**로 설정되어 있음을 확인합니다.

    >**참고**: **aduser1** 사용자 계정이 표시되도록 하려면 잠시 기다렸다가 **새로 고침**을 선택합니다.

1. **사용자 \| 모든 사용자** 블레이드에서 **aduser1** 항목을 선택합니다.

1. **aduser1 \| 프로필** 블레이드의 **작업 정보** 섹션에서 **부서** 특성이 설정되지 않음에 유의합니다.

1. **adVM**에 대한 원격 데스크톱 세션 내에서 **Active Directory 관리 센터**로 전환하고 **ToSync** OU의 개체 목록에서 **aduser1** 항목을 선택하고 **ToSync** 섹션의 **작업** 창에서 **속성**을 선택합니다.

1. **aduser1** 창에 있는 **조직** 섹션의 **부서** 텍스트 상자에 **영업**을 입력하고 **확인**을 선택합니다.

1. **adVM**에 대한 원격 데스크톱 세션 내에서 **Windows PowerShell**을 시작합니다.

1. **관리자**로부터:** Windows PowerShell** 콘솔은 다음을 실행하여 Azure AD Connect 델타 동기화를 시작합니다:

    ```powershell
    Import-Module -Name 'C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1'

    Start-ADSyncSyncCycle -PolicyType Delta
    ```

1. **aduser1 \| 프로필** 블레이드를 표시하는 Internet Explorer 창으로 전환하고 페이지를 새로 고친 후 **부서** 속성이 **영업**으로 설정되어 있음을 유의하십시오.

    >**참고**: **부서** 특성이 설정되지 않은 상태로 유지되면 몇 분 동안 기다렸다가 페이지를 다시 새로 고침하세요.

> **결과**: 이 연습을 완료한 후 디렉터리 동기화용 AD DS를 준비하고, Azure AD Connect를 설치했으며, 디렉터리 동기화를 확인했습니다.


**리소스 정리**

>**참고**: 먼저 Azure AD 동기화를 사용하지 않도록 설정합니다

1. **adVM**에 대한 원격 데스크톱 세션 내에서 관리자로 Windows PowerShell을 시작합니다.

1. Windows PowerShell 콘솔에서 다음을 실행하여 MsOnline PowerShell 모듈을 설치합니다(메시지가 나타날 때 NuGet 공급자에서 대화 상자를 계속하려면 **예**를 입력한 뒤 Enter 키를 누릅니다):

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
    Install-Module MsOnline -Force
    ```

1. Windows PowerShell 콘솔에서 다음을 실행하여 AdatumSync Azure AD 테넌트에 연결합니다(메시지가 표시되면 **syncadmin** 자격 증명으로 로그인합니다):

    ```powershell
    Connect-MsolService
    ```

1. Windows PowerShell 콘솔에서 다음을 실행하여 Azure AD Connect 동기화를 해제합니다.

    ```powershell
    Set-MsolDirSyncEnabled -EnableDirSync $false -Force
    ```

1. Windows PowerShell 콘솔에서 다음을 실행하여 동작이 성공했는지 확인합니다.

    ```powershell
    (Get-MSOLCompanyInformation).DirectorySynchronizationEnabled
    ```

    >**참고**: 결과는 'False'이어야 합니다. 그렇지 않은 경우 1분 정도 기다렸다가 명령을 다시 실행합니다.

    >**참고**: 다음으로 Azure 리소스를 제거

1. Azure Portal에서 **디렉터리 + 구독** 필터를 **adVM** Azure VM을 배포한 Azure 구독과 연결된 Azure AD 테넌트로 설정합니다.

1. Azure Portal 오른쪽 위의 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 

1. Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **PowerShell**을 선택하고 메시지가 표시되면 **확인**을 클릭합니다. 

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 제거합니다.
  
    ```powershell
    Remove-AzResourceGroup -Name "AZ500LAB06" -Force -AsJob
    ```
1. **Cloud Shell** 창을 닫습니다.

    >**참고**: 마지막으로 Azure AD 테넌트를 제거합니다

1. Azure Portal로 돌아가서 **디렉터리 + 구독** 필터를 사용하여 **AdatumSync** Azure Active Directory 테넌트로 전환합니다.

1. Azure Portal에서 **사용자 - 모든 사용자** 블레이드로 이동하여 **syncadmin** 사용자 계정을 나타내는 항목을 클릭합니다. **syncadmin - 프로필** 블레이드에서 **삭제**를 클릭하고 확인하라는 메시지가 나타나면 **확인**을 클릭합니다.

1. 단계를 동일한 순서로 반복하여 **aduser1** 사용자 계정을 삭제합니다.

1. Azure AD 테넌트의 **AdatumSync - 개요** 블레이드로 이동하여 **테넌트 삭제**를 클릭하고 **'AdatumSync' 디렉터리 삭제** 블레이드에서 **Azure 리소스를 삭제할 권한 얻기** 링크를 클릭한 뒤 Azure Active Directory의 **속성** 블레이드에서 **Azure 리소스에 대한 액세스 관리**를 **예**로 설정하고 **저장**을 클릭합니다.

1. Azure Portal에서 로그아웃하고 다시 로그인합니다. 

1. **'AdatumSync' 디렉터리 삭제** 블레이드로 다시 이동하여 **삭제**를 클릭합니다.

> 이 작업에 대한 자세한 내용은 [https://docs.microsoft.com/ko-kr/azure/active-directory/users-groups-roles/directory-delete-howto](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-delete-howto)를 참조하세요.
