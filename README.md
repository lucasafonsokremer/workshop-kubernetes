## Workshop Kubernetes


![](https://avatars.githubusercontent.com/u/13629408?s=200&v=4)

Neste repositório descrevo passo à passo como executar um cluster Kubernetes localmente para fins de estudo. Nos tópicos abaixo você vai obter informações para criar cluster `Múltiplos Nós`, `Load Balancer`, `Observabilidade` e afins.

**Conteúdo:**

01. [**Instalação e configuração do Kind**](Content/kind/README.md)
02. [**LoadBalancer com MetalLB**](Content/metallb/README.md)
03. [**Metrics Server**](Content/metrics-server/README.md)
04. [**Kubernetes Dashboard**](Content/dashboard/README.md)
05. [**Nginx Ingress Controller**](Content/nginx-ingress-controller/README.md)

**Requisito:**

01. Docker devidamente instalado e em execução
02. Executar o comando abaixo afim de validar a instalação do docker:

```
docker run hello-world
```

* Você deve obter um resultado como este:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
