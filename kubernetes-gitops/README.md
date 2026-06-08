- Добавить для одной из нод taint, а также лэйбл:

```shell
kubectl taint node cl16maikn5ue0148lr0c-exud node-role=infra:NoSchedule
kubectl label node cl16maikn5ue0148lr0c-exud node-role=infra
```

- Установить Argo CD через Helm
```shell
helm repo add argo-cd https://argoproj.github.io/argo-helm
helm install argo-cd --namespace argo-cd --create-namespace argo-cd/argo-cd -f values.yaml
```

- Создать манифесты проекта и приложений, применить
