# Os principais workloads e controllers 

"Na robótica e em automações, um loop de controle é um loop sem fim que regula o estado de um sistema." Definição disponível na [documentação oficial](https://kubernetes.io/docs/concepts/architecture/controller/).

## Pod

[Documentação oficial](https://kubernetes.io/docs/concepts/workloads/pods/)

O Pod é a menor instância do Kubernetes, mas qual a diferença entre o pod e um container? **Basicamente um pod pode ser composto por um ou mais containers**. Mas no fim, qual o benefício disso? Basicamente com esta topologia, você consegue manter a ideia de responsabilidade mínima de cada container e não transformar o mesmo, em uma máquina virtual com N funções. Isso tudo é possível graças ao namespace, funcionalidade do kernel linux que explicamos anteriormente, no qual **um pod, compartilha os namespaces entre todos os containers**.

Como exemplo podemos ter a seguinte topologia, no qual um pod possui dois containers rodando nele, um com a sua aplicação devidamente empacotada e em execução e outro container que é responsável por ler os logs gerados pela aplicação e enviar para um centralizador de logs por exemplo. Isso tudo é possível já que eles compartilham do mesmo namespace, portanto os dados que o container A cria, o container B consegue coletar, pois eles compartilham da mesma árvore de processos de disco, rede, usuário e afins.

Para criar um pod com apenas um container é simples, podemos criar um exemplo com o seguinte manifesto:

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Agora basta executar o seguinte comando:

```
kubectl create -f pod.yaml
```

Para remover basta:

```
kubectl delete -f pod.yaml
```

## Deployment

[Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

O Deployment é um dos principais senão o principal controller do seu cluster. Com ele podemos definir em mais alto nível o estado de um pod e um replicaset. Nele, possuímos três pontos importates:

* Agora possuímos as especificações do deployment;
* Definição da quantidade de réplicas nas especs do deployment;
* Definição de selector nas specs do deployment.

Os principais usos de um deployment seriam em alto nível:

* 

## Daemonset

## Service Account

## Services

Uma das principais funcionalidades do Kubernetes, é o balanceamento de carga automático que funciona graças ao services, você possui uma forma de expor seu serviço, seja através de ClusterIP, NodePort ou LoadBalancer para então conseguir distribuir as requisições entre diversos pods do seu deployment por exemplo.

O Service consegue este balanceamento, já que outro controller, o Endpoints, possui a lista com todos os endereços de pods, de um deployment em específico.

Ao expor um serviço, precisamos ter em mente que:

* ClusterIP = É a opção padrão. Acessível apenas de dentro do cluster e também é possível resolver o DNS dele internamente
* NodePort = Abre uma porta alta externa, em todos os nós e as requisições que chegam ali, são redirecionadas para dentro do cluster. Quando criamos um serviço do tipo NodePort, ele automaticamente cria um ClusterIP e no topo, um NodePort.
* LoadBalancer = Muito utilizado quando é necessário uma iteração externa, como um cloud provider. O serviço LoadBalancer é criado no topo do NodePort e ClusterIP.

## Secrets

## Configmap

## Cronjobs

## Horizontal pod autoscalers

## Persistent volumes

## Persistent volumes claims

## Recomendo também a leitura de outros temas importantes

Você pode começar por:

* [Statefulsets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [Replicasets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* [Endpoints](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Poddisruptionbudgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
* [Storageclasses](https://kubernetes.io/docs/concepts/storage/storage-classes/)
