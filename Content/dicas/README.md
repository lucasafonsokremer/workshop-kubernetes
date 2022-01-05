# Dicas para aumentar a produtividade

## Bash Completion

* Para facilitar nossa vida e fazer com que os comandos possuam o autocompleate, vamos instalar o bash-completion em nossa máquina, basta abrir seu terminal favorito e executar os seguintes comandos:

```$bash
sudo apt-get install bash-completion
echo 'source /usr/share/bash-completion/bash_completion' >> ~/.bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

* Feito isso é necessário abrir uma nova sessão no seu terminal, para que tudo funcione.

## Plugins do Kubectl com Krew

### Extensão do Kubernetes

* O Kubernetes é uma ferramenta altamente configurável e de fácil extensão. Com isso é possível customizar em vários níveis da plataforma.

* Para um maior entendimento sobre as possibilidades de expansão do Kubernetes confira esta [documentação](https://kubernetes.io/docs/concepts/extend-kubernetes).

### O Krew

**Krew itself is a kubectl plugin that is installed and updated via Krew (yes, Krew self-hosts).**

* O Kubectl por padrão não possui várias funcionalidades, e através do Krew, podemos gerenciar os plugins de forma fácil e "acoplar" novas funções ao nosso kubectl.

* A documentação do Krew se encontra no repositório oficial, que você pode conferir neste [link](https://github.com/kubernetes-sigs/krew).

### Instalação do Krew

* Documentação oficial de instalação está disponível [aqui](https://krew.sigs.k8s.io/docs/user-guide/setup/install/), mas vou resumir logo abaixo com alguns exemplos:

**Instalação no shell**

```
( 
  set -x; cd "$(mktemp -d)" && 
  OS="$(uname | tr '[:upper:]' '[:lower:]')" && 
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" && 
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" && 
  tar zxvf krew.tar.gz && 
  KREW=./krew-"${OS}_${ARCH}" && 
  "$KREW" install krew 
)
```

**Após instalado basta fazer o export para encontrar o binário**

```
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.bashrc
```

### Instalando plugins

* Para listar todos os plugins basta:

```
kubectl krew search
```

* Para instalar algum plugin é fácil, basta executar o comando:

```
kubectl krew install [nome do plugin]
```

* Alguns plugins que faço uso são:

```
kubectl krew install ctx
kubectl krew install ns
kubectl krew install example
kubectl krew install score
```

### Utilizando um plugin

* Para fazer uso de um plugin executamos:

```
kubectl example deployment
```

* Ele deve retornar:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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
