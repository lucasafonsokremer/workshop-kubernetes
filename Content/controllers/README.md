# Os principais controllers 

"Na robótica e em automações, um loop de controle é um loop sem fim que regula o estado de um sistema." Definição disponível na [documentação oficial](https://kubernetes.io/docs/concepts/architecture/controller/).

**Quando utilizamos diretamente o docker e o container morre, você normalmente necessita intervir. Com o controller, é como se um supervisor cuidasse da saúde e estado deste container, em tempo integral**

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

O Deployment é um "supervisor".

O Deployment é um dos principais senão o principal controller do seu cluster. Com ele podemos definir em mais alto nível o estado de um pod e um replicaset. 

<p align="center">
  <img src="https://www.bluematador.com/hs-fs/hubfs/blog/new/Kubernetes%20Deployments%20-%20Rolling%20Update%20Configuration/Kubernetes-Deployments-Rolling-Update-Configuration.gif?width=1600&name=Kubernetes-Deployments-Rolling-Update-Configuration.gif"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

Nele, possuímos três pontos importates:

* Especificações do deployment, área não existente quando criamos o exemplo do pod;
* Definição da quantidade de réplicas nas especificações do deployment;
* Definição de selector nas especificações do deployment. Através dele o service vai conseguir identificar qual pod ele precisa encaminhar a requisição.

Exemplo de um deployment que faz a mesma criação do pod do exemplo anterior, mas agora controlamos a quantidade de réplicas e passamos outras definições importantes.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
 
Seguindo com o Deployment, possuímos alguns pontos importantes que SEMPRE precisam ser aplicados, como por exemplo:

* Limitação de CPU e memória (até o momento, o Kubernetes gerencia nativamente apenas estes dois)
* Uso do liveness e readiness probes

### Limitação de recursos

Para limitar recursos, trabalhamos com limits e requests, com ambos o escalonador do Kubernetes consegue tomar as melhores decisões. Por padrão, quando um cluster está sobrecarregado, os pods que excedem suas requests serão encerrados antes dos que excederem seus limits.

O requests e limits funcionam como um soft e hard limit de uso de hardware. Caso um serviço venha consumir mais que o limits, o próprio OOM Killer do Linux irá se encarregar de encerrar o pod.

Um ponto importante é que o requests é o mínimo necessário para subir sua aplicação, caso este recurso não esteja disponível no node, o pod não entrará em execução.

O uso de ResourceQuota e LimitRange é algo que precisa ser utilizado com cuidado, já que cada aplicação possui a sua particularidade, e limitar tão alto nível pode ser catastrófico.

* Requests e Limits

```
spec:
  containers:
  - name: demo
    image: cloudnatived/demo:hello
    ports:
    - containerPort: 8888
    resources:
      requests:
        memory: "10Mi"
        cpu: "100m"
      limits: 
        memory: "20Mi" 
        cpu: "250m"
```

<p align="center">
  <img src="https://blog.kubecost.com/assets/images/k8s-recs-ands-limits.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

### Health Check da aplicação com liveness e readiness probes

O Health Check é peça fundamental para redução do downtime de uma aplicação. No Kubernetes possuímos dois probes importantes que seriam:

* liveness
* readiness

Cada um com sua função e que muitas das vezes, vemos apenas um deles descritos em yamls, mas é uma boa prática ter ambos pelos seguintes motivos:

* liveness = Neste probe, o Kubernetes avalia se o container ou pod está "vivo" através do protocolo e path, **caso ele não responda as requisições, ele será reiniciado**
* readiness = Um pod cujo o readiness probe falhe, não será adicionado ou será removido de qualquer services que corresponda este pod, ele basicamente valida se o pod está "pronto". **Em outras palavras, este probe é responsável garantir que um pod não saudável não receba tráfego.**

```
spec:
  containers:
  - name: demo
    image: cloudnatived/demo:hello
    ports:
    - containerPort: 8888
    resources:
      requests:
        memory: "10Mi"
        cpu: "100m"
      limits: 
        memory: "20Mi" 
        cpu: "250m"
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8888
      initialDelaySeconds: 10
      periodSeconds: 6
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8888
      periodSeconds: 4
      timeoutSeconds: 5
      failureThreshold: 1
```

<p align="center">
  <img src="https://miro.medium.com/max/1400/1*fdNZeCl7TESpFtrCcZKYcg.gif"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## Namespaces

[Documentação oficial](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

É sempre conveniente trabalharmos com vários namespaces em um cluster, afim de agrupar recursos relacionados e facilitar trabalhar com eles dentro de um cluster. Normalmente se trabalha separando por aplicações ou times.

Os namespaces não podem ser aninhados. Mas é possível realizar o bloqueio do trafego de entrada e saída, além da limitação de recursos com ResourceQuota e LimitRange.

Outra boa prática é não rodar ambientes de dev, qa e produção, dentro do mesmo cluster separados por namespaces, já que algumas políticas de PCI Security Standards Council proíbem esta prática.

Exemplo de um namespace:

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
```

## Service Account

[Documentação oficial](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

Service Account é um objeto importante do cluster. Ela está presente em todos os pods, mesmo você não especificando. Por padrão quando criamos um pod, se a service account não for parametrizada no deployment, é atribuida a service account "default" daquele namespace. Portanto é uma boa prática de segurança, criar service accounts para cada workload, evitando-se assim atribuir permissões de uma forma generalizada dentro do cluster, sem a intenção.

No Kubernetes, não existe o recurso "usuário", para o Kubernetes, um usuário é um indivíduo em posse de um certificado e uma chave, portanto ele é algo independente do cluster e pode ser gerenciado por exemplo pelo cloud provider.

Uma Service Account por outro lado, é um recurso gerenciado pela API do Kubernetes, com ele é possível que processos internos de um pod se comuniquem com o apiserver do Kubernetes e tomem alguma ação. Um exemplo prático seria um Kubernetes Operator, muito utilizado para gerir aplicações complexas.

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: build-robot
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

## Services

[Documentação oficial](https://kubernetes.io/docs/concepts/services-networking/service)

Uma das principais funcionalidades do Kubernetes, é o balanceamento de carga automático que funciona graças ao services, você possui uma forma de expor seu serviço já com DNS devidamente configurado, seja através de ClusterIP, NodePort ou LoadBalancer para então conseguir distribuir as requisições entre diversos pods do seu deployment por exemplo.

O Service consegue este balanceamento, já que outro controller, o Endpoints, possui a lista com todos os endereços de pods de um deployment em específico.

<p align="center">
  <img src="https://miro.medium.com/max/700/1*KIVa4hUVZxg-8Ncabo8pdg.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

Ao expor um serviço, precisamos ter em mente que:

* ClusterIP = É a opção padrão. Acessível apenas de dentro do cluster e também é possível resolver o DNS dele internamente
* NodePort = Abre uma porta alta externa, em todos os nós e as requisições que chegam ali, são redirecionadas para dentro do cluster. Quando criamos um serviço do tipo NodePort, ele automaticamente cria um ClusterIP e no topo, um NodePort.
* LoadBalancer = Muito utilizado quando é necessário uma iteração externa, como um cloud provider. O serviço LoadBalancer é criado no topo do NodePort e ClusterIP.

[Maiores informações na documentação oficial](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: backend
  name: backend
spec:
  containers:
  - image: nginx
    name: backend
  dnsPolicy: ClusterFirst
  restartPolicy: Always

---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: frontend
  name: frontend
spec:
  containers:
  - image: nginx
    name: frontend
  dnsPolicy: ClusterFirst
  restartPolicy: Always

---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: backend
  name: backend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: backend

---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: frontend
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: frontend
  type: ClusterIP
```

Lembrando que ambos os serviços, embora o backend não esteja diretamente especificado mas é o valor padrão, são ClusterIP, portanto são serviços que estão acessíveis apenas de dentro do cluster.

Para testar se está tudo ok basta:

```
kubectl exec frontend -- curl -Is backend
```

Supondo agora que o serviço rabbitMQ se encontre em outro namespace, posso fazer o seguinte para consumir o serviço do meu frontend:

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: rabbitnamespace

---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: rabbit
  name: rabbit
  namespace: rabbitnamespace
spec:
  containers:
  - image: nginx
    name: rabbit
  dnsPolicy: ClusterFirst
  restartPolicy: Always

---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: frontend
  name: frontend
spec:
  containers:
  - image: nginx
    name: frontend
  dnsPolicy: ClusterFirst
  restartPolicy: Always

---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: rabbit
  name: rabbit
  namespace: rabbitnamespace
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: rabbit

---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: frontend
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: frontend
  type: ClusterIP
```

Para acessar basta:

```
kubectl exec frontend -- curl -Is rabbit.rabbitnamespace
```

## Daemonset

[Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

O Daemonset possui uma pequena diferença se comparado ao Deployment, nele não especificamos a quantidade de réplicas que serão criadas. Basicamente um pod será escalado em cada node do cluster.

Normalmente o Daemonset é utilizado para aplicações que necessitem estar vinculadas algum node, como um sistema de monitoração ou logs.

## Horizontal pod autoscalers

Horizontal pod autoscaler é uma das features mais importantes para criar elasticidade na aplicação. Com ele é possível que um Deployment ou StatefulSet se adapte com base em um ou mais parâmetros, afim de suportar o aumento de demanda no ambiente.

Nele especificamos números de mínimo, máximo e com base em que métrica o deployment terá uma maior quantidade de réplicas.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu:latest
        command: [ "/bin/bash", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        resources:
          # É necessário especificar requests para o hpa atuar
          # baseado em uso de CPU
          requests:
            cpu: "250m"

---
# A partir da versão 1.23 utilizar "autoscaling/v2"
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: ubuntu
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ubuntu
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 100Mi
```

Agora para testar se tudo está ok vamos fazer um teste com o stress:

```
# validar nome do pod
kubectl get pods
# conectar no pod
kubectl exec -it ubuntu-5d9d76f597-vflnl -- /bin/bash
# agora dentro do pod instalar o stress e rodar o comando
apt-get update && apt-get install stress -y && stress --cpu 3 --vm 1 --timeout 60
```

Em poucos instantes podemos ver o HPA atuando

```
kubectl get horizontalpodautoscalers.autoscaling
```

**O scale down irá levar alguns minutos**

Outro ponto importante é que é possível fazer uso do HPA com métricas customizadas, como HTTP Requests, que é muito melhor que apenas CPU e memória:

[Documentação das métricas customizadas](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#scaling-on-custom-metrics)

# Recomendo também a leitura de outros controllers importantes

Você pode começar por:

* [Statefulsets - Um método para criar apps stateful](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [Replicasets - Controller utilizado na gestão de pods](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* [Secrets - Um método de armazenar e consumir segredos, do cluster ou da cloud](https://kubernetes.io/docs/concepts/configuration/secret/)
* [Configmaps - Um método de armazenar e consumir arquivos de conf ou variáveis](https://kubernetes.io/docs/concepts/configuration/configmap/)
* [Storageclasses - Driver de conexão a sistemas de arquivos externos](https://kubernetes.io/docs/concepts/storage/storage-classes/)
* [Persistent volumes - Pedaço de storage que consome o storageclass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [Persistent volumes claims - Abstração de storage para a aplicação](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [Cronjobs - Jobs que necessitam nascer, executar e morrer em determinados horários](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
* [Endpoints - Responsável pelo balanceamento do services](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Poddisruptionbudgets - Forma de controle de recursos](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
