# Manipulação e criação de imagens

[Documentação oficial sobre boas práticas de criação de imagens](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Definição sobre containers

[Nesta documentação](https://github.com/lucasafonsokremer/workshop-kubernetes/blob/main/Content/origem/README.md#containers) explico com exemplos sobre imagens e containers.

## Camadas das imagens

Uma boa definição sobre isso é do Jérome Petazzoni onde ele fala o seguinte:

**" Uma imagem é como se você possuísse um livro, que você pode fazer anotações como queira, entretanto, a cada anotação que você faz alguém tira um xerox desta página e te entrega apenas a cópia"**

Com base nesta definição, uma imagem seria várias instruções e cada instrução é construída em camadas (somente leitura), e no topo dela eu possuo uma camada read-write.

Um exemplo, seria criar uma imagem e instalar um programa no sistema operacional via gerenciador de pacotes (apt-get), como ele faz um cache local no SO, se você não fizer a limpeza deste cache na mesma camada que está instalando os pacotes, na próxima camada você não conseguirá fazer esta limpeza. Portanto é uma boa prática agrupar comandos da mesma "natureza" em uma única instrução.

**Um ponto importante é que apenas as instruções RUN, COPY e ADD que criam camadas, as demais instruções criam camadas intermediárias temporárias e que não acrescentam ao tamanho final da imagem**.

## Instruções

Algumas das instruções mais comuns são:

```
FROM debian:latest             # especifica qual imagem ele vai usar como base
LABEL app="frontend"           # Informacao que vai ficar dentro do metadata do contaner
ENV maintainer="user@domain"   # Variavel de ambiente que vai ta dentro do container
RUN mkdir diretoriodomeuapp    # RUN executa um comando em momento de build
```

Agora para fazer o build é simples, basta executar o seguinte comando:

```
docker image build -t imagemtesteworkshop:1.0 .
```

Outras instruções importantes são:

* **ENTRYPOINT** = Principal processo do meu container, como se fosse um init, se este processo morrer, meu container morre. O modo EXEC é o preferido a ser utilizado no entrypoint.
  * Exemplo: ENTRYPOINT ["/usr/sbin/apachectl"]
* **CMD**        = Se não existir o entrypoint, os parâmetros devem ser passados via CMD. Ele deve executar um comando/processo quando o container subir (SO PODE TER 1 CMD POR CONTAINER)
  * Exemplo: ["CMD stress --cpu 1"]
* **ADD e COPY** = O "COPY" copia um arquivo do diretório e joga no container, o "ADD" por outro lado copia o arquivo mas tem mais opções como descompactação e compactação de arquivos, além de conseguir baixar arquivos de uma derminada URL
* **WORKDIR**    = Diretório padrão a ser utilizado dentro do meu container

Exemplo prático de um [container](https://github.com/lucasafonsokremer/docker-build-awx):

```
# Versao do awx a ser utilizada como base
FROM ansible/awx:14.1.0

# Definir aqui a versao do pip, casando com a da plataforma para ser utilizada na criacao dos virtual env
ENV VERSAOAWXPIP=19.3.1

# Labels para identificar o responsavel destes containers
LABEL author="Lucas Afonso Kremer"

# Usuario atualmente configurado no projeto oficial
USER root

# Copia o diretorio com as libs a serem instaladas nos venvs
# Respeitar a tag das imagens e colocar o nome dos arquivos neste padrao
ADD Dockerfile_requirements /tmp/Dockerfile_requirements

# Instala o zabbix-sender, libcurl-devel e gcc para compilar algumas libs em python
RUN yum install -y http://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-sender-5.0.0-1.el8.x86_64.rpm gcc libcurl-devel ; \
    yum clean all

# Cria o diretorio base dos venvs custom, por padrao seguir a tag da versao do AWX 
RUN for virtualenvs in $(ls -1 /tmp/Dockerfile_requirements/) ; \
    do \
        mkdir -p /opt/custom_venv/${virtualenvs} ; \
        chmod -R 0755 /opt/custom_venv ; \
    done

# Cria o novo venv sem o pip para selecionar uma versao especifica depois
# Cria os venvs com visao nas libs globais do python que sao instaladas via gerenciador de pacotes
RUN for virtualenvs in $(ls -1 /tmp/Dockerfile_requirements/) ; \
    do \
        virtualenv --no-pip --system-site-packages /opt/custom_venv/${virtualenvs} ; \
    done

# Ativa o venv e instala todas as bibliotecas que o custom necessita
# Instala uma versao especifica para o pip
RUN for virtualenvs in $(ls -1 /tmp/Dockerfile_requirements/) ; \ 
    do \
    sh -c "source /opt/custom_venv/${virtualenvs}/bin/activate ; \
           curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py ; \
           python get-pip.py pip==${VERSAOAWXPIP} ; \
           pip install -r /tmp/Dockerfile_requirements/${virtualenvs} ; \
           deactivate" ; \
    done
```

## Multi Stage