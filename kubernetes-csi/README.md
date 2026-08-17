- Создать бакет s3 через интерфейс yandex cloud
- Создать service-account с ролью `storage.admin`
- Зайти в созданный service account и создать для него ключ (static key)
- Создать [секрет](./s3-secret.yaml) для ключа
- Создать [StorageClass](./csi-storage-class.yaml)
- Установить csi драйвер
```shell
git clone https://github.com/yandex-cloud/k8s-csi-s3.git
cd ./k8s-csi-s3/deploy/kubernetes
kubectl create -f provisioner.yaml
kubectl create -f driver.yaml
kubectl create -f csi-s3.yaml
```
- Создать [pvc](./csi-pvc.yaml)
- Создать [pod](./pod.yaml), который будет писать в бакет 
