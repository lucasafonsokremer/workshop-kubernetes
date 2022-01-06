# The Big Bang: Containers, Namespaces, CGroups, Runtime e Kubernetes

## Containers

**Históricamente, havia um problema na carga e transporte de mercadorias em návios comerciais, já que as cargas possuiam diferentes tamanhos, formatos e peso. Isto em grande escala, tornava complexo o processo de gerenciamento e transporte. Com este problema em mãos, foi desenvolvido um "padrão" de armazenamento que, indiferente da carga, o gerênciamento e transporte seriam sempre os mesmos, aumentando assim nossa escala e produtividade.**

**Os containers em computação, não são muito diferentes já que ele é um agrupamento de uma aplicação com todas as suas dependências e que compartilha o Kernel do sistema operacional** do host onde ele está executando, seja ela uma máquina física ou virtual.

O container é nada mais que uma imagem em execução. Esta imagem por sua vez deve ser enxuta, contendo apenas o necessário para rodar a aplicação, sem que ocorra nenhuma alteração, criando-se assim a ideia de imutabilidade. **O Kernel compartilhado, proporciona um melhor desempenho devido ao gerenciamento centralizado de recursos, é possível inclusive fazer uma analogia a ideia de: "É melhor ter um único porteiro para um edifício, já que centralizamos nele todo o gerênciamento de entrada no condomínio, imagine agora ter um porteiro por apartamento..."**

Além da facilidade de empacotar aplicações, o uso de containers permite emular um novo sistema operacional e reutilizar os recursos de hardware de uma forma mais inteligente.

Por fim mas não menos importante, um container tem como característica a portabilidade. Como ele deve ser imutável e todas as dependências imbutidas nele, ele irá rodar em qualquer outro sistema que possua por exemplo, um Docker instalado.

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

Existem
