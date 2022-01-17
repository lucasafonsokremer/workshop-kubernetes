# The Big Bang: Containers, Namespaces, CGroups, Runtime e Kubernetes

## Containers

**Historicamente, havia um problema na carga e transporte de mercadorias em navios comerciais, já que as cargas possuíam diferentes tamanhos, formatos e pesos. Isto em grande escala, tornava complexo o processo de gerenciamento e transporte. Com este problema em mãos, foi desenvolvido um "padrão" de armazenamento que, indiferente da carga, o gerenciamento e transporte seriam sempre os mesmos, aumentando assim nossa escala e produtividade.**

**Em computação, os containers não são muito diferentes, trata-se de um agrupamento de uma aplicação com todas as suas dependências e que compartilha o Kernel do sistema operacional** do host onde ele está executando, seja ela uma máquina física ou virtual.

O container é nada mais que uma imagem em execução. Esta imagem por sua vez deve ser enxuta, contendo apenas o necessário para rodar a aplicação, sem que ocorra nenhuma alteração, criando-se assim a ideia de imutabilidade. **O Kernel compartilhado, proporciona um melhor desempenho devido ao gerenciamento centralizado de recursos, é possível inclusive fazer uma analogia a ideia de: "É melhor ter um único porteiro para um edifício, já que centralizamos nele todo o gerênciamento de entrada no condomínio, imagine agora ter um porteiro por apartamento..."**

Além da facilidade de empacotar aplicações, o uso de containers permite emular um novo sistema operacional e reutilizar os recursos de hardware de uma forma mais inteligente.

Por fim mas não menos importante, um container tem como característica a portabilidade. Como ele deve ser imutável e todas as dependências embutidas nele, ele irá rodar em qualquer outro sistema que possua por exemplo, um Docker instalado.

### Ambiente tradicional vs container

A imagem abaixo compara a execução de aplicações em servidores físicos, virtuais e containeres, apresentando a visão de camadas entre hardware, sistema operacional e aplicação.

* Traditional deployment: Aplicações instaladas diretamente em servidores físicos. Dificultando a combinação de necessidades das aplicações e hardware disponível no servidor, gerando desperdício de recurso e problemas de escalabilidade.

* Virtualized deployment: Servidores virtuais com recursos suficientes para a execução das aplicações, compartilham o hardware, possibilitando melhor uso dos recursos. Porém, necessita de um SO Guest (isolado) para cada VM, com mais camadas entre o hardware e aplicação final aumenta o overhead e o consumo de memória.

* Container deployment: Containers são similares à VMs, porém compartilham o SO possuindo um isolamento mais leve, essa característica reduz o overhead e permite que containeres possam ser instanciados em poucos segundos, tendo também um footprint de memória mais enxuto, apenas com a aplicação fim e as libs necessárias para sua execução.

<p align="center">
  <img src="https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

## Namespaces

O Namespace é uma funcionalidade do Kernel Linux que vem desde a versão 2.6.24 e ele permite o total isolamento de processos. **Em poucas palavras, os namespaces são nosso separador lógico**. Basicamente ele permite que um container possua seu próprio ambiente isolado, ou seja, cada container possui a sua árvore de processos, sem que um interfira na execução do outro.

* PID namespace = Permite que cada container possua seus próprios IDs de processo
* Net namespace = Permite que cada container possua sua própria interface de rede e portas
* Mnt namespace = É a evolução do chroot, com ele é possível que cada container seja dono do seu ponto de montagem no sistema de arquivos
* IPC namespace = Permite um SystemV IPC isolado
* UTS namespace = Permite o isolamento de hostname, domínio, versão do sistema operacional e afins
* User namespace = Este é o último namespace que foi adicionado ao kernel, com ele é possível manter um mapa de IDs de usuários isolados para cada container

## CGroups

O CGroups é responsável por limitar o uso de hardware do host pelos containers, portanto conseguimos gerenciar memória, CPU, disco e afins. **O CGroups basicamente, é nosso separador físico**.

## Runtime

Vou deixar o seguinte [vídeo](https://youtu.be/RyXL1zOa8Bw) com uma palestra interessante da CNCF sobre container runtime e OCI.

Em 2015 nasceu o Open Container Initiative (OCI), que especifica referências de runtime e especificações de formato de imagens.

Com o OCI, é possível ter uma garantia de padronização nas imagens e especificações que um runtime utiliza, fazendo que eu consiga rodar qualquer imagem de container em qualquer runtime. Hoje existem vários runtimes disponíveis no mercado, com diferentes performances, estabilidade, extensibilidade, compatibilidade com o Kubernetes, isolamento adicional e entre outros.

Alguns runtimes utilizados no mercado:
* Containerd (utilizado pelo Docker)
* CRI-O

No seguinte [link](https://sysdig.com/blog/sysdig-2021-container-security-usage-report/) é possível ver com detalhes a % de uso dos runtimes em 2021, através da pesquisa da Sysdig.

## Kubernetes

Kubernetes nada mais é que um sistema de orquestração de containers, desenvolvido pela Google e liberado na comunidade em 2014, mas que agora é mantido pela Cloud Native Computing Foundation (CNCF).

Neste [link](https://kubernetes.io/pt-br/docs/concepts/overview/what-is-kubernetes) podemos ter uma descrição mais ampla do que é a ferramenta.

Existem alguns orquestradores no mercado além do Kubernetes, como por exemplo o Nomad e Swarm. Mas o Kubernetes possui algumas características que o tornam disparado o principal orquestrador de containers do mercado, sendo suas principais características:

* **Possui infraestrutura declarativa: Em resumo eu falo para o kubernetes qual o estado desejado e ele se encarrega de entregar aquele estado**
* **Reconciliation loop: Basicamente o scheduler fala o estado desejado para os diferentes subsistemas (chamados de controllers) e eles trabalham para atingir este objetivo.**

Além destas características, outros pontos bem importantes e que diferenciam de outros orquestradores são:

* Descoberta de serviço e balanceamento de carga
* Orquestração de armazenamento
* Lançamentos e revisões automatizadas
* Empacotamento binário automático
* Autocorreção
* Gerenciamento de configuração e segredos

Para que tudo isso seja possível, o Kubernetes possui uma arquitetura distribuída, baseada em micro serviços, que você pode validar como o control plane funciona neste [link](https://kubernetes.io/pt-br/docs/concepts/architecture/control-plane-node-communication/) e neste [link](https://kubernetes.io/pt-br/docs/concepts/architecture/cloud-controller/) você terá uma visão dos componentes.

Mas para facilitar, seus principais componentes são:

* kube-apiserver = Tudo passa por ele, somente ele pode escrever no ETCD, portanto ele é a central de operação do cluster
* ETCD = Banco de dados que armazena o estado do cluster, rede e outras informações
* kube-scheduler = Gerencia onde será escalado novos pods, analisando qual node é o melhor para receber aquele pod
* kube-controller-manager = Controle principal que interage com o kube-apiserver para validar determinado estado
* kubelet = Nosso "node agent" que roda em todos os nós. Ele é responsável por conversar com nosso container runtime e garantir que os containers que precisam estar em execução, realmente estejam
* kube-proxy = Quando eu crio um serviço, o kube-proxy vai criar todas as regras de iptables (rede) para estruturar os serviços em todos os nós
* CNI = Cria uma interface de rede, responsável por tratar a comunicação pod to pod.

### Arquitetura do Kubernetes

<p align="center">
  <img src="https://d33wubrfki0l68.cloudfront.net/518e18713c865fe67a5f23fc64260806d72b38f5/61d75/images/docs/post-ccm-arch.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

### Mas e como todos estes componentes se conversam, para subir meu container?

<p align="center">
  <img src="https://cdn-images-1.medium.com/max/1500/1*WDJmiyarVfcsDp6X1-lLFQ.png"/>
</p>

Fonte: Imagem obtida diretamente pelo Google

