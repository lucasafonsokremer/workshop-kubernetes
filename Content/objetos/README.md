# Objetos do Kubernetes

No kubernetes, possuímos vários tipos de objetos, como por exemplo os controllers (Ex.: Deployment).

Os objetos são entidades persistentes no sistema do Kubernetes, afim de representar um estado.

## Objeto spec e status

Quando você cria um novo objeto, você precisa especificar por exemplo o spec. O status você normalmente vai ver quando realizar um describe do objeto em si, nele podemos validar o estado atual do componente.

Alguns campos **required** na construção de um yaml:

* apiVersion
* kind
* metadata
* spec

Exemplo:

```
# Qual versão da API do Kubernetes você está utilizando para gerenciar o objeto
apiVersion: apps/v1
# Qual tipo do objeto você está gerenciando
kind: Deployment
# Informações para manter único aquele objeto
metadata:
  name: nginx-deployment
  # labels do deployment
  labels:
    app: nginx
  namespace: default
# este é o spec do deployment com o estado desejado do objeto
spec:
  replicas: 3
  # com base neste selector o service e o deployment consegue encontrar os pods
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      # labels do pod
      labels:
        app: nginx
    # este spec é o do pod podendo conter um ou mais containers
    # contem nome do container, imagem, porta, limitação de recursos, liveness e readiness probes e afins
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```
