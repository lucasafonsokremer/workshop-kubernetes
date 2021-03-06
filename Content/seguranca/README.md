# Algumas práticas de segurança

## Usuário não privilegiado

Não é uma boa práticia utilizar o usuário root (padrão) nas imagens, portanto na criação do seu container, o ideial é definir um usuário não privilegiado para executar a aplicação.

Segundo este [report da sysdig](https://sysdig.com/blog/sysdig-2021-container-security-usage-report/), quase 60% das aplicações que rodam em produção, rodam com usuário root.

## Fazer uso do multi-stage

É uma boa prática fazer o uso do multi-stage [conforme documentação e exemplo neste link](https://github.com/lucasafonsokremer/workshop-kubernetes/blob/main/Content/imagens/README.md#multistage).


## Realizar scan das imagens

Além do scan de muitos registrys, como o ECR, é possível fazer scan de imagens localmente como por exemplo, utilizando o [Trivy](https://github.com/aquasecurity/trivy)

## Hardening: Service Account

Além de criar uma service account para cada app, já que se não for especificado no deployment, a aplicação fará uso da service account default.

Uma boa prática é criar a service account e remover o token de acesso da API do Kubernetes, já que difícilmente será utilizado. Lembrando que a parametrização no deployment, possui precedencia sobre o da service account, conforme [descrição da documentação oficial](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server).

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```

## Hardening: Deployment

O security context é o campo para realizar especificações e parametrizar alguns "guardrails".

[Nesta documentação](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) podemos aprofundar um pouco mais sobre o tema.

Um ponto importante é cuidar para colocar o security context no local correto, pois se colocar a nível do pod, é possível que afete a funcionalidade de algum sidecar como o Istio.

### Security Context para sobreescrever o usuário padrão do pod:

```
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```

### Security Context para garantir que um processo não escale privilégios dentro do pod:

```
  securityContext:
    allowPrivilegeEscalation: false
```

### Security Context para garantir que o file system está como read only:

```
  securityContext:
    readOnlyFileSystem: true
```

## Hardening: Secrets

[Neste vídeo](https://youtu.be/f4Ru6CPG1z4) podemos ter uma boa base sobre melhores formas de armazenar e utilizar segredos no Kubernetes. 

## Hardening: RBAC

Manter o RBAC sempre com o menor privilégio e o máximo de restrição. Exemplo de como fazer isto [nesta documentação](https://github.com/lucasafonsokremer/workshop-kubernetes/blob/main/Content/rbacApi/README.md#role-based-access-control-rbac).
