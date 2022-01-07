# Os principais controllers 

"Na robótica e em automações, um loop de controle é um loop sem fim que regula o estado de um sistema." Definição disponível na [documentação oficial](https://kubernetes.io/docs/concepts/architecture/controller/).

## Pod

[Documentação oficial](https://kubernetes.io/docs/concepts/workloads/pods/)

O Pod é a menor instância do Kubernetes, mas qual a diferença entre o pod e um container? **Basicamente um pod pode ser composto por um ou mais containers**. Mas no fim, qual o benefício disso? Basicamente com esta topologia, você consegue manter a ideia de responsabilidade mínima de cada pod, e não transformar o mesmo, em uma máquina virtual, com N funções. Isso tudo é possível graças ao namespace, funcionalidade do kernel linux que explicamos anteriormente, no qual **um pod, compartilha os namespaces entre todos os containers**.

Como exemplo podemos ter a seguinte topologia, no qual um pod possui dois containers rodando nele, um com a sua aplicação devidamente empacotada e em execução e outro container que é responsável por ler os logs gerados pela aplicação e enviar para um centralizador de logs por exemplo. Isso tudo é possível já que eles compartilham do mesmo namespace, portanto os dados que o container A cria, o container B consegue coletar, pois eles compartilham da mesma árvore de processos de disco, rede, usuário e afins.
