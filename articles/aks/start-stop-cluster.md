---
title: Azure Kubernetes 서비스 (AKS) 시작 및 중지
description: AKS (Azure Kubernetes Service) 클러스터를 중지 하거나 시작 하는 방법에 대해 알아봅니다.
services: container-service
ms.topic: article
ms.date: 09/24/2020
author: palma21
ms.openlocfilehash: 87d51f9c1d084faf79c7ec1cf1255a6fb3c8245d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2021
ms.locfileid: "103201012"
---
# <a name="stop-and-start-an-azure-kubernetes-service-aks-cluster"></a>AKS (Azure Kubernetes Service) 클러스터를 중지 하 고 시작 합니다.

업무 시간 중에만 사용 되는 개발 클러스터와 같이 AKS 워크 로드를 지속적으로 실행할 필요가 없을 수도 있습니다. 이로 인해 Azure Kubernetes 서비스 (AKS) 클러스터가 유휴 상태일 수 있으며 시스템 구성 요소를 초과 하지 않는 시간이 발생 합니다. [모든 `User` 노드 풀을 0으로 확장](scale-cluster.md#scale-user-node-pools-to-0)하여 클러스터 공간을 줄일 수 있지만, 클러스터가 실행되는 동안에는 시스템 구성 요소를 실행 하는 데에도 [ `System` 풀이](use-system-pools.md) 필요 합니다. 이러한 기간 동안 비용을 더 최적화 하려면 클러스터를 완전히 해제(중지) 할 수 있습니다. 이 작업을 수행하면 제어 평면과 에이전트 노드가 모두 중지 되므로 모든 계산 비용을 절감할 수 있을 뿐만 아니라 다시 시작할 때에 저장된 모든 개체 및 클러스터 상태를 유지할 수 있습니다. 그런 다음, 주말의 왼쪽에 있는 위치를 선택 하거나 batch 작업을 실행 하는 동안에만 클러스터를 실행할 수 있습니다.

## <a name="before-you-begin"></a>시작하기 전에

이 문서에서는 기존 AKS 클러스터가 있다고 가정합니다. AKS 클러스터가 필요한 경우 AKS 빠른 시작 [Azure CLI 사용][aks-quickstart-cli] 또는 [Azure Portal 사용][aks-quickstart-portal]을 참조하세요.

### <a name="limitations"></a>제한 사항

Cluster start/stop 기능을 사용 하는 경우 다음 제한 사항이 적용 됩니다.

- 이 기능은 Virtual Machine Scale Sets 지원 되는 클러스터에 대해서만 지원 됩니다.
- 중지 된 AKS 클러스터의 클러스터 상태는 최대 12 개월 동안 보존 됩니다. 클러스터가 12 개월 넘게 중지 된 경우 클러스터 상태를 복구할 수 없습니다. 자세한 내용은 [AKS 지원 정책](support-policies.md)을 참조 하세요.
- 중지 된 AKS 클러스터를 시작 하거나 삭제할 수만 있습니다. 크기 조정 또는 업그레이드와 같은 작업을 수행 하려면 먼저 클러스터를 시작 합니다.

## <a name="stop-an-aks-cluster"></a>AKS 클러스터 중지

명령을 사용하여 `az aks stop` 실행 중인 AKS 클러스터의 노드 및 제어 평면을 중지할 수 있습니다. 다음 예제에서는 *myAKSCluster* 라는 클러스터를 중지 합니다.

```azurecli-interactive
az aks stop --name myAKSCluster --resource-group myResourceGroup
```

[az aks show][az-aks-show] 명령을 사용하여 클러스터가 중지 된 시기를 확인하고 `powerState` `Stopped` 아래 출력에 표시 된 것 처럼 표시 되는지 확인할 수 있습니다.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Stopped"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

`provisioningState` `Stopping` 이 표시되면 클러스터가 아직 완전히 중지 되지 않았음을 의미 합니다.

> [!IMPORTANT]
> [Pod 중단 예산을](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) 사용하는 경우 드레이닝 프로세스를 완료 하는 데 시간이 더 오래 걸릴 수 있으므로 중지 작업에 시간이 더 오래 걸릴 수 있습니다.

## <a name="start-an-aks-cluster"></a>AKS 클러스터 시작

명령을 사용하여 `az aks start` 중지 된 AKS 클러스터의 노드 및 제어 평면을 시작할 수 있습니다. 클러스터가 이전 제어 평면 상태 및 에이전트 노드 수로 다시 시작 됩니다.  
다음 예제에서는 *myAKSCluster* 라는 클러스터를 시작 합니다.

```azurecli-interactive
az aks start --name myAKSCluster --resource-group myResourceGroup
```

[az aks show][az-aks-show] 명령을 사용하여 클러스터가 시작 된 시기를 확인 하 고 `powerState` `Running` 아래 출력에 표시 된 것으로 확인 합니다.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Running"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

`provisioningState` `Starting` 이 표시되면 클러스터가 아직 완전히 시작 되지 않았음을 의미 합니다.

## <a name="next-steps"></a>다음 단계

- 풀 크기를 0으로 조정 하는 방법에 대한 자세한 `User` 내용은 [ `User` 풀 크기 조정](scale-cluster.md#scale-user-node-pools-to-0)을 참조 하세요.
- 스폿 인스턴스를 사용하여 비용을 절감 하는 방법을 알아보려면 [AKS에 별색 노드 풀 추가](spot-node-pool.md)를 참조 하세요.
- AKS 지원 정책에 대한 자세한 내용은 [AKS 지원 정책](support-policies.md)을 참조 하세요.

<!-- LINKS - external -->

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-show]: /cli/azure/aks#az_aks_show
