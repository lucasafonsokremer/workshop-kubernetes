# RBAC, Certificados e API

## Certificados

Vamos iniciar pelos certificados, parte fundamental dentro do Kubernetes.

[Neste vídeo](https://www.youtube.com/watch?v=gXz4cq3PKdg) temos uma palestra excelente da CNCF.

Como o Kubernetes possui como coração as suas APIs, os certificados das partes possuem como principal papel, a **autenticação das partes no momento da conversa**. Com eles o client autentica no server e o server autentica no client e conseguimos assim garantir uma comunicação segura entre os componentes do cluster.

Dentro do cluster possuímos uma Unidade Certificadora com:

* Trusted root para todo o cluster
* Todos os certificados do cluster assinados pela CA interna
* Utilizado pelos componentes como API Server, Kubelet e afins

Dependendo a versão do Kubernetes, você pode validar com um dos seguintes comandos os certificados (isso se você está utilizando um cluster não gerenciado):

```
kubeadm certs check-expiration
```

ou

```
kubeadm alpha certs check-expiration
```

Com ele vamos conseguir confirmar o seguinte:

```
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Oct 27, 2022 20:34 UTC   286d                                    no      
apiserver                  Oct 27, 2022 20:34 UTC   286d            ca                      no      
apiserver-etcd-client      Oct 27, 2022 20:34 UTC   286d            etcd-ca                 no      
apiserver-kubelet-client   Oct 27, 2022 20:34 UTC   286d            ca                      no      
controller-manager.conf    Oct 27, 2022 20:34 UTC   286d                                    no      
etcd-healthcheck-client    Oct 27, 2022 20:34 UTC   286d            etcd-ca                 no      
etcd-peer                  Oct 27, 2022 20:34 UTC   286d            etcd-ca                 no      
etcd-server                Oct 27, 2022 20:34 UTC   286d            etcd-ca                 no      
front-proxy-client         Oct 27, 2022 20:34 UTC   286d            front-proxy-ca          no      
scheduler.conf             Oct 27, 2022 20:34 UTC   286d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Oct 25, 2031 20:34 UTC   9y              no      
etcd-ca                 Oct 25, 2031 20:34 UTC   9y              no      
front-proxy-ca          Oct 25, 2031 20:34 UTC   9y              no
```

Todos os certificados do cluster estão disponíveis em:

```
/etc/kubernetes/pki
```

## Conversando com a API do cluster

### Consumindo a API do Kubernetes de dentro do pod

Para consumir e realizar alguma ação na API do Kubernetes, necessitamos de duas coisas:

* Uma Service Account com token de acesso
* RBAC liberando ações para esta Service Account

Normalmente, uma aplicação comum não deve possuir o token e nenhuma regra fazendo bind liberando assim o seu acesso. Você até pode ver aplicações interagindo com a API do Kubernetes, mas normalmente será um operator.

Por padrão, ao criar um pod, se não especificado, ele terá acesso ao token da Service Account, mas não terá permissão alguma na API. Podemos fazer uma requisição e ver tudo funcionando da seguinte forma:

* Primeiro criamos uma service account:

```
kubectl create serviceaccount testeapi
```

* Agora vinculamos à um pod:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: testeapi
  name: testeapi
spec:
  serviceAccountName: testeapi
  containers:
  - image: nginx
    name: testeapi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

* Agora entramos no pod e pegamos o token de acesso, que está montado via volume no pod:

```
kubectl exec -it testeapi -- /bin/bash

cat /run/secrets/kubernetes.io/serviceaccount/token
```

* Com o token em mãos, basta fazer a requisição:

```
curl https://kubernetes -k -H "Authorization: Bearer TOKENAQUI"
```

* Devemos receber o seguinte retorno:

```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}
```

### Consumindo a API do Kubernetes externamente

Agora vamos fazer uma requisição manual na API do kubernetes, utilizando o mesmo yaml que o kubectl utiliza.

* Primeiro passo é pegar o conteúdo do usuário padrão do kubernetes:

```
cat /etc/kubernetes/admin.conf
```

* Agora que possuímos os dados utilizados para autenticar na API, vamos extrair o que necessitamos para realizar uma request:

```
echo conteudo-certificate-authority-data | base64 -d > ca
echo conteudo-client-certificate-data | base64 -d > crt
echo conteudo-client-key-data | base64 -d > key
```

* Agora que extraímos os dados, basta fazer uma requisição para o endereço onde a API está exposta:

```
curl https://ENDERECOCLUSTER:6443 --cacert ca --cert crt --key key
```

* O resultado será o seguinte:

```
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/chaos-mesh.org",
    "/apis/chaos-mesh.org/v1alpha1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/coordination.k8s.io/v1beta1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
    "/apis/extensions.istio.io",
    "/apis/extensions.istio.io/v1alpha1",
    "/apis/extensions/v1beta1",
    "/apis/install.istio.io",
    "/apis/install.istio.io/v1alpha1",
    "/apis/metrics.k8s.io",
    "/apis/metrics.k8s.io/v1beta1",
    "/apis/networking.istio.io",
    "/apis/networking.istio.io/v1alpha3",
    "/apis/networking.istio.io/v1beta1",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/networking.k8s.io/v1beta1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/rbac.authorization.k8s.io/v1beta1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/scheduling.k8s.io/v1beta1",
    "/apis/security.istio.io",
    "/apis/security.istio.io/v1beta1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/apis/telemetry.istio.io",
    "/apis/telemetry.istio.io/v1alpha1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/aggregator-reload-proxy-client-cert",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/max-in-flight-filter",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-cluster-authentication-info-controller",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/aggregator-reload-proxy-client-cert",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/max-in-flight-filter",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-cluster-authentication-info-controller",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/informer-sync",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/aggregator-reload-proxy-client-cert",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/max-in-flight-filter",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}
```

## Role-based Access Control (RBAC)


