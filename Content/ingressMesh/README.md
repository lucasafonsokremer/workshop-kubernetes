# Ingress e Service Mesh

Antes de começarmos, o seguinte [livro](https://microservices.io/book) foi utilizado como base para várias destas fundamentações.

## Definição de uma API

- "É uma interface projetada para ser invocada em uma rede que permite outro usuário obter acesso programático dos dados e funcionaliades" 

## Confusão sobre utilização de um service mesh e um API Gateway

- Um service mesh como o Istio possui várias funcionalidades que um API Gateway por exemplo também possui, devido a esta sobreposição é comum uma certa confusão e acreditar que um substitui o outro.

- Onde estas ferramentas se sobrepoem:

```
Coleta de telemetria
Rastreamento distribuído
Descoberta de serviço
Balanceamento de carga
TLS Termination
Validação de JWT
Solicitar roteamento
Divisão de tráfego
Canary release
Mirroring
Rate limit
```

## Onde o service mesh diverge de um API Gateway

O service mesh opera em um nivél mais baixo que o API Gateway. Ele basicamente trabalha mais horizontalmente (leste-oeste) dentro do meu cluster. Basicamente um service mesh visa resolver problemas como timeout, rate limit, telemetria, roteamento, security, balanceamento de carga, descoberta de serviço e várias outras coisas para qualquer serviço / aplicativo que faz uso dele de forma transparente a nível de camada 7. Em outras palavras, o mesh se funde ao serviço afim de adicionar mais capacidades (super poderes) sem adição de código ao serviço.

O API Gateway por outro lado, desempenha o papel de abstrair detalhes e desacopla as implementações, onde ele basicamente cria uma abstração para o nosso usuário final sobre a arquitetura do nosso aplicativo, já que em sistemas distribuídos, baseados em microsserviços ou até mesmo funções lambda é fácil de perder a noção de onde minha aplicação está executando e como devo chamá-la. Portanto um API Gateway vive acima dos aplicativos e serviços, gerenciando o tráfego norte-sul do cluster.

Além disso um API Gateway atua em três recursos que um service mesh não atua:

- Desacoplamento de fronteira
- Controle de tráfego norte-sul rígido
- Faz a ponte de domínios de confiança e segurança

<p align="center">
  <img src="https://media1-production-mightynetworks.imgix.net/asset/24216529/Gateway_vs_Service_Mesh.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## Onde o service mesh e o API Gateway se transformam em uma solução robusta

- Vários API Gateways do mercado e até alguns service mesh, utilizam o Envoy como proxy, um poderoso proxy que proporciona vários filtros a nível de camada 7, possibilitando rotear o usuário para serviços em bare metal, VMs e Kubernetes. Essa combinação de abordagem permite uma integração fim a fim, onde o API Gateway possui suas responsabilidades no sentido norte-sul enquanto o service mesh atua no sentido leste-oeste. Com isso o maior desafio está em combinar este ferramental e que ambos possuam definições de limitações bem claras, para evitar assim qualquer sobreposição de tarefas.

- Além disso, encontrar serviços de API Gateway que executem dentro do seu cluster Kubernetes, potencializa ainda mais as questões de segurança, já que desta forma é possível montar um sistema zero trust onde o API Gateway faz o uso do TLS do service mesh, com isto não é necessário adicionar algo chamado last mile signature no API Gateway externo, como por exemplo um AWS API Gateway.

## O API Management

- O API Management visa resolver o problema de "e quando desejarmos expor nossas APIs para outros consumirem?", com rastreamento, políticas de permissão, fluxos de segurança, catálogos de serviços e governança. Além disso podemos com ele compartilhar APIs de acordo com os nossos termos, impondo limites claros para todos ou para um grupo. Alguns serviços de API Management também permitem cobrança com preço específico até certo volume de requests.

- Normalmente um API Management acompanha de um console web, com um admin portal e um banco para armazenar o estado da estrutura.

- Exemplos de software para API Management:

  - Google Apigee
  - Red Hat 3Scale
  - Mulesoft
  - Kong

<p align="center">
  <img src="https://blog.christianposta.com/images/identity-crisis/api-management-sketch.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## O Ingress Controller

Com este serviço, podemos querer um tipo de "gateway de entrada" para nossa estrutura, já que ele será responsável por ser o sentinela do tráfego e atuar como proxy para as aplicações dentro do nosso cluster. Alguns ingress controllers, tem a capacidade de expor por exemplo um monolito, ou aplicações em microsserviços dentro do Kubernetes, serviços gRPC, caches, filas de mensageria e até mesmo banco de dados.

Exemplos de ingress controller:

- Baseadas no Envoy:
  - Istio
  - Ambassador
  - Gloo
- Outras tecnologias:
  - HAProxy
  - Nginx
  - Traefik

<p align="center">
  <img src="https://blog.christianposta.com/images/identity-crisis/cluster-ingress-sketch.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## O API Gateway

No pattern de API Gateway, estamos basicamente simplificando explicitamente a chamada de um grupo de APIs para emular o uso de um aplicativo para um conjunto de usuários, clientes ou consumidores. Em outras palavras, um API Gateway está muito mais próximo da visão do mundo do desenvolvedor e com menos foco em qual porta ou serviço será exposta. O API Gateway também é diferente do API Management, já que o API Gateway pode misturar chamadas de API ao backend com chamadas RPC, sistemas legados ou até mesmo chamadas para outros serviços como funções lambda.

Algumas responsabilidades de um API Gateway:

- URL mapping and rewriting
- Rate limit
- API key management
- Authorization service
- HTTP cache

Exemplos de API Gateway:

  - Gloo Edge
  - Emissary Ingress
  - Spring Cloud Gateway
  - Netflix Zuul

<p align="center">
  <img src="https://blog.christianposta.com/images/identity-crisis/api-gateway-pattern.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## Lab: Criando e expondo serviços com um API Gateway

Observação: O API Gateway vai demandar de um serviço tipo Load Balancer, caso você esteja atuando em ambiente onpremise é necessário instalar o MetalLB conforme [esta documentação](https://github.com/lucasafonsokremer/k8s-dev-env-with-kind/blob/main/Content/metallb/README.md)

### Instalando o glooctl

- Primeiramente instalar o glooctl

```
curl -sL https://run.solo.io/gloo/install | sh
export PATH=$HOME/.gloo/bin:$PATH
```

- Validar se está funcional

```
glooctl version
```

### Instalando o Gloo Edge (Open Source Edition)

- Instalando o repo helm

```
helm repo add gloo https://storage.googleapis.com/solo-public-helm
helm repo update
kubectl create namespace gloo-system
```

- Instalando o gloo

```
helm install gloo gloo/gloo --namespace gloo-system
```

- Validando a instalação

```
kubectl get all -n gloo-system
```

```
NAME                                READY     STATUS    RESTARTS   AGE
pod/discovery-f7548d984-slddk       1/1       Running   0          5m
pod/gateway-proxy-9d79d48cd-wg8b8   1/1       Running   0          5m
pod/gloo-5b7b748dbf-jdsvg           1/1       Running   0          5m



NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                     AGE
service/gateway-proxy   LoadBalancer   10.97.232.107   192.168.10.100    80:30221/TCP,443:32340/TCP  5m 
service/gloo            ClusterIP      10.100.64.166   <none>            9977/TCP,9988/TCP,9966/TCP  5m



NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/discovery       1/1     1            1           5m
deployment.apps/gateway-proxy   1/1     1            1           5m
deployment.apps/gloo            1/1     1            1           5m



NAME                                      DESIRED   CURRENT   READY     AGE
replicaset.apps/discovery-f7548d984       1         1         1         5m
replicaset.apps/gateway-proxy-9d79d48cd   1         1         1         5m
replicaset.apps/gloo-5b7b748dbf           1         1         1         5m
```

### Criando uma aplicação de testes

- Vamos instalar uma aplicação do próprio gloo

```
kubectl apply -f https://raw.githubusercontent.com/solo-io/gloo/v1.11.x/example/petstore/petstore.yaml
```

- Validando a instalação

```
kubectl -n default get pods
kubectl -n default get svc petstore
```

- Validar se o discovery colocou o upstream como ativo

```
glooctl get upstreams | grep petstore
```

- Agora basta criar uma rota para o serviço

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: default
  namespace: gloo-system
  ownerReferences: []
status:
  reportedBy: gateway
  state: Accepted
  subresourceStatuses:
    '*v1.Proxy.gloo-system.gateway-proxy':
      reportedBy: gloo
      state: Accepted
spec:
  virtualHost:
    domains:
    - '*'
    routes:
    - matchers:
      - exact: /all-pets
      options:
        prefixRewrite: /api/pets
      routeAction:
        single:
          upstream:
            name: default-petstore-8080
            namespace: gloo-system
```

```
kubectl create -f virtualservicepetstore.yaml
```

- Por fim podemos testar o serviço acessando o api gateway

```
curl $(glooctl proxy url)/all-pets
```

- O retorno deve ser o seguinte

```
[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```

- Em caso de erro você pode validar as configurações do seu API Gateway da seguinte forma

```
glooctl check
```

### External Authorization com OPA

- Vamos primeiramente criar outro app de testes

```
kubectl create -f https://raw.githubusercontent.com/istio/istio/master/samples/httpbin/httpbin.yaml
```

- Agora vamos criar o roteamento da aplicação de testes

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: default
  namespace: gloo-system
  ownerReferences: []
status:
  reportedBy: gateway
  state: Accepted
  subresourceStatuses:
    '*v1.Proxy.gloo-system.gateway-proxy':
      reportedBy: gloo
      state: Accepted
spec:
  virtualHost:
    domains:
    - '*'
    routes:
    - matchers:
      - exact: /all-pets
      options:
        prefixRewrite: /api/pets
      routeAction:
        single:
          upstream:
            name: default-petstore-8080
            namespace: gloo-system
    - matchers:
      - exact: /get 
      routeAction:
        single:
          upstream:
            name: default-httpbin-8000 
            namespace: gloo-system
    - matchers:
      - exact: /post  
      routeAction:
        single:
          upstream:
            name: default-httpbin-8000 
            namespace: gloo-system
```

```
kubectl apply -f vs.yaml
```

- Vamos testar as configs do API Gateway novamente

```
glooctl check
```

- Agora podemos validar nossa aplicação do httpbin

```
curl -XGET -Is $(glooctl proxy url)/get
curl -XPOST -Is $(glooctl proxy url)/post
```

- Agora que estamos com a aplicação de testes em funcionamento, vamos aplicar uma policy simples via OPA-Envoy para validar o token e o método utilizado em cada endpoint

```
############################################################
# Configuration to bootstrap OPA-Envoy sidecars.
############################################################
apiVersion: v1
kind: ConfigMap
metadata:
  name: opa-envoy-config
data:
  config.yaml: |
    plugins:
      envoy_ext_authz_grpc:
        addr: :9191
        path: envoy/authz/allow
    decision_logs:
      console: true

---

############################################################
# Policy to enforce into OPA-Envoy sidecars.
############################################################
apiVersion: v1
kind: ConfigMap
metadata:
  name: opa-policy
data:
  policy.rego: |
    package envoy.authz

    import future.keywords

    import input.attributes.request.http as http_request

    default allow := false

    allow if {
        is_token_valid
        action_allowed
    }

    is_token_valid if {
        token.valid
        now := time.now_ns() / 1000000000
        token.payload.nbf <= now
        now < token.payload.exp
    }

    action_allowed if {
        http_request.method == "GET"
        token.payload.role == "guest"
    }

    action_allowed if {
        http_request.method == "GET"
        token.payload.role == "admin"
    }

    action_allowed if {
        http_request.method == "POST"
        token.payload.role == "admin"
    }

    token := {"valid": valid, "payload": payload} if {
        [_, encoded] := split(http_request.headers.authorization, " ")
        [valid, _, payload] := io.jwt.decode_verify(encoded, {"secret": "secret"})
    }

---
############################################################
# OPA-Envoy deployment. 
############################################################
apiVersion: apps/v1
kind: Deployment
metadata:
  name: opa
  labels:
    app: opa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: opa
  template:
    metadata:
      labels:
        app: opa
    spec:
      containers:
        - name: opa
          image: openpolicyagent/opa:0.45.0-envoy
          volumeMounts:
            - readOnly: true
              mountPath: /policy
              name: opa-policy
            - readOnly: true
              mountPath: /config
              name: opa-envoy-config
          args:
            - "run"
            - "--server"
            - "--config-file=/config/config.yaml"
            - "--addr=0.0.0.0:8181"
            - "--ignore=.*"
            - "/policy/policy.rego"
      volumes:
        - name: opa-policy
          configMap:
            name: opa-policy
        - name: opa-envoy-config
          configMap:
            name: opa-envoy-config

---
############################################################
# Expose OPA-Envoy
############################################################
apiVersion: v1
kind: Service
metadata:
  name: opa
spec:
  selector:
    app: opa
  ports:
    - name: grpc
      protocol: TCP
      port: 9191
      targetPort: 9191
```

```
kubectl apply -f opadeployment.yaml
```

- Vamos agora alterar as configs do API Gateway para trabalhar com o OPA

```
global:
  extensions:
    extAuth:
      extauthzServerRef:
        name: default-opa-9191 
        namespace: gloo-system
```

- Aplicamos as configs acima da seguinte forma

```
helm upgrade --install --namespace gloo-system --create-namespace -f gloo.yaml gloo gloo/gloo
```

- Agora alteramos o arquivo de rotas para trabalhar com o OPA

```
spec:
  virtualHost:
    options:
      extauth:
        customAuth: {}
```

```
kubectl patch vs default -n gloo-system --type=merge --patch "$(cat vs-patch.yaml)"
```

### Testando nossa aplicação com os filtros de autorização do OPA

- Exportamos dois tokens em nosso terminal apenas para testes

```
export ALICE_TOKEN="eyJhbGciOiAiSFMyNTYiLCAidHlwIjogIkpXVCJ9.eyJleHAiOiAyMjQxMDgxNTM5LCAibmJmIjogMTUxNDg1MTEzOSwgInJvbGUiOiAiZ3Vlc3QiLCAic3ViIjogIllXeHBZMlU9In0.Uk5hgUqMuUfDLvBLnlXMD0-X53aM_Hlziqg3vhOsCc8"
export BOB_TOKEN="eyJhbGciOiAiSFMyNTYiLCAidHlwIjogIkpXVCJ9.eyJleHAiOiAyMjQxMDgxNTM5LCAibmJmIjogMTUxNDg1MTEzOSwgInJvbGUiOiAiYWRtaW4iLCAic3ViIjogIlltOWkifQ.5qsm7rRTvqFHAgiB6evX0a_hWnGbWquZC0HImVQPQo8"
```

- Rodamos via curl um teste

```
curl -XGET -Is -H "Authorization: Bearer $ALICE_TOKEN" $(glooctl proxy url)/get
HTTP/1.1 200 OK
```

```
curl http -XPOST -Is -H "Authorization: Bearer $ALICE_TOKEN" $(glooctl proxy url)/post
HTTP/1.1 403 Forbidden
```

```
curl http -XPOST -Is -H "Authorization: Bearer $BOB_TOKEN" $(glooctl proxy url)/post
HTTP/1.1 200 OK
```

- Validando tempo de resposta

```
time curl -XGET -Is -H "Authorization: Bearer $ALICE_TOKEN" $(glooctl proxy url)/get
HTTP/1.1 200 OK
server: envoy
date: Sat, 15 Oct 2022 19:08:44 GMT
content-type: application/json
content-length: 528
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 1
x-envoy-decorator-operation: httpbin.default.svc.cluster.local:8000/*


real	0m0,091s
user	0m0,056s
sys	0m0,038s
```

