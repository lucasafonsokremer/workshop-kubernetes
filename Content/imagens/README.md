# Manipulação e criação de imagens

[Documentação oficial sobre boas práticas de criação de imagens](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Definição sobre containers

[Nesta documentação](https://github.com/lucasafonsokremer/workshop-kubernetes/blob/main/Content/origem/README.md#containers) explico com exemplos sobre imagens e containers.

## Camadas das imagens

Uma boa definição sobre isso é do Jérome Petazzoni onde ele fala o seguinte:

**"Uma imagem é como se você possuísse um livro, que você pode fazer anotações como queira, entretanto, a cada anotação que você faz alguém tira um xerox desta página e te entrega apenas a cópia."**

Com base nesta definição, uma imagem consistiria em um conjunto de instruções, cada instrução constituindo uma camada (read-only), em forma de pilha e no topo delas temos mais uma camada read-write.

Um exemplo seria criar uma imagem e instalar um programa no sistema operacional via gerenciador de pacotes (apt-get), ao executar o apt será criado um cache local no SO na mesma camada e por isso apenas nela o cache pode ser limpo, na próxima camada você não conseguirá fazer esta limpeza. Portanto é uma boa prática agrupar comandos da mesma "natureza" em uma única instrução.

**Um ponto importante é que apenas as instruções RUN, COPY e ADD criam camadas, as demais instruções criam camadas intermediárias temporárias e que não acrescentam ao tamanho final da imagem**.

## Instruções

Vamos dar início aos nossos testes, criando nossa primeira imagem.

Primeiramente é necessário criar um diretório e colocar nele apenas arquivos que devem conter na aplicação e o próprio Dockerfile:

```
# criar um diretório de testes:
mkdir /tmp/testedocker
# criar dockerfile
touch /tmp/testedocker/Dockerfile
```

Agora basta colar as seguintes instruções dentro do Dockerfile:

```
FROM debian:latest             
LABEL app="frontend"           
ENV maintainer="user@domain"   
RUN mkdir diretoriodomeuapp    
```

* **FROM**       = Especifica qual imagem será usada como base 
* **LABEL**      = Informação que vai ficar dentro do metadata do contaner
* **ENV**        = Variável de ambiente que vai ta dentro do container
* **RUN**        = Executa um comando em momento de build

Para fazer o build, basta executar o seguinte comando:

```
docker image build -t imagemtesteworkshop:1.0 .
```

Outras instruções importantes são:

* **USER**       = Usuário a ser utilizado na execução do container, por padrão é o root, mas é preferível utilizar um usuário com menos privilégios administrativos.
* **ENTRYPOINT** = Principal processo do container, como se fosse um init, se este processo morrer, o container morre. O modo EXEC é o preferido a ser utilizado no entrypoint.
  * Exemplo: ENTRYPOINT ["/usr/sbin/apachectl"]
* **CMD**        = Se não existir o entrypoint, os parâmetros devem ser passados via CMD. Ele deve executar um comando/processo quando o container subir (apenas um CMD por container).
  * Exemplo: CMD ["/usr/bin/stress", "--cpu", "1"]
* **ADD e COPY** = O "COPY" copia um arquivo do diretório para o container, o "ADD" por outro lado copia o arquivo e tem mais opções como descompactação e compactação de arquivos, além de conseguir baixar arquivos de uma determinada URL
* **WORKDIR**    = Define o diretório padrão a ser utilizado dentro do container.
  * Exemplo: WORKDIR /opt/app/

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

## Multistage

Em poucas palavras, o multistage cria uma pipeline entre dois containers visando deixar o container final mais enxuto, também é a única forma de utilizar duas instruções FROM no mesmo Dockerfile.

O Multistage ataca dois pontos importantes:

* Diminui a superfície de contato, deixando menos pacotes e possíveis "lixos" dentro do container, que são utilizados apenas no momento do build. Com isso aumentamos a segurança da nossa aplicação;
* Diminui o tamanho final da imagem que será executada, portanto é mais rápido fazer o download de um registry, diminuindo assim um possível downtime da aplicação durante o processo de atualização.

Para criar nosso app com multistage basta:

```
# criar um diretório de testes
mkdir /tmp/multistage
# criar dockerfile
touch /tmp/multistage/Dockerfile
# criar arquivo go
touch /tmp/multistage/meugo.go
```

Agora basta colar as seguintes instruções dentro do Dockerfile (a instrução AS significa apelido):

```
FROM golang:latest AS buildando
WORKDIR /app
ADD . /app
RUN go mod init meugo.go
RUN go build -o meugo.go

FROM alpine:latest
WORKDIR /buildfeito
COPY --from=buildando /app/meugo.go /buildfeito/
ENTRYPOINT ./meugo.go
```

Agora, crie o app copiando as instruções para o arquivo meugo.go:

```
package main
import "fmt"

func main(){
        fmt.Println("Meu app em go")
}
```

Agora realize o build da imagem docker executando o seguinte comando:

```
docker image build -t multistageworkshop:1.0 .
```

### Comparando tamanho e segurança em images com o multistage

Para validar a diferença de tamanho do nosso app, basta rodar o seguinte comando e comparar o tamanho da imagem do golang com alpine:

```
docker images | agrep 'multistage|golang'
```

O retorno foi:

```
multistageworkshop            1.0                 69cd0c2dca6d   22 hours ago    7.35MB
golang                        latest              8b86bf336a01   7 days ago      941MB
```

Agora como diminuímos consideravelmente a superfície de contato do container, vamos validar se realmente possuímos divergência na quantidade de riscos em aberto, mesmo utilizando imagens atualizadas:

* Primeiro rodamos o comando do trivy, para a imagem do alpine, que é a imagem final da nossa aplicação:

```
trivy image multistageworkshop:1.0
```

Nosso retorno para esse caso foi:

```
multistageworkshop:1.0 (alpine 3.15.0)
======================================
Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```

* Agora vamos verificar quantas vulnerabilidades existem na imagem do golang que utilizamos apenas para o build do nosso app:

```
trivy image golang:latest
```

O resultado foi o seguinte:

```
golang:latest (debian 11.2)
===========================
Total: 355 (UNKNOWN: 2, LOW: 254, MEDIUM: 54, HIGH: 31, CRITICAL: 14)
```

## Enviando a imagem para um registry

O primeiro passo é validar se o repositório que você busca já se encontra no registry da AWS, que você pode acessar por este [link](https://console.aws.amazon.com/ecr/repositories?region=us-east-1), lembrando é claro de mudar a região.

Após encontrar o repositório, é necessário autenticar-se na AWS através do "Command line or programmatic access".

Inserindo as credenciais no seu terminal e tendo o aws-cli devidamente instalado, basta realizar login no registry com o seguinte comando:

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin IDREGISTRY.dkr.ecr.us-east-1.amazonaws.com
```

Após receber o ok de login, é necessário realizar o build da nova imagem:

```
docker build -t IDREGISTRY.dkr.ecr.us-east-1.amazonaws.com/logrotate:1.0.0
```

Feito o build, agora é só enviar a imagem pronta para o registry e utilizar dentro do seu EKS.

```
docker push IDREGISTRY.dkr.ecr.us-east-1.amazonaws.com/logrotate:1.0.0
```
