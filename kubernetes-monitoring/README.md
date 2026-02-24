### kubernetes-monitoring

- Создать Dockerfile образа ngnix, указав в конфиг файле, что сервер будет отдавать
метрики по адресу `/metrics`

- Установить prometheus operator из helm'а 
`helm install hw-monitoring oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack --namespace homework`

Вывод после установки оператора может пригодиться:
```shell
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace homework get pods -l "release=hw-monitoring"

Get Grafana 'admin' user password by running:

  kubectl --namespace homework get secrets hw-monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace homework get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=hw-monitoring" -oname)
  kubectl --namespace homework port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace homework -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

- Создать deployment для приложения ngnix, так же определить контейнер c ngnix-exporter, указав
ему путь к метрикам приложения. Создать service для этого деплоймента
