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
