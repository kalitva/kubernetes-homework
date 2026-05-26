- Добавить для одной из нод taint, а также лэйбл:

```shell
kubectl taint node cl16maikn5ue0148lr0c-amyr node-role=infra:NoSchedule
kubectl label node cl16maikn5ue0148lr0c-amyr node-role=infra
```

Вывод комманд:

```shell
$  kubectl get node -o wide --show-labels
NAME                        STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP       OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME     LABELS
cl16maikn5ue0148lr0c-amyr   Ready    <none>   41d   v1.33.3   10.130.0.25   158.160.140.255   Ubuntu 22.04.5 LTS   5.15.0-168-generic   containerd://1.7.27   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v4a,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-d,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl16maikn5ue0148lr0c-amyr,kubernetes.io/os=linux,node-role=infra,node.kubernetes.io/instance-type=standard-v4a,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,topology.kubernetes.io/zone=ru-central1-d,yandex.cloud/node-group-id=catif92rppp9o92e02jn,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
cl1ra555nv6j8v9n2csu-icog   Ready    <none>   41d   v1.33.3   10.130.0.7    158.160.183.210   Ubuntu 22.04.5 LTS   5.15.0-168-generic   containerd://1.7.27   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v4a,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-d,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1ra555nv6j8v9n2csu-icog,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v4a,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,topology.kubernetes.io/zone=ru-central1-d,yandex.cloud/node-group-id=cat6l28vlu0tm544epdi,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
```

```shell
$ kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
NAME                        TAINTS
cl16maikn5ue0148lr0c-amyr   [map[effect:NoSchedule key:node-role value:infra]]
cl1ra555nv6j8v9n2csu-icog   <none>
```

- Создать s3 хранилище, создать сервисный аккаунт с правами на доступ к хранилищу (например 
`storage.admin`), сгенерировать ключ (static access key). Зайти в кластер, найти вкладку
marketplace, установить csi driver - *Container Storage Interface for S3*. При установке указать
ключи и имя хранилища

- Подготовить [values.yaml](./loki-values.yaml) и установить loki командами:
```shell
helm repo add grafana-community https://grafana-community.github.io/helm-charts
helm repo update
helm install loki grafana-community/loki -f loki-values.yaml
```
