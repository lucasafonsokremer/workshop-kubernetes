# Ingress e Service Mesh

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

O API Gateway por outro lado, desempenha o papel de abstrair detalhes e desacopla as implementações, onde ele basicamente cria uma abstração para o nosso usuário final sobre a arquitetura do nosso aplicativo, já que em sistemas distribuídos, baseados em microsserviços é fácil de perder a noção de onde minha aplicação está executando e como devo chamá-la. Portanto um API Gateway vive acima dos aplicativos e serviços, gerenciando o tráfego norte-sul do cluster.

Além disso um API Gateway atua em três recursos que um service mesh não atua:

- Desacoplamento de fronteira
- Controle de tráfego norte-sul rígido
- Faz a ponte de domínios de confiança e segurança

<p align="center">
  <img src="https://media1-production-mightynetworks.imgix.net/asset/24216529/Gateway_vs_Service_Mesh.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google
