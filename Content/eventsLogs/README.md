# Eventos e logs para troubleshooting

[Aqui na documentação oficial](https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/) você deve encontrar uma lista com os comandos mais úteis para o dia a dia.

## Listando pods

O primeiro passo é listar os pods que você deseja validar, você pode listar todos ou selecionar apenas pela aplicação, com base no selector do deployment:


```
kubectl get pods --namespace default
```

```
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79f774bdb9-7jl89       2/2     Running   12         9d
details-v1-79f774bdb9-9jjzr       2/2     Running   12         9d
productpage-v1-6b746f74dc-ql2mg   2/2     Running   12         9d
productpage-v1-6b746f74dc-r2mpm   2/2     Running   10         9d
ratings-v1-b6994bb9-22brt         2/2     Running   12         9d
ratings-v1-b6994bb9-2k9x7         2/2     Running   12         9d
reviews-v1-545db77b95-8jrm8       2/2     Running   12         9d
reviews-v1-545db77b95-hbh97       2/2     Running   12         9d
reviews-v2-7bf8c9648f-6kz7c       2/2     Running   12         9d
reviews-v2-7bf8c9648f-gh648       2/2     Running   12         9d
reviews-v3-84779c7bbc-rlrxc       2/2     Running   12         9d
reviews-v3-84779c7bbc-vfm4k       2/2     Running   12         9d
```

Para listar apenas a aplicação reviews:

```
kubectl get pods --selector app=reviews
```

```
NAME                          READY   STATUS    RESTARTS   AGE
reviews-v1-545db77b95-8jrm8   2/2     Running   12         9d
reviews-v1-545db77b95-hbh97   2/2     Running   12         9d
reviews-v2-7bf8c9648f-6kz7c   2/2     Running   12         9d
reviews-v2-7bf8c9648f-gh648   2/2     Running   12         9d
reviews-v3-84779c7bbc-rlrxc   2/2     Running   12         9d
reviews-v3-84779c7bbc-vfm4k   2/2     Running   12         9d
```

Com base nestas informações, já conseguimos os nomes dos pods e sabemos se eles estão em execução ou não.

## Coletando eventos

A coleta de eventos auxilia em muitos casos, em certos momentos conseguimos através do Events, confirmar um possível problema, como erro ao fazer pull de uma imagem do registry (ECR).

Para coletar os eventos de um deployment ou pod, basta executar o comando describe e olhar a última guia que fala sobre Events:

```
kubectl describe deployments.apps --namespace default reviews-v1 
```

```
Name:                   reviews-v1
Namespace:              default
CreationTimestamp:      Tue, 04 Jan 2022 11:09:47 -0300
Labels:                 app=reviews
                        version=v1
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=reviews,version=v1
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=reviews
                    version=v1
  Service Account:  bookinfo-reviews
  Containers:
   reviews:
    Image:      docker.io/istio/examples-bookinfo-reviews-v1:1.16.2
    Port:       9080/TCP
    Host Port:  0/TCP
    Environment:
      LOG_DIR:  /tmp/logs
    Mounts:
      /opt/ibm/wlp/output from wlp-output (rw)
      /tmp from tmp (rw)
  Volumes:
   wlp-output:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
   tmp:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  reviews-v1-545db77b95 (2/2 replicas created)
NewReplicaSet:   <none>
Events:          <none>
```

## Coletando logs do pod ou container

Para coletar os logs de um pod basta passar o nome do mesmo

```
kubectl logs reviews-v1-545db77b95-8jrm8 --namespace default
```

Para coletar o log de um container específico dentro do pod, é necessário passar o parâmetro -c

```
kubectl logs reviews-v1-545db77b95-8jrm8 -c istio-proxy --namespace default
```

Para coletar os logs de todos os pods, é possível filtrar através da label no selector

```
kubectl logs --selector app=reviews --namespace default
```
