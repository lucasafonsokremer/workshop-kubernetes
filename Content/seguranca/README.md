# Algumas práticas de segurança

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

 
