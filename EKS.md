# EKS Cluster 구성

이번에는 `EKS`를 활용해 `Wordpress`를 배포하고 모니터링 환경을 구축하는 실습을 진행하겠습니다.

`EKS`와 `Kubernetes`에 대해 잘 모르시는 분들께서는 이 글들을 참고해 주세요.
> [Kubernets 구조](https://velog.io/@squarebird/Kubernetes-Kubernetes-%EA%B5%AC%EC%A1%B0)

#### 기술 스택
>
1. AWS EKS
2. Wordpress + Mysql
3. Grafana + Prometheus

`EKS`를 통해 `Kubernetes`환경에 Wordpress 애플리케이션을 배포하겠습니다.
배포한 뒤, `Grafana`와 `Prometheus`를 배포해 모니터링을 수행하겠습니다.

---

## 1. 네트워크 및 보안그룹 구성

먼저 `EKS Cluster`를 구축할 네트워크를 구성하겠습니다.

>
VPC : 10.0.0.0/16
bastion-Subnet : 10.0.0.0/24
Subnet-01 : 10.0.2.0/24
Subnet-02 : 10.0.3.0/24

Service라는 이름의 `VPC`를 구성한 뒤, 2개의 `Subnet`을 구성하겠습니다.

![](https://velog.velcdn.com/images/squarebird/post/71e6b43e-0547-446c-929c-e88db9733b7a/image.png)

0.0.0.0/0으로 가는 모든 트래픽은 `Internet Gateway`로 라우팅 하도록 설정하고, `Subnet` 두 개를 해당 라우팅 테이블에 연결하겠습니다.

다음으로 보안그룹을 구성하겠습니다.

![](https://velog.velcdn.com/images/squarebird/post/964f8959-4cbd-4824-888c-dcdb5d807e70/image.png)

원격 연결과 WEB서비스 접근이 가능하도록 22와 80 포트를 오픈하겠습니다.

SSH에 대해 테스트용도로 소스IP를 0.0.0.0/0으로 설정했지만, 보안을 위해 특정 IP를 설정하는 것을 권장합니다.

---

## 2. 권한 설정

`AWS`의 어떤 서비스를 사용하던, 가장 먼저 해야할 일은 권한에 대한 설정입니다.

![](https://velog.velcdn.com/images/squarebird/post/7ed9425a-83a7-4d17-9d00-fdaa62509e72/image.png)

먼저, `IAM`에서 역할을 만들어 줍시다.

`EKS`에서는 크게 두 가지 역할이 필요합니다.

`EKS Cluster`와 `EKS WorkerNode`가 가져야 하는 역할입니다.

> **eks-cluster-role**
- AmazonEKSClusterPolicy
- AmazonEKSServicePolicy

> **eks-worker-role**
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly
- CloudWatchLogsFullAccess
- AmazonEKS_CNI_Policy
- AmazonRoute53FullAccess

![](https://velog.velcdn.com/images/squarebird/post/ae3bd513-42f8-48df-88e5-3ae939d32d9f/image.png)

`eks-cluster-role`과 `eks-worker-role`이라는 두 역할을 만들고, 위에 나열된 정책들을 적용시켜 줍니다.

---

