---
lab:
    title: '09 - ACR 및 AKS 구성 및 보안'
    module: '모듈 02 - 플랫폼 보호 구현'
---

# 랩 09: ACR 및 AKS 구성 및 보안
# 학생 랩 매뉴얼

## 랩 시나리오

Azure Container Registry 및 Azure Kubernetes Service에 개념 증명을 배포하라는 메시지가 표시됩니다. 특히 개념 증명은 다음을 보여줘야 합니다.

- Dockerfile을 사용하여 이미지를 빌드합니다.
- Azure Container Registry를 사용하여 이미지를 저장합니다.
- Azure Kubernetes Service 구성하기
- 컨테이너 애플리케이션을 내부 및 외부적으로 보호하고 액세스합니다. 

> 이 랩에 있는 모든 리소스의 경우 **미국 동부** 지역을 사용합니다. 이 지역을 수업에 사용할 것인지 강사에게 확인합니다. 

## 랩 목표

이 랩에서는 다음과 같은 연습을 완료합니다.

- 실습 1: ACR 및 AKS 구성 및 보안

## 랩 파일:

- **\\Allfiles\\Labs\\09\\nginxexternal.yaml**
- **\\Allfiles\\Labs\\09\\nginxinternal.yaml**

### 실습 1: ACR 및 AKS 구성 및 보안

### 예상 시간: 45분

> 이 랩에 있는 모든 리소스의 경우 **미국 동부** 지역을 사용합니다. 강사에게 이 지역을 수업에서 사용하는지 확인합니다. 

이 연습에서는 다음 작업을 수행합니다.

- 작업 1: Azure Container Registry 만들기
- 작업 2: Dockerfile 만들기, 컨테이너 빌드 및 Azure Container Registry로 밀어넣기
- 작업 3: Azure Kubernetes Service 클러스터 만들기
- 작업 4: ACR에 액세스할 수 있는 AKS 클러스터 권한 부여
- 작업 5: AKS에 외부 서비스 배포
- 작업 6: 외부 AKS 호스팅 서비스에 액세스할 수 있는지 확인합니다.
- 작업 7: AKS에 내부 서비스 배포
- 작업 8: 내부 AKS 호스팅 서비스에 액세스할 수 있는지 확인

#### 작업 1: Azure Container Registry 만들기

이 작업에서는 랩에 대한 리소스 그룹인 Azure Container Registry를 만듭니다.

1. Azure Portal **`https://portal.azure.com/`** 에 로그인합니다.

    >**참고**: 이 랩에서 사용하는 Azure 구독의 Owner 또는 Contributor 역할과 해당 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할을 가진 계정을 사용하여 Azure Portal에 로그인합니다.

1. Azure Portal 오른쪽 위의 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 메시지가 표시되면 **Bash** 및 **스토리지 만들기**를 클릭합니다.

1. Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **Bash**를 선택했는지 확인합니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 이 랩에 대한 새 리소스 그룹을 만듭니다.

    ```sh
    az group create --name AZ500LAB09 --location eastus
    ```

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 리소스 그룹이 만들어졌는지 확인합니다.

    ```
    az group list --query "[?name=='AZ500LAB09']" -o table
    ```

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 새 ACR(Azure Container Registry) 인스턴스(ACR의 이름은 전역적으로 고유해야 합니다)를 만듭니다. 

    ```sh
    az acr create --resource-group AZ500LAB09 --name az500$RANDOM$RANDOM --sku Basic
    ```

1. Cloud Shell 창의 Bash 세션에서 다음을 실행하여 새 ACR이 만들어졌는지 확인합니다.

    ```sh
    az acr list --resource-group AZ500LAB09
    ```

    >**참고**: ACR의 이름을 기록합니다. 다음 작업에 필요합니다.

#### 작업 2: Dockerfile 만들기, 컨테이너 빌드 및 Azure Container Registry로 밀어넣기

이 작업에서는 Dockerfile을 만들고 Dockerfile에서 이미지를 빌드하고 이미지를 ACR에 배포합니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 Dockerfile을 만들어 Nginx 기반 이미지를 만듭니다. 

    ```sh
    echo FROM nginx > Dockerfile
    ```

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 Dockerfile에서 이미지를 빌드하고 이미지를 새 ACR로 밀어 넣습니다. 

    >**참고**: 명령줄 끝에 있는 후행 기간이 필요합니다. 현재 디렉터리를 Dockerfile의 위치로 지정합니다. 

    ```sh
    ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)

    az acr build --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .
    ```

    >**참고**: 명령이 성공적으로 완료될 때까지 기다립니다. 완료되려면 약 2분이 소요됩니다.

1. Cloud Shell 창을 닫습니다.

1. Azure Portal에서 **AZ500Lab09** 리소스그룹으로 이동한 다음, 리소스 목록에서 이전 작업에서 프로비전한 Azure Container Registry 인스턴스를 나타내는 항목을 클릭합니다.

1. Container Registry 블레이드의 **서비스** 섹션에서 **리포지토리**를 클릭합니다. 

1. 리포지토리 목록에 **sample/nginx**라는 새 컨테이너 이미지가 포함되어 있는지 확인합니다.

1. **sample/nginx** 항목을 클릭하고 이미지 버전을 식별하는**v1** 태그의 현재 상태를 확인합니다.

1. **v1** 항목을 클릭하여 이미지 매니페스트를 확인합니다.

    >**참고**: 매니페스트에는 sha256 다이제스트, 매니페스트 생성 날짜 및 플랫폼 항목이 포함됩니다. 

#### 작업 3: Azure Kubernetes Service 클러스터 만들기

이 작업에서는 Azure Kubernetes Service를 만들고 배포된 리소스를 검토합니다. 

1. Azure Portal에서 Azure Portal 페이지 상단에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **Kubernetes Service**를 입력하고 **Enter** 키를 누릅니다.

1. **Kubernetes Service** 블레이드에서 **+ 추가**를 클릭하고 드롭다운 메뉴에서 **+ Kubernetes 클러스터 추가**를 클릭합니다.

1. **Kubernetes 클러스터 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 남겨둡니다).

    |설정|값|
    |----|----|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |리소스 그룹|**AZ500LAB09**|
    |Kubernetes 클러스터 이름|**MyKubernetesCluster**|
    |지역|**(미국) 미국 동부**|
    |가용성 영역 |**없음**|
    |노드 수|**1**|

1. **다음: 노드 풀 >** 을 클릭하고 **Kubernetes 클러스터 만들기** 블레이드의 **노드 풀** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 남겨둡니다).

    |설정|값|
    |----|----|
    |가상 노드 사용|확인란 선택 취소|
    |VM 확장 집합|확인란 선택 취소|
	
1. **다음: 인증 >** **Kubernetes 클러스터 만들기** 블레이드의 **인증** 탭에서 기본값을 수락하고 **다음: 네트워킹 >** 을 클릭합니다. 

1. **Kubernetes 클러스터 만들기** 블레이드의 **네트워킹** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 남겨둡니다).

    |설정|값|
    |----|----|
    |네트워크 구성|**Azure CNI**|
    |DNS 이름 접두사|유효하고 전역적으로 고유한 DNS 호스트 이름|

    >**참고**: AKS는 프라이빗 클러스터로 구성할 수 있습니다. 이렇게 하면 API 서버에 개인 IP를 할당하여 API 서버와 노드 풀 간의 네트워크 트래픽이 개인 네트워크에만 남아 있는지 확인합니다. 자세한 내용은 [프라이빗 Azure Kubernetes Service 클러스터 만들기](https://docs.microsoft.com/ko-kr/azure/aks/private-clusters) 페이지를 참조하세요.

1. **다음: 통합 >** 그리고 **Kubernetes 클러스터 만들기** 블레이드의 **통합** 탭에서 **컨테이너 모니터링**을 **사용 안 함**으로 설정합니다. 

    >**참고**: 프로덕션 시나리오에서 모니터링을 활성화합니다. 이 경우 모니터링은 랩에서 다루지 않으므로 사용하지 않습니다. 

1. **검토 + 만들기**를 클릭한 다음, **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 10분 정도 걸릴 수 있습니다.

1. 배포가 완료되면 Azure Portal에서 Azure Portal 페이지 상단에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자에서 **리소스 그룹**을 입력하고 **Enter** 키를 누릅니다.

1. **리소스 그룹** 블레이드의 리소스 그룹 목록에서 AKS 노드의 구성 요소를 보유하는 **MC_AZ500LAB09_MyKubernetesCluster_eastus** 라는 새 리소스 그룹을 기록합니다. 이 리소스 그룹에서 리소스를 검토합니다. 
	
1. **리소스 그룹** 블레이드로 돌아와서 **AZ500LAB09** 항목을 클릭합니다. 

    >**참고**: 리소스 목록에 AKS 클러스터 및 해당 가상 네트워크에 기록합니다.

1. Azure Portal의 Cloud Shell에서 Bash 세션을 엽니다. 

    >**참고**: Cloud Shell 창의 왼쪽 위 모서리에 있는 드롭다운 메뉴에서 **Bash**를 선택했는지 확인합니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 Kubernetes 클러스터에 연결합니다.

    ```sh
    az aks get-credentials --resource-group AZ500LAB09 --name MyKubernetesCluster
    ```

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 Kubenetes 클러스터의 노드를 나열합니다. 

    ```sh
    kubectl get nodes
    ```

    >**참고**: 클러스터 노드의 **상태**가 **준비 상태**로 나열되어 있는지 확인합니다.

#### 작업 4: ACR에 액세스하여 해당 가상 네트워크를 관리할 수 있도록 AKS 클러스터 권한 부여

이 작업에서는 ACR에 액세스하여 해당 가상 네트워크를 관리할 수 있도록 AKS 클러스터 권한을 부여합니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 이 랩에서 이전에 만든 Azure Container Registry 인스턴스를 사용하기 위한 AKS 클러스터를 구성합니다. 

    ```sh
    ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)

    az aks update -n MyKubernetesCluster -g AZ500LAB09 --attach-acr $ACRNAME
    ```

    >**참고**: 이 명령은 ACR에 'acrpull' 역할 할당을 부여합니다. 

    >**참고**: 이 명령을 완료하려면 몇 분 정도 걸립니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음 명령을 실행하여 AKS 클러스터에 해당 가상 네트워크에 대한 참가자 역할을 부여합니다. 

    ```sh
    RG_AKS=AZ500LAB09
    
    AKS_VNET_NAME=AZ500LAB09-vnet
    
    AKS_CLUSTER_NAME=MyKubernetesCluster
    
    AKS_VNET_ID=$(az network vnet show --name $AKS_VNET_NAME --resource-group $RG_AKS --query id -o tsv)
    
    AKS_MANAGED_ID=$(az aks show --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --query identity.principalId -o tsv)
    
    az role assignment create --assignee $AKS_MANAGED_ID --role "Contributor" --scope $AKS_VNET_ID
    ```

#### 작업 5: AKS에 외부 서비스 배포

이 작업에서는 매니페스트 파일을 다운로드하고 YAML 파일을 편집하고 변경 내용을 클러스터에 적용합니다. 

1. Cloud Shell 창 내 Bash 세션에서 **파일 업로드/다운로드** 아이콘을 클릭하고 드롭다운 메뉴에서 **업로드**를 클릭합니다. 그런 다음 **열기** 대화 상자에서 랩 파일을 다운로드한 위치로 이동하여 **\\Allfiles\\Labs\\09\\nginxexternal.yaml**을 선택하고 **열기**를 클릭합니다. 그런 후에 **\\Allfiles\\Labs\\09\\nginxinternal.yaml**을 선택하고 **열기**를 클릭합니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 Azure Container Registry 인스턴스의 이름을 식별합니다.

    ```sh
    echo $ACRNAME
    ```

    >**참고**: Azure Container Registry 인스턴스 이름을 기록합니다. 이 작업 뒷부분에서 해당 이름이 필요합니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 nginxexternal.yaml 파일을 열면 콘텐츠를 편집할 수 있습니다. 

    ```sh
    code ./nginxexternal.yaml
    ```

    >**참고**: *외부* yaml 파일입니다.

1. 편집기 창에서 **줄 24**까지 아래로 스크롤하여 **`<ACRUniquename>`** 자리 표시자를 ACR 이름으로 바꿉니다.

1. 편집기 창의 오른쪽 상단 모서리에서 **타원** 아이콘을 클릭하고 **저장**을 클릭한 다음, **편집기 닫기**를 클릭합니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 클러스터에 변경 사항을 적용합니다.

    ```sh
    kubectl apply -f nginxexternal.yaml
    ```

1. Cloud Shell 창 내의 Bash 세션에서 이전 작업에서 실행한 명령 출력을 검토하여 배포 및 해당 서비스가 만들어졌는지 확인합니다. 

    ```
    deployment.apps/nginxexternal created
    service/nginxexternal created
    ```

#### 작업 6: 외부 AKS 호스팅 서비스에 액세스할 수 있는지 확인합니다.

이 작업에서는 공용 IP 주소를 사용하여 컨테이너에 외부에서 액세스할 수 있는지 확인합니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 이름, 유형, IP 주소 및 포트를 포함한 nginxexternal 서비스에 대한 정보를 검색합니다. 

    ```sh
    kubectl get service nginxexternal
    ```

1. Cloud Shell 창 내의 Bash 세션에서 출력을 검토하고 외부 IP 열에서 값을 기록합니다. 다음 단계에서 해당 값이 필요합니다. 

1. Internet Explorer를 열고 이전 단계에서 식별한 IP 주소를 찾아봅니다.

1. **nginx에 오신 것을 환영합니다!** 페이지가 표시되는지 확인합니다. 

#### 작업 7: AKS에 내부 서비스 배포

이 작업에서는 AKS에서 내부 연결 서비스를 배포합니다. 

1. Cloud Shell 창의 Bash 세션에서 다음을 실행하여 nginxintenal.yaml 파일을 열면 콘텐츠를 편집할 수 있습니다. 

    ```sh
    code ./nginxinternal.yaml
    ```

    >**참고**: *내부* yaml 파일입니다.

1. 편집기 창에서 아래쪽의 컨테이너 이미지 참조가 포함된 줄로 스크롤하여 **`<ACRUniquename>`** 자리 표시자를 ACR 이름으로 바꿉니다.

1. 편집기 창의 오른쪽 상단 모서리에서 **타원** 아이콘을 클릭하고 **저장**을 클릭한 다음, **편집기 닫기**를 클릭합니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 클러스터에 변경 사항을 적용합니다.

    ```sh
    kubectl apply -f nginxinternal.yaml
    ```

1.  Cloud Shell 창 내의 Bash 세션에서 출력을 검토하여 배포 및 서비스가 만들어졌는지 확인합니다.

    ```
    deployment.apps/nginxinternal created
    service/nginxinternal created
    ```

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 이름, 유형, IP 주소 및 포트를 포함한 nginxinternal 서비스에 대한 정보를 검색합니다. 

    ```sh
    kubectl get service nginxinternal
    ```

1. Cloud Shell 창 내의 Bash 세션에서 출력을 검토합니다. 이 경우 외부 IP는 프라이빗 IP 주소입니다. 

    >**참고**: 이 IP 주소를 기록합니다. 다음 작업에 필요합니다. 

    >**참고**: 내부 서비스 엔드포인트에 액세스하려면 클러스터에서 실행되는 Pod 중 하나에 대화형으로 연결합니다. 

    >**참고**: 클러스터-IP 주소를 사용할 수도 있습니다.

#### 작업 8: 내부 AKS 호스팅 서비스에 액세스할 수 있는지 확인

이 작업에서는 AKS 클러스터에서 실행되는 Pod 중 하나를 사용하여 내부 서비스에 액세스합니다. 

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 AKS 클러스터의 기본 네임스페이스에 Pod를 나열합니다.

    ```sh
    kubectl get pods
    ```

1. Pod 목록에서 **NAME** 열의 첫 번째 항목을 복사합니다.

    >**참고**: 후속 단계에서 사용할 Pod입니다.

1. Cloud Shell 창 내의 Bash 세션에서 다음을 실행하여 대화형으로 첫 번째 Pod에 연결합니다(`<pod_name>` 자리 표시자를 이전 단계에서 복사한 이름으로 바꿉니다).

    ```sh
    kubectl exec -it <pod_name> -- /bin/bash
    ```

1. Cloud Shell 창의 Bash 세션에서 다음을 실행하여 서비스의 프라이빗 IP 주소를 통해 Nginx 웹 사이트를 사용할 수 있는지 확인합니다(`<internal_IP>` 자리 표시자를 이전 작업에서 기록한 IP 주소로 바꿉니다).

    ```sh
    curl http://<internal_IP>
    ```

1. Cloud Shell 창을 닫습니다.

> 결과: ACR 및 AKS를 구성 및 보호했습니다.


**리소스 정리**

> 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

1. Azure Portal 오른쪽 위의 첫 번째 아이콘을 클릭하여 Cloud Shell을 엽니다. 

1. Cloud Shell 창의 왼쪽 위 드롭다운 메뉴에서 **PowerShell**을 선택하고 메시지가 표시되면 **확인**을 클릭합니다.

1. Cloud Shell 창 내의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 제거합니다.
  
    ```powershell
    Remove-AzResourceGroup -Name "AZ500LAB09" -Force -AsJob
    ```

1.  **Cloud Shell** 창을 닫습니다. 
