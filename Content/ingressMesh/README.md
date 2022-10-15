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
